---
title: "What I'm Working On"
type: "page"
date: 2026-06-29
lastmod: 2026-06-29
description: "A living priority queue of what I'm building across my open-source projects – and why your feature request might be taking a while."
featured_image: '/images/devschedule.png'
menu:
  main:
    weight: 2
---

I'm a husband and father of two young kids. I've spent the past few years co-founding a
startup, which means my open-source side projects get whatever scraps of attention I can
carve out during evenings and weekends – but the backlog isn't getting shorter.

I maintain several projects and there's always something I "owe" someone: a bug fix, a
feature, an enhancement. This page exists for two reasons:

1. **For you** – so you can see where your bug fix / feature request sits relative to
   everything else, and understand why estimates slip.
2. **For me** – an honest prioritized list so I can stop staring at open tabs and actually
   make a dent.

The ordering within and across projects reflects how I *currently* think about the work.
It shifts. I'll try to keep this updated when it does.

## Priority Queue

Project sections are in rough priority order – liwords is most active right now, then
Macondo, then the others. That doesn't mean I won't touch Macondo until liwords is done;
I switch between them. The Aerolith and ESB sections get little love at the moment.

Estimates are left blank intentionally – when I fill them in they're usually wrong anyway.
The `note` on each item explains context or blockers.

{{< queue >}}

<style>
article h3 { margin-top: 2rem; }
#TableOfContents li > ul { margin-top: 0.5rem; padding-left: 1rem; }
</style>

## By Project

### liwords / Woogles

[Woogles](https://woogles.io) is the online crossword board game platform I co-built.
Real-time games, annotation and analysis tools, tournament infrastructure. The codebase is
[woogles-io/liwords](https://github.com/woogles-io/liwords) – a Go backend with a
TypeScript/React frontend.

This is the most active project right now. The main threads:

**Puzzles.** The puzzle system exists but is pretty bare – no tags, no filtering, no
structured search. Overhauling this depends on code I very recently merged into Macondo,
so it's unblocked and near the top of the queue. ([#1706](https://github.com/woogles-io/liwords/issues/1706))

**New languages.** Adding Slovene ([#1845](https://github.com/woogles-io/liwords/issues/1845))
is on the list. New languages also require changes to
[word-golib](https://github.com/domino14/word-golib), the shared lexicon library I maintain
that both Macondo and liwords depend on.

**Infrastructure upgrade.** Postgres, NATS, and Redis all need updating to current versions.
The catch: any serious upgrade means downtime, and downtime means I need to ship adjourn-game
functionality first so live games can survive a restart. So the infra upgrade is gated on
that feature.

**Game architecture overhaul.** This is a large refactor being organized using the [Mikado method](https://mikadomethod.info/) – a technique for untangling complex dependency graphs where you try a change, note what breaks, back it out, fix the prerequisite, and repeat until the path is clear. The goals: remove the in-process game cache (which currently forces hard-cutover deploys and blocks running more than one API node), port the referee/game-rules logic into a new `pkg/game/` package inside liwords, and drop Macondo as a liwords runtime dependency. Finished games would also move to S3, keeping the database lean. Some of this is already in flight.

**Broadcast features.** Three open issues for improving the broadcasting/OBS integration:
player records ([#1855](https://github.com/woogles-io/liwords/issues/1855)),
native tournament mode ([#1856](https://github.com/woogles-io/liwords/issues/1856)),
and more slot types ([#1857](https://github.com/woogles-io/liwords/issues/1857)).

**Analysis.** Two related items: allowing players to analyze a game in the browser turn by
turn (using the existing server-side analysis infra, no WASM needed for now), and moving
stored analyses off the database and onto S3 to keep the DB lean.

And then there's just a long tail of smaller issues in the [repo](https://github.com/woogles-io/liwords/issues)
that each feel tractable in isolation but add up.

---

### Macondo

[Macondo](https://domino14.github.io/macondo) is the crossword board game engine – move
generator, equity calculator, endgame solver, and bot. It powers analysis and bots on
Woogles and is probably the strongest open-source engine for this class of game.

**New lexicons.** Adding Swedish ([#491](https://github.com/domino14/macondo/issues/491))
is the concrete next item. This, and the upcoming Slovene support in liwords, both require
changes to [word-golib](https://github.com/domino14/word-golib) (the shared library for
lexicon data and letter distributions). There's also a bug in word-golib where lexicons
with unrecognized prefixes throw errors instead of being handled gracefully –
[#496](https://github.com/domino14/macondo/issues/496) covers that.

**Bayesian inference.** I've been doing research into improving the inference module –
searching mathematically for the best value of τ (the softmax temperature that controls how
rational we assume the opponent to be when computing P(play | leave): lower values assume
near-optimal play, higher values allow more weight for sub-optimal moves) and running
related experiments. This one is slow-burn research rather than a concrete deadline.

**Next-gen static evaluator.** I trained a CNN-based static evaluator for Scrabble that
already beats HastyBot's long-standing equity engine – details in
[this post](/posts/2025-06-21-nn-scrabble/). But that model was trained on HastyBot
self-play data, which means it learned from a bot that is itself imperfect. The next step
is to collect a much larger corpus of higher-quality self-play data, explore better training
algorithms (Q-learning is one candidate), and retrain to see how much further we can push
the eval. The long-term horizon for this is more ambitious: convert the trained network into
a [NNUE](https://www.chessprogramming.org/NNUE)-style architecture so it runs fast enough
to replace the static evaluator inside Monte Carlo search entirely. If that works it would
be a meaningful leap for Scrabble engine strength – the same jump chess engines made when
Stockfish adopted NNUE.

**GUI.** Eventually Macondo should have a proper graphical interface so it's usable without
a command line ([#35](https://github.com/domino14/macondo/issues/35)). This is in the
"idea" bucket for now – I want to do it, I just haven't figured out the right approach yet.

Repo: [domino14/macondo](https://github.com/domino14/macondo)

---

### Aerolith / WordVault

[Aerolith](https://aerolith.org) is my oldest project, started in 2007 as a Qt desktop
app for studying Scrabble word lists. It's been in maintenance mode for a long time.
When a new official lexicon drops I'll spend a few hours to get everything current, but
it doesn't get sustained attention.

**WordVault** is a study-card system I built on top of Aerolith using the
[FSRS](https://github.com/open-spaced-repetition/fsrs4anki/wiki/ABC-of-FSRS)
spaced-repetition algorithm. It shipped, people use it, and then I didn't touch it for
about two years. It's slowing down – queries are getting sluggish, and I need to revisit
how much guess history we're retaining. This isn't a feature; it's a reliability issue
that needs a pass before it bites harder.

Beyond that, I have ideas for new Wordwalls modes – infinite word walls, a Tetris-inspired
"Tetrolith" variant, and a few others. These are in the idea stage.

Repos: [domino14/webolith](https://github.com/domino14/webolith) ·
[domino14/word_db_server](https://github.com/domino14/word_db_server) ·
[domino14/aerolith-infra](https://github.com/domino14/aerolith-infra)

---

### ESB – Electronic Scrabble Board

A hardware project I'm committed to delivering within the next year. It's still in the
ideation / pre-prototype phase – I don't have a repo to link yet. The concept: a physical
electronic board that can be used to play and track Scrabble games, bridging the gap between
over-the-board and online play. More details to come as it takes shape.

---

### Other / private

There's a collaborative project where I'm licensing a game from a partner who also handles
video creation and advertising. I will share more details in the near future.
It competes for the same hours as the open-source work, which is part of why
"a few weeks" can stretch into months.

---

*Last updated: June 29, 2026. This page's source is in
[github.com/domino14/cesars-blog](https://github.com/domino14/cesars-blog).*
