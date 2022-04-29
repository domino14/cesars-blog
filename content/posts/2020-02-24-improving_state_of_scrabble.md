+++
date = "2020-02-24T01:31:26+00:00"
draft = false
tags = ["devjournal"]
title = "Ideas for improving the state of Scrabble"
summary = "I have a lot of ideas for improving the state of tournament and recreational Scrabble, mostly revolving around the technological aspect of things. I am aware that there are many other ways in which it is lacking, but my expertise is in technology, and that's probably the best way I can make an impact"
+++
I have a lot of ideas for improving the state of tournament and recreational Scrabble, mostly revolving around the technological aspect of things. I am aware that there are many other ways in which it is lacking, but my expertise is in technology, and that's probably the best way I can make an impact.

Let's start with some projects I have already built:

### Aerolith

https://www.aerolith.org is my word study website. It's been around in some form or another since 2007; I believe I launched it as a downloadable desktop app around the time of the Players' Championship. I rewrote the whole thing in 2011 as a web app, a considerable undertaking as I knew nothing about web programming, but the experience acquired here helped me obtain a job as an early employee-turned-cofounder at a YC company (Leftronic), which was then acquired a few years later, giving me some minor version of success.

I've used it as a bit of a technological testbed; I added and removed sockets at some point, I've rewritten it with the latest in JS tech (React.js at the time, although it looks like that's here to stay for a bit longer), and played with backend deploy technologies like Kubernetes (mostly a mistake, honestly, especially for such a small site). I get a few hundred users a month, around 450 monthly actives and 150 daily actives last I checked. It's a nice site that I'm pretty proud of, and is mostly self-sustaining in that I get just enough in donations to cover running it for free.

As far as the usefulness of it, it's a bit debatable. I certainly like using it and have learned a lot of words from it, and many people enjoy it and tell me about it, but it has at least a few glaring omissions. The one that comes to mind is some programmed way to study; i.e., something like spaced repetition / cardboxing. I've collected a ton of data on word difficulty that I could use, but I haven't had much time to add more features to my app recently. The deployment of it is also suffering, and I need to greatly simplify it.

For the future, the main things I want to add to it are spaced repetition / some sort of intelligent way to study words, which would keep per-user and global statistics on words that are easy or hard and know how to ask these of the user. I think this would make the tool much more useful. There are also a ton of small improvements, like showing previously guessed anagrams per question, allowing user to easily continue when their time runs out, untimed quizzes, etc.

### Macondo

Macondo is not released yet; it is intended to be a machine-learning version of Quackle. I am still quite early in the development of it, although I have made a bunch of progress. Mostly, it can play games against itself with some preliminary "superleave" values for every rack. We need to determine these superleave values in a better way and have them converge to, or at least be close to Quackle values (or explain why they're not).

Machine learning hasn't been done with Scrabble yet, but I think this field is ripe for innovation. I think a proper machine-learning enabled AI can beat Quackle's Championship Player at least 60-70% of the time. The ideal place to showcase the finished AI would be in a best-of-X match against a top player (ideally Nigel Richards!) with some sponsored prize money on the line. I think this would provide great publicity for the game of Scrabble. Additionally, the tool would make huge strides in making players even better.

Some improvements I would add to it as well:

- Automatic rangefinding
- Heat maps for simulations, other statistics
- A great pre-endgame solver, with a UI clearly showing winning and losing variations, etc
- A great endgame solver (this is a prerequisite for the pre-endgame solver). The Quackle one is good but has issues with stuck tile endgames. I have spent a ton of time building a minimax-based solver with alpha-beta pruning; it's quite slow and buggy. I think I can significantly improve its speed.
- Lots of graphs (showing values of tiles as the game progresses, winning % as game progresses - for easy analysis of famous games, etc)

It's still quite early and it's hard to find time to devote to both projects and my full-time job. Progress can be found here: https://github.com/domino14/macondo

### Crossword Online

Or whatever we want to call it. The state of the art for playing online Scrabble _sucks_ to put it mildly. ISC has an ancient interface, may actually have legitimate randomizer issues (I don't fully buy it but I've heard enough anecdotes and some data), and its phone app is not very functional. EA Scrabble is awful and is going to be discontinued. Its replacement is not getting rave reviews. We need something akin to lichess.org for Scrabble - this beautiful online Chess site is mostly a one-man job it seems. It is my dream to build something like this for Scrabble, but a giant undertaking in terms of time. I actually started on a game viewer and made a lot of progress on the front end for this app, but have almost no server code, and abandoned it once I saw ISC was releasing a phone version. However, after seeing the phone/browser version of ISC it's clear we still need a good solution.

From another perspective, I think an app that allows us to have proper tournaments would be great for the environment. I don't think it's sustainable in the long term for people to be flying everywhere constantly. I'm not saying the world will turn into Ready Player One anytime soon, but with an awesome app and proper anti-cheating technology we could start having some more high-profile tournaments without players having to travel to each other. I do, however, still prefer the analog feel of Scrabble, the actual tiles and board.

Once I decide to tackle this again, I should be able to salvage a lot of my front-end viewer code, and combine it with Macondo to provide a back-end engine. I'd just need to write multiplayer code. The app should support both live games and correspondence games.

### Phone apps for everything

Conrad has designed some nice mocks for Aerolith Mobile, and the Crossword app should also have a mobile app. I can probably use React Mobile to make it easy to cross-develop for Android / iPhone. I've never built a phone app before, but can always learn!

### Aeroblogs, articles, etc

People write ephemeral tournament reports / blogs / strategy articles nowadays as Facebook posts. That's not ideal. They're hard to search, and if you're not on Facebook when they're written you might never see them. LiveJournal was cool while it lasted but people moved on. I think we need a proper blogging platform with actual Scrabble integration (imagine pasting a board diagram in ASCII and it generates a viewable board, with comments, etc, or links to tournaments on cross-tables, etc etc).

I also want to have strategy articles and video content easily available for search on this site. I'm thinking it could be something like channelfireball.com (for Magic The Gathering). We'd need to get content makers, but I think we have a few people who are willing to do that.

## What do I need?

Time and funding. Even though I have some time on weekends, I need to be better at managing it. I am considering setting up a Patreon to see if people are interested. None of this makes much sense if people are not interested - after all, people vote with their wallets.