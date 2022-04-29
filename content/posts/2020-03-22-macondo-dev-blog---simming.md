+++
date = "2020-03-22T03:03:31+00:00"
draft = false
tags = ["scrabble", "ai", "golang", "machinelearning", "devjournal"]
title = "Macondo Dev Blog - simming"
+++
I'm going to log more of my progress on the apps that I wrote about in an earlier post, in an attempt to:

1) make myself more likely to work on these apps
2) write a log for me and others and drum up some excitement! ;)

Monte Carlo simulation is basically working on Macondo. I expect that since I just got it working, that I'll discover some bugs and special cases, and there's so much more I want to do with it, but for now I'm excited that I got it working. As a nod to Scrabble expert Ron Tiekert, who might have been the first person to invent Scrabble simulations back in the 1980s, I plugged in the rack that he used to explain this concept, AAADERW.

In the 1980s many top Scrabblers were proponents of turnover theory; that is, they believed that the more tiles you turn over, the better. The theory certainly makes sense to an extent, if your rack doesn't have a bingo, why not race to the blanks, Ss, and other good tiles? So with the rack above, many experts might have played AWARD or WARED, keeping a rack of AE or AA (and nowadays, ADWARE is another good option). But Ron explained to Maven author Brian Sheppard that he preferred opening with AWA, and demonstrated to him how picking 50 random racks after AWA results in a higher score next turn than picking 50 random racks after the other longer plays. Brian explains that this was like "manna from heaven" and proceeded to co-invent the gold standard in Scrabble AIs so far - Monte Carlo simulations as applied to the game of Scrabble. This also disproved turnover theory, again, to an extent. The counter-argument goes that blowing up a potentially nice rack makes it just as likely that you would draw bad tiles as good tiles. The counter-argument to _that_ is that it's situation dependent; turnover certainly has a place when the tile bag looks great, especially towards the end of the game if the blanks are still unseen, and you suspect your opponent doesn't have one, and so forth.

Here are some results after plugging in AAADERW as an opening rack, and simulating the top 10 moves for about 3183 2-ply iterations (100 seconds overall; I should be able to speed this up significantly with more cores and optimizations):


    **Ply 1 (Opponent)**
                    Play    Mean   Stdev Bingo %
    --------------------------------------------
               8C ADWARE  38.379  21.916  21.081
                8H AWARD  36.233  22.763  19.384
               8G ADWARE  39.977  23.149  21.426
               8H ADWARE  38.391  23.625  21.426
                  8H AWA  30.548  19.686  14.923
                  8F AWA  30.111  19.716  14.860
               8D ADWARE  39.559  22.138  21.112
                8D AWARD  33.883  22.050  18.002
               8E ADWARE  35.946  22.839  20.390
                8D WARED  37.001  23.323  20.672

    **Ply 2 (You)**
                    Play    Mean   Stdev Bingo %
    --------------------------------------------
               8C ADWARE  39.689  21.931  19.981
                8H AWARD  38.933  22.196  20.515
               8G ADWARE  40.635  22.082  19.541
               8H ADWARE  39.750  21.416  19.133
                  8H AWA  45.639  25.015  37.041
                  8F AWA  45.594  24.901  36.789
               8D ADWARE  40.212  21.328  18.410
                8D AWARD  37.740  22.550  20.138
               8E ADWARE  38.484  21.037  18.724
                8D WARED  31.642  17.380   9.237


I still need to calculate the overall "equity" for the play; basically the spread difference, but the results seem close to Quackle's; for example with roughly the same number of iterations, Quackle has:

    Ply1 (opp):

    8C ADWARE 39.3219 23.9456 25.2424
    8H AWA    31.5039 21.7387 17.6103

    Ply2 (us):

    8C ADWARE 40.5299 24.0505 22.8965
    8H AWA    46.0031 26.7589 40.4442


A few things I notice here is that I might still have some bugs; my bingo % for opp after ADWARE seems a bit low. I know much of the time it's close to 25% when you are going 2nd, and there are some nice letters for them to bingo through. The mean scores are close though, so I'll investigate more.

In any case, after this my plan is to get multi-core simming working and some sort of graphical logging (for heat maps), and then work on a pre-endgame engine. My endgame engine is finally bug-free, I believe, but quite slow since it's exhaustive. It still should work great for the majority of endgames, but at some point I'll have to rewrite some pieces of it to make it faster.

