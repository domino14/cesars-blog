---
title: Breaking the Zyzzyva encryption
description: >-
  The article here sums up pretty well the debacle that North American
  tournament Scrabble players have been faced with since the…
date: '2015-06-20T08:43:19.316Z'
categories: []
keywords: []
tags: ["scrabble"]
---

The article here sums up pretty well the debacle that North American tournament Scrabble players have been faced with since the introduction of the new Tournament Word List in April of 2015:

[**Major Scrabble Brouhaha: Can You Copyright a List of Words?**
_In the 1980s, when Brian Sheppard created a computer program that played Scrabble, he typed in a lot of words-more than…_www.slate.com](http://www.slate.com/articles/life/gaming/2014/09/major_scrabble_brouhaha_can_you_copyright_a_list_of_words.html "http://www.slate.com/articles/life/gaming/2014/09/major_scrabble_brouhaha_can_you_copyright_a_list_of_words.html")[](http://www.slate.com/articles/life/gaming/2014/09/major_scrabble_brouhaha_can_you_copyright_a_list_of_words.html)

Basically, Hasbro or Merriam-Webster or both decided to take the volunteer work various Scrabble players did in putting together a new word list, slapped a copyright on it, and made it impossible for players to obtain a digital version of the word list. This rendered various study tools such as [Quackle](http://quackle.org), the best Scrabble AI, close to useless. I’ve heard a few players argue: who cares if Quackle doesn’t find every last word? It will still give you pretty good plays. I’m sorry, but playing Scrabble without all the acceptable words is like playing chess without knowing how the rook moves. Not quite, but it almost feels like that. Other players argued that we kids had it easy and that back in the day people studied by poring through the physical copy of the book. This is a ridiculous argument; computer study methods such as Leitner spaced repetition or even my own site [Aerolith](https://www.aerolith.org) make it much easier to search for and remember words than looking through a giant book.

During this whole mess, NASPA (the North American Scrabble Players Association) bought out [Zyzzyva](http://zyzzyva.net/), which is the main study program used by players all over the world, and added the word list to it, making it the only program legally able to read the new lexicon. Features were added to it disallowing it from exporting more than 200 words at a time, therefore making it impossible for players to use or make their own study tools. It also limits the lexicon to only NASPA members, which means anyone who wants to play Scrabble on their own, or online, or anywhere, cannot study this word list!

### Hacking

I took a curiosity in the fact that Zyzzyva is a downloadable program that had an encrypted version of the word list in it. Previous versions of Zyzzyva were licensed under the GPL, so the coders amongst us tournament players know it uses a SQLite database to hold all the words - so there is a starting point here. At some point, growing more annoyed that I couldn’t study the new words the way I wanted to (on my phone — NASPA never updated the iOS version of Zyzzyva so it’s broken for new iPhones), I decided to figure out just how Zyzzyva went about encrypting the word list. Although to be honest, I already had a Frankenstein version of the word list cobbled together by various volunteers using OCR and manual methods. Thanks, guys!

Real hackers will find this pretty trivial, but it’s always fun to mess around with different tools that I wouldn’t normally deal with. Plus, if you ever want to break “DRM” used by a stand-alone program that sits right on your computer, this article might guide you along!

I started by downloading the Mac OS version Zyzzyva from the website and looking into its directory structure. I quickly found libqsqlcipher.a, a static library, and googling+ (The plus symbol indicates that GOOGLING is a new word in our lexicon) led me to the SQLCipher page, which promises full SQLite encryption. Aha! I read a bit about it and learned that decrypting a database involves typing PRAGMA key = ‘somekey’.

Running the command “strings” on the Zyzzyva executable shows all the user-readable strings amongst the binary; I was hoping perhaps the programmer had put the key in plaintext somewhere in the source code. I also ran it on a couple of the dynamic library files that come with Zyzzyva. There seemed to be many candidates that could possibly be passwords, and I started testing them by hand with a compiled version of SQLCipher, and quickly ran out of patience. I did a few searches for strings of certain lengths, but there were so many that it seemed fruitless.

#### Read the docs

Heading back to the SQLCipher docs show that the C API has `sqlite3_key()` and `sqlite3_key_v2()` functions. Would these show up somewhere in the source code? Perhaps if the debug symbols were on…

I did a grep search on `sqlite3_key` and found some matches in the libzyzzyva.3.dylib library file that came with the program. This is a great sign! Maybe I can find out where it’s being called and with which arguments.

I must admit that getting to this step actually took me way too long; I spent a couple of hours trying to figure out why there were PUBLIC KEYs embedded in the plaintext of the Zyzzyva executable, and trying to use fingerprints of these as arguments to PRAGMA key, as well as compiling SQLCipher on my laptop (had to spin up a Linux virtual machine for it because I couldn’t get it to compile on my Mac)… but I digress.

#### Time to spin up a debugger

The real hackers will know that as soon as I found evidence of `sqlite3_key_v2` in the Zyzzyva dylib file that getting the key was inevitable. I don’t actually know the steps for removing debug symbols from compiled code off the top of my head, but I bet if this had been done, this would have made my job much, much harder.

I spun up lldb, an OSX debugger, and set it loose on Zyzzyva:

```
$ lldb Zyzzyva
(lldb) breakpoint set -n sqlite3_key_v2
(lldb) run
```

I clicked the annoying nag dialog telling me my NASPA membership is about to expire and thus I will lose access to this lexicon of words that was put together by a bunch of volunteers, and boom, breakpoint hit! Basically, this means I hit the part of the code where this function was called.

```
Process 77243 stopped
* thread #1: tid = 0xe7a6a5, 0x000000010025c330 libzyzzyva.3.dylib`sqlite3_key_v2, queue = ‘com.apple.main-thread’, stop reason = breakpoint 3.1
 frame #0: 0x000000010025c330 libzyzzyva.3.dylib`sqlite3_key_v2
libzyzzyva.3.dylib`sqlite3_key_v2:
-> 0x10025c330 <+0>: pushq %rbp
 0x10025c331 <+1>: movq %rsp, %rbp
 0x10025c334 <+4>: subq $0x30, %rsp
 0x10025c338 <+8>: movq %rdi, -0x8(%rbp)
```

#### X86 assembly!

Yesss. Time to get out the x86 assembly hats. I haven’t seen assembly for a while, but at this point finding the key feels so close! It helps that SQLCipher is open-source and the C code for this function is here:

[**sqlcipher/sqlcipher**
_sqlcipher - SQLCipher is an SQLite extension that provides 256 bit AES encryption of database files._github.com](https://github.com/sqlcipher/sqlcipher/blob/master/src/crypto.c "https://github.com/sqlcipher/sqlcipher/blob/master/src/crypto.c")[](https://github.com/sqlcipher/sqlcipher/blob/master/src/crypto.c)

The first few lines look like:

```
int sqlite3_key_v2(sqlite3 *db, const char *zDb, const void *pKey, int nKey) {
 if(db && pKey && nKey) {
```

Stepping through the assembly code (with the ‘s’ command) quickly showed me various comparisons, which must be the if-statement above:

```
-> 0x10025c355 <+37>: cmpq $0x0, %rax
 0x10025c359 <+41>: je 0x10025c3a7 ; <+119>
 0x10025c35b <+43>: movl -0x1c(%rbp), %eax
 0x10025c35e <+46>: cmpl $0x0, %eax
(lldb)
```

The movl instruction operates on a “long”, which has to be the int nKey argument; the other two arguments are pointers so 64-bits on my machine, which means they would use a movq instruction instead (This is all in Google). The instruction above put some gibberish in register “eax” and then is comparing it with 0 — if it equals 0, this means the if statement above would return false. Assembly is very simple but verbose.

Whatever is in the eax register is the length of the key:

```
(lldb) register read eax
 eax = 0x00000043
```

This translates to 67 in decimal. Woohoo! Now to find the contents of pKey:

```
-> 0x10025c34f <+31>: je 0x10025c3a7 ; <+119>
 0x10025c351 <+33>: movq -0x18(%rbp), %rax
 0x10025c355 <+37>: cmpq $0x0, %rax
 0x10025c359 <+41>: je 0x10025c3a7 ; <+119>
```

This is simpler than it looks. I’m assuming that the “rax” register is the culprit here, as this compare was immediately before the one that gave me the length of the key, and this is also the order in the C code. Also, you can see the movq instruction; 64-bit pointers FTW.

```
(lldb) register read
General Purpose Registers:
 rax = 0x0000000105141300
```

The address `0x0000000105141300` points to a location in memory that has my 67-byte key. What remains here is to read this from memory:

```
(lldb) memory read -s1 -fc -c 67 0x0000000105141300
0x105141300: x’66da82dc4ca3aeae45e01413cdb1f7
0x105141320: ed8a5a19382a1b5fe1b9731606a83ba3
0x105141340: f6'
```

Woohoo!! I have no idea how this key was generated, since it wasn’t in plain text anywhere I looked, but it could have been stored in pure binary, or been part of a picture, or something. Now to try it out…

```
sqlite> PRAGMA key = “x’66da82dc4ca3aeae45e01413cdb1f7ed8a5a19382a1b5fe1b9731606a83ba3f6'”;
sqlite> .schema
CREATE TABLE words (word text, length integer, playability integer, pla..... [redacted]
```

YES.