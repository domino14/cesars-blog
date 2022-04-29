---
title: Using NATS to build a very functional Websocket server
summary: How we built the Woogles backend messaging system using NATS and Websockets.
date: '2020-07-22T05:06:31.568Z'
categories: []
keywords: []
tags: ["devjournal", "golang"]
featured_image: 'https://cdn-images-1.medium.com/max/800/1*h3ZgS7RCKyuXEhi_z3uxjg.jpeg'
---

My new app (still in internal alpha), Woogles.io, is intended to be a place where people can play real-time word games online. Especially during this pandemic, it appears in-person games of any time have completely ceased, and we board gamers need something to keep us sane. Besides joining like 6 different games in the upcoming [Mind Sports Olympiad](https://msoworld.com/about/), the main way I’ve tried to scratch this itch is to gather a team to code up a way to keep crossword board games alive.

![](https://cdn-images-1.medium.com/max/800/1*h3ZgS7RCKyuXEhi_z3uxjg.jpeg)

I first saw [lichess](http://lichess.org) like 5 years ago and thought it was the most amazing thing ever. I’d be lying if I wasn’t inspired by this app, although by now our design is very unique, I learned a lot about the architecture of lichess by looking through its codebase.

Specifically, they mention having a separate [websocket server](https://github.com/ornicar/lila-ws) (or servers), along with Redis to handle the communication between the Websocket servers and the API server. The socket server code has quite a bit of responsibility and seems to have a bunch of chess-related code in it. I am pretty sure this is because the code was recently split from the lichess monolith, so there was likely a bit of technical debt in it. My hope is that starting from scratch means designing it to be as simple as possible to begin with.

### Message routing

As far as I can tell, the basic gist of the lichess socket architecture is the following: the socket server(s) handle Websocket connections directly from the user’s web browser. It then uses Redis Pub/Sub to publish messages directly to the lone API server. The API server does all the chess-related stuff, then publishes back on one or more Redis channels. Finally, the socket servers, which are subscribed to these channels, figures out which client’s Websocket to respond to, and writes information back.

I wanted to go for something similar, but realized pretty quickly that this only works if you have one single API server listening on the Redis subscription, as lichess seemingly does (it is a very large, powerful server, though). Otherwise, every API server would get the message, and every chess move would be performed in X-plicate, etc. The solution is [NATS](https://nats.io/) and the Request-Reply pattern! NATS is a simple, but very powerful message broker that allows for pub/sub and other similar patterns. In particular, request-reply makes it so that you can form a “queue group” out of the different API servers, and deliver the message to only one of the servers, with an optional timeout. The chosen server would then respond back to the message on a specific NATS channel, and the socket server, which blocks on this response (in a goroutine, of course), can then forward this response to the correct client.

NATS also allows us to have a large number of ephemeral channels, without having to pre-declare them (as opposed to something like RabbitMQ). It makes it very easy and fast to construct “realms” where certain types of messages can pass. Some examples from our code are below:

*   gametv.{gameID}: This channel is “TV mode” for a specific game ID — basically, an observer mode that can see both sides of the game.
*   user.{userID}: This channel takes in user-specific messages. In our typical crossword board game, we absolutely do not want to send information about our opponent’s rack back to both users. Although the information can be sanitized on the front-end, any savvy user can easily see both opponent racks by looking at the Websocket messages. For example, [ISC](http://www.isc.ro) allows this (as well as allowing you to modify your rack, as all draws are done client-side!). Other messages that might be on this channel would be private messages from other users, or user-specific notifications.
*   game.{gameID}: This channel is for game-specific messages. For example, the results of a “challenge” event can be shown here, or any specific chat during the game, etc. This differs from gameTV in that the game channel is a bit more private — only chat between the players would go in this channel. We plan for the observers to have their own channel, to disallow any accidental (or intentional!) “kibitzing”.
*   lobby: The lobby will show games that are in progress, open game requests, lobby chat, etc. Keeping those messages in this channel is good for bandwidth; players currently in a game do not need to know who is seeking out new games or who is playing who.

In our socket server, we also have to have some analogous “realms”. When a player goes to a game id, such as `/game/abcdef` , our socket server sets the user’s realm accordingly. If the user is in the `/` main lobby, their realm is set to `lobby` ; if the user is in a game, their realm is set to `game-gameid` or `gametv-gameid` depending on whether they are one of the players of the game, or just an observer. We use a dash `—` instead of a `.` to quickly visually distinguish between our own per-socket-server “Realm” construct and the NATS channel. The `user.userid` channel does not map to a traditional “realm” like the others; a user should \*always\* get all messages that come on this channel.

Note that this Realm mapping makes it so that we don’t create one NATS channel per user socket connection. This seemed a bit excessive and could get complex. Instead, the socket server manages which sockets correspond to which NATS channels by using our Realm mapping.

For the socket server, we used the [gorilla/websocket chat example](https://github.com/gorilla/websocket/tree/master/examples/chat) heavily, and modified it for our purposes. For example, our `Hub` structure looks like this:

As you can see, the `Hub` realms are just a map of a Realm ID to a list of clients. Each `Client`structure essentially contains the actual Websocket connection for that client.

As stated above, our websocket server is designed to be as simple as possible, and thus it doesn’t know anything about the games that it is playing, or even about databases. Therefore, when faced with a request to subscribe a specific websocket `Client` to a realm, it directly asks the API server using NATS:

The `addToRealm` function just adds the client socket to the given Realm, such as `lobby` or `game-gameID` .

Then, the socket server has a pubsub object, which is essentially an object wrapped around a connection to the NATS server. It is listening on the channels I listed above, using the `channel-prefix.>` format. For example,

This `PubsubProcess` function runs on a separate goroutine, and just listens for messages coming in on various NATS channels. I left out the part where we subscribe to each channel, but we just simply create a few subscriptions to the different named topics (`gametv.>` , etc). Also, the `sendToRealm` function above was left out, but it’s pretty simple — it simply sends the socket message to every Websocket that’s in the given realm.

You can see more detail in the Github repo for the socket server here: [https://github.com/domino14/liwords-socket](https://github.com/domino14/liwords-socket)

When we receive a socket message from the user, for example, if they just made a move in a game, we must route this message to the API server eventually. We do a

h.pubsub.natsconn.Publish(“ipc.pb.gameplayEvent.” + userID), msg)

essentially, where msg is the message that just came in. The msg contains the game ID and the actual move that was played.

On the API side, we have a very similar message bus running, which is subscribed to its own NATS channels. Analogously to the above situation, we have it subscribe to the channel named `ipc.pb.gameplayEvent.>` so that it could get all gameplay events. It then fetches the user ID from the user store (a database), the game ID from the game store, ensures the user is playing the given game, validates the move, and then updates the game state, saves it back, and publishes the new gameplay event back to the `gametv.{gameID}` channel and the individual user channels as well.

Note that we don’t publish back to the `game.{gameID}` channel necessarily, because of the fact that each user event should be sanitized (stripped of rack information for the other user). So instead, we sanitize the event per user, and publish it back on the `user.{userID}`channel as well, for each user. This is an implementation detail, though, and in games where there are no secrets, you could just use a single `game` channel if you’d like.

### Multiple API servers

You may notice that this line wouldn’t work properly when we have multiple API servers:

```
h.pubsub.natsconn.Publish(“ipc.pb.gameplayEvent.” + userID), msg)
```

This is because all the API servers would get the message, and they would all try to do the same thing — verify the move, play it, save it into the store, etc. This is something that we will have to fix once we deploy multiple servers. As a hobby app that is not expected to get more than a few thousand concurrent users right off the bat, we hope that we can hold on with a fairly large API server like lichess does. However, it seems like good practice to architect for high scalability even when we have a fairly stateful server. There are two choices here:

1.  **Change the publish to a** `**natsconn.Request**` **like in the realm registration example above.**

This is a fairly simple approach, and it may work for your game (and it may work for ours too, we will run some tests). The main issue is that this requires a completely stateless backend API server. The loop of loading a crossword board game completely from database, replaying it to the current turn (including calculating all of the right anchors / cross-sets needed for bot play — see typical solver algorithms such as Steven Gordon’s [GADDAG](https://en.wikipedia.org/wiki/GADDAG)), validating a move, playing it, recalculating all of the board parameters, scores, and serializing everything back to the database, for one single turn, just might be a bit too intensive. We are honestly not sure, but with many games going on at once and particularly for blitz games my feeling is this will be pretty processor-intensive. For many other games this would likely be fine, though, and statelessness in an API server is always a good idea!

**2\. Build an in-memory cache for the game, and always load from it.**

More intensive games have dedicated stateful servers. Now we’re not building Fortnite here, but we believe that a better experience will be had if we cache games in memory. Our current game store right now is actually entirely in-memory, and it performs rapidly. However, we need to work on adding persistence, so that players can go back and examine/analyze/share their old games.

The solution I have in mind is to save all moves like normal to the database, but load games from the in-memory cache. The cache already would have a completely realized game, with a board, bag state, random seed, anchors, cross-sets, scores, etc. But obviously the problem is that the cache is per-server. Therefore, the NATS subscriber on each API server will have to discard all messages that include a game ID that are not in the cache. I think this should be fast enough.

There would likely be additional issues here, for example if the server crashes, or if for some reason no currently accessible server can find its game in cache (a net split or something of the sort). We wouldn’t want the front-end to just hang forever. Deploying new API servers will also take some time — we’d have to tell the old API servers to accept no more new game requests, then wait until the games are over before de-commissioning them. We’ll have to think of some ways to mitigate these issues. Your comments are very welcome!

### Repos

Code for the socket and main API servers can be found at:

[https://github.com/domino14/liwords-socket](https://github.com/domino14/liwords-socket) and [https://github.com/domino14/liwords](https://github.com/domino14/liwords) respectively.

I’ve enjoyed using NATS and Websockets together to build a scalable, high-performance message bus for a somewhat complex online game. Hope you got some use out of this article!