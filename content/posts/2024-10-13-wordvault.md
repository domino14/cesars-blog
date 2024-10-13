---
title: "WordVault - a new spaced repetition web app for learning words"
date: 2024-10-13
tags: ["aerolith", "scrabble", "study"]
summary: I just released a new web app for learning words called WordVault. Read more about it here.
featured_image: '/images/wordvault-card.png'

---

**Introducing WordVault: A New Tool for Mastering Scrabble with Spaced Repetition**

I'm excited to introduce a new feature for Scrabble players who are serious about improving their word knowledge: WordVault. If you've ever had trouble keeping those tough bingo words or uncommon thirteen-letter words fresh in your mind, WordVault has you covered. It uses an FSRS-based spaced repetition system to help you focus on the words that need practice the most, right when you need it.

You can access WordVault here: [https://aerolith.org/wordvault](https://aerolith.org/wordvault)

Note: you must have an Aerolith account to start learning. You can register at [https://aerolith.org](https://aerolith.org) if you don't already have one.

### What is WordVault?

WordVault is a spaced repetition card app tailored for Scrabble players. It uses FSRS (Free Spaced Repetition Scheduler), a cutting-edge, open-source algorithm that has been shown to outperform older systems like legacy Anki, SuperMemo, and the Leitner cardbox method in terms of both retention and efficiency.

For the curious, you can dive into the details of FSRS [here](https://github.com/open-spaced-repetition/fsrs4anki/wiki/abc-of-fsrs).

FSRS is currently one of the options offered by Anki.

### Getting Started

Once you load WordVault, adding cards is simple. Head over to the "Word Search" tab to select alphagrams to study. I’d recommend starting small if you’re new to spaced repetition—no need to add 300 cards on your first day. A good starting point could be three-letter words or some high-probability bingos.

Some card recommendations for beginners:
- The three-letter words
- The high-value four- and five-letter words
- Top 100 seven-letter and eight-letter bingos

I personally prefer to learn shorter words using Wordwalls, but your mileage may vary.

Example search: 7 and 8 letter words between 1 and 200 in probability.

![High prob 7s and 8s](/images/wordvault-searching-for-bingos.png)

**Note**: If you add the same cards twice, the system will know what to do and will not create duplicates. So if you've forgotten which cards you've added, but you want to make sure all the gaps are covered, you could try re-adding, for example, 1-100, 101-200, etc for the letter lengths you're interested in.

You can add at most 1000 cards on any given search.

### Quizzing Yourself

After building your WordVault, you're ready to start quizzing. Click "Load scheduled questions" and begin! The app will present you with cards to solve each day, ensuring you never feel overwhelmed but still keep progressing. And don't worry—once you've tackled all your daily questions, you can either add more or simply take a break. Tomorrow, the cycle continues!

#### Grading: How FSRS Works its Magic

The real power of FSRS lies in how you grade your responses. Every time you flip a card and see the answer, you'll grade how well you recalled the word. The options are straightforward:
- **Missed**: You didn’t recall the word at all. (Click or type "1")
- **Hard**: You got the word, but it took serious effort. (Type "2")
- **Good**: It was just the right challenge, but you recalled it without too much trouble. (Type "3")
- **Easy**: You nailed it instantly. (Type "4")

One important note: Even if you struggled but got the word right, don’t confuse that with missing it! “Hard” doesn’t mean you forgot—it means you got it after a tough battle.


![Grading cards](/images/wordvault-card.png)


### Staying on Track with Scheduling

Consistency is key with spaced repetition. WordVault will keep you on track by showing you a schedule of your upcoming cards. If life gets busy and you can’t keep up for a few days, you’ll see some overdue cards start to pile up. But no worries—you can click the “Postpone” button to intelligently shift the review dates for those cards you know best, so you can focus on catching up.

Just don’t get too comfortable postponing! Do it too many times, and those words might start slipping through the cracks.

![Grading cards](/images/wordvault-schedule.png)

### Stats to Track Your Progress

WordVault isn’t just about drilling words endlessly; it’s also about knowing how you’re improving. Type in any alphagram to pull up statistics and see your history with that word. Tracking your progress is a great way to stay motivated and make sure your study sessions are paying off.

![Grading cards](/images/wordvault-cardstats.png)

### Mobile

Of course, the app works nicely on mobile as well as desktop. You must have an Internet
connection to use it as it syncs with our server after every card you do.

The app has a light and dark mode:

![Mobile with light mode](/images/wordvault-mobile-light.png)

### Cardboxes

FSRS has no concept of cardboxes. The Leitner cardbox system is another method of doing spaced repetition, but FSRS outperforms it in terms of efficiency and retention.

You can see the stats for every card once you solve it, or with the Card Statistics button. This will give you the main FSRS parameters (Retrievability, Stability, and Difficulty) for every card, as well as how many times you've seen it and forgotten it (Note that if you miss a card the very first time it doesn't count as forgetting it, since you are just learning it!)

### Is it free?

Aerolith is open-source and so is the implementation of this algorithm. You can see the source for both at:

https://github.com/domino14/Webolith

https://github.com/domino14/word_db_server

If you use the hosted version of this server on aerolith.org it is free for the first few thousand cards. You can get unlimited cards with a yearly donation to Aerolith. Please see https://aerolith.org/supporter/ if you are interested in donating!


### Try WordVault Today

Whether you're a Scrabble expert looking to perfect your bingo game or just starting to build your word knowledge, WordVault has something for you. By harnessing the power of FSRS, WordVault brings a smarter, more efficient way to study. Ready to take your Scrabble skills to the next level? Give WordVault a try and see how spaced repetition can change the way you learn and remember words!

[https://aerolith.org/wordvault](https://aerolith.org/wordvault)

Let me know how you like it, and happy word hunting!
