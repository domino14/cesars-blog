---
title: "2022 is finally the year of Linux on the desktop, and I can prove it"
summary: "Linux provides the best and most cost-effective desktop experience."
date: 2022-05-13T13:42:00-04:00
tags: ["linux"]
featured_image: '/images/IMG_7728.jpg'
---

Back in 2001 when I was at Caltech, my good friend Jonny was running Linux. I don't remember the distro, honestly, but he convinced me to try it, and eventually installed it on my Pentium computer. I don't remember the specs of it either, but it was decent for its time. At the time, I was studying electrical engineering, so I mostly needed the computer to write assembly programs, SSH (or probably Telnet, actually) into campus machines, flirt badly on AIM, and listen to music.

Having just discovered the best band in the world, which was Weezer at the time, I would put the Blue Album / Pinkerton on repeat (and sometimes the Green album) using a Linux media player that looked a lot like Winamp. It might have been XMMS. It was pretty sweet. The main problem, really, was the fact that my music would randomly stutter. Sometimes it was related to what I was doing, sometimes it wasn't. There was no rhyme or reason to it. Jonny seemed to know way more about how to hack into the internals on it, but I don't remember us being able to figure it out. One time I got so frustrated that I decided I had enough and re-installed Windows on the machine. It was almost like I had betrayed Jonny, he was upset for a brief period of time, but then probably understood. I was just not cool enough to run Linux.

![](https://www.xmms.org/screenshots/playlist.png "Pretty sweet, hmm?")


A couple years later a Linux laptop with 2 GB RAM was being sold at our campus bookstore. This was way more RAM than I'd ever had, and significantly more than whatever the Mac/Windows laptops had in them. Actually, even right now, in 2022, 2 GB of RAM in a Chromebook is not the end of the world. I decided to give it another try, as I needed a laptop. Immediately, I was unable to do the most basic things: connect to a network if I remember correctly, bring up the X server (graphical display), and, most egregiously, actually see the 2 GB of RAM. I believe the computer only saw something along the lines of 512MB or maybe 1 GB. This was a long time ago, but when I called whatever customer support line the manufacturer provided, they were unable to help me. Apparently the drivers or whatever was needed to actually access that RAM were out of date. You can imagine the sour taste that Linux left in my mouth. I convinced the bookstore to let me exchange it for a Mac laptop, and since then I've been on team Mac all the way until earlier this year.

Of course, I still used Linux for everything. As a software engineer and onetime startup CTO I became familiar with server-side Linux. A majority of the software we all interact with when we go to a website is likely running on Linux. In general, I love open-source, and the two have almost become synonymous... so I always wanted to use Linux on my desktop again, but kept putting it off.

As a side note, whenever I've set up a computer for my mom, I always put XUbuntu on it. She doesn't even know it's not Windows, as she only really uses it for email / looking at baby pictures / web surfing.

### Apple

An iMac, which I bought in 2017, was my primary desktop computer, but it was beginning to show its age. Everytime I restarted it (or it restarted itself, which happened frequently), it would take over 10 minutes before it was fully responsive. Why? I even tried reinstalling the OS, which did make it faster, but I would still get occasional kernel panics. I could not stream and code at the same time (I like streaming coding sessions on Twitch) without serious performance degradation -- for example, Webpack running in Docker already took like 2 minutes to compile one of my web apps, and when doing this while streaming it was 5 or more minutes. If I had too many windows open it would crash. etc.

![](/images/twitch2.png "Code streaming on Twitch on new Ubuntu PC")


When the Mac Studio was announced I salivated. I had gotten a 2021 Macbook Pro with an M1 chip, and I did not tire of singing its praises. I still don't actually, it's an amazing laptop. But connecting it to an external display and running Docker + OBS, I still see some stuttering. A YouTube video would briefly stutter whenever I switched to another window, even if I wasn't hooked up to an external display. How? (Actually, how? I think it's because Docker was running in the background, which on a Mac involves spinning up a virtual machine, but why can't this state of the art laptop from last year handle it?). The Mac Studio has at least a 2-month backorder time, and the model that I want costs $4000 -- it's probably too expensive.

So, I decided that for my main use cases - developing web apps, writing this blog on open-source software (Hugo!), using Docker, and streaming code/[Woogles](https://woogles.io), that a Linux desktop was the way. Of course, I could also dual-boot with Windows, so that I could play the occasional game.

### Building my own PC

I have always been very intimidated by the process of building a PC. The funny thing is, I'm an electrical engineer by trade, and I *HAVE* built my own PC - from the constituent chips, in a couple of classes at Caltech. One of the classes involved wire-wrapping an 8088 processor to all of its RAM and supporting chips and then programming the whole thing with assembly. I didn't count, but I would estimate I connected more than 1000 pins to each other across many different chips, without an error (I think). In the sequel class we turned it into an audio recorder with a display and buttons. But a modern computer is pretty far away from this. It's actually way easier. With some help on the [r/buildapc](https://www.reddit.com/r/buildapc/) forum on Reddit I was able to put together the following pieces:

- Intel Core i5-12600K 3.7 GHz 10-Core Processor
- Scythe Mugen 5 Rev.C CPU Air Cooler
- MSI MAG B660M Motherboard
- Silicon Power GAMING 64 GB (4 x 16 GB) DDR4-3200 CL16 Memory
- WD BLACK SN750 NVMe SSD 1TB with Heatsink
- Corsair 4000D Airflow ATX Mid Tower Case
- Corsair RMx (2021) 850 W 80+ Gold Certified Fully Modular ATX Power Supply
- GIGABYTE Eagle GeForce RTX 3070 Ti 8GB

The overall total was around $1930 with tax, less than half as much as I wanted to spend for the Mac Studio.

Putting the PC together was **fun**. I haven't done this before, so I watched a couple of YouTube videos, read the various manuals, and spread out all the pieces on a table.


![](/images/IMG_7723.jpg "Building a sweet PC")
![](/images/IMG_7724.jpg "On the top right, I've inserted the processor into the socket, and am hoping I didn't break it")

After several hours, it turned into a PC.

![](/images/IMG_7766.jpg "My first home-built PC")

### Windows

Now, to install Windows. I flashed Windows 10 onto a USB drive and set my motherboard to boot from it, and then...

![](/images/IMG_7728.jpg)

I tried again with Windows 11. Maybe Windows 10 was too old. Nope, same issue. I don't have an optical drive and my motherboard came with a DVD. So I went to the manufacturer's website and downloaded all their drivers onto another USB disk, unzipped them, and tried to have the installer app read from the disk. I tried pointing it at every single one of those unzipped driver files. No dice. Note that this process took many frustrating hours. I even updated the firmware of the motherboard.

I also tried removing the graphics card to see if that was the issue, and I still got the same errors. I never figured out how to progress beyond this screen.

Therefore, it certainly is not the year of Windows. Now, I know this board is relatively new (seems to be from early 2022?) but surely Windows should come with drivers for the older chipsets on it. If not, why coudln't it install them from the USB disk? Do I actually need an optical drive to install Windows in 2022?

### Linux Mint

The next day I tried installing the latest version of Linux Mint but all I got was a black screen. After removing the graphics card, though, I was able to actually install it and booted into the Desktop environment ... except, it can't see my network cards at all. I wasn't able to connect to the Internet, either wired or wireless.

### Ubuntu 21.10

Of course, at this point I am sad. I'm thinking I must have actually broken something during the process of installation. Maybe the hard drive, the memory isn't sitting right, something came unsoldered, I broke the power supply, etc. I tried the latest version of stable Ubuntu (which was 21.10 in early April), and success! I actually booted, and everything works!

Ubuntu is a _great_ user experience now. Everything is fast. The computer itself is insanely responsive and fast. Docker runs like a dream; recompiling my web app frontend takes seconds, I can stream, play music, have a ton of tabs open, and there's never a problem. It's not to say that there aren't a few little (and sometimes baffling) bugs. I'll talk about those later.

For now, the computer meets everything that I want it to do, except for playing the latest games. I don't really play computer games (I'm more of a Switch person) but I do really like Starcraft 2 and was hoping to play it again. I followed some tutorial to get SC2 working on Linux but at the end I couldn't get it to work. The tutorial was written for 20.04 I believe, so maybe something has changed in between.

#### Bugs

There are a few bugs and other weirdness that I've run into with Ubuntu. It's nothing major. Actually, I'd rather have these bugs than be unable to run Windows at all.

- When I install a new app (with Snap) I often have to click it twice to open it. That is, I click it, nothing happens, then after 10-15 seconds or so I click it again. This time it opens.
- I believe I still needed to have the graphics card disconnected in order to get it to initially install. I'm not 100% sure of this, because it may actually also have been an older version of the motherboard firmware that was causing some issues here. However, it detects the graphics card fine now.
- It's unclear what GFX card drivers are running on my computer. See this screenshot:

![](/images/drivers.png)

I know I installed the second one on this list (nvidia-driver-470). But when I do the `sudo apt update` and all it tries to uninstall it. At some point I must have typed in `Y` because I was also trying to get CUDA working and I don't know how that whole ecosystem works. So now, I'm using driver 470 but the computer thinks it's some sort of manually installed driver.

I'd like to use `nvidia-driver-510` but when I tried it the computer is more glitchy / less responsive. Netflix especially stutters a ton. It works perfectly fine with 470 though. Hopefully this can be fixed? Why would this be the case?

- CUDA

So I have a fancy graphics card and no games to play with it. Since I dabble in machine learning, I figured I could start by installing CUDA and then getting PyTorch or Tensorflow or whatever on the computer afterwards. I tried the basic `hello.cu` program which can be found [here](https://developer.nvidia.com/blog/easy-introduction-cuda-c-and-c/) and after compiling it I get the following output:

```shell
cesar@themonolith:~$ ./hello
Max error: 2.000000
```

Yes, 2. It's supposed to be zero. I believe at some point when messing with and installing/uninstalling different versions of drivers, I also got `1.000000` and even `0.000000`. Wtf? This might actually be the biggest issue once I try to do machine learning stuff again. Anyone have any ideas what is happening here?

My plan when I try this again will be to just uninstall and reinstall everything NVIDIA-related and see if that does it.

I'd love to help _fix_ these bugs but the process of developing Linux for right now is very daunting to me. I wonder if I could figure it out.

#### Awesomeness

Using this computer is a dream. Mainly, I just like how _fast_ and responsive it is. I don't think I've ever seen it stutter in doing anything yet, and I also feel very empowered to eventually replace the processor if that ever becomes a bottleneck. I do wonder if I should have gotten an i7 or i9 but this one just hums along perfectly fine.

I tried some benchmarks (mostly Scrabble-related). I have a Scrabble endgame on which I used my [Macondo](https://domino14.github.io/macondo) engine, which takes about two hours on an older Macbook Pro with 16 GB of RAM. On my MBP (which only has 8 GB) I just eventually killed the process, and on my old iMac the program would just consistently crash after 30 or so minutes. Admittedly, this particular endgame is largely RAM-limited, but it also requires a fast processor -- on this computer it only takes 7 minutes.

Doing a Monte Carlo simulation of a mid-game position uses all 16 cores (for some reason Ubuntu thinks I have 16 instead of 10 cores, I think it's counting threads but I'm not going to fight it), and it's very very fast. I can do many hundreds of iterations per second for most positions.

Interestingly enough, the single-core performance is actually slightly worse than that of the M1 chip on my Macbook Pro. I sort of expected that because the M1 chip really is a masterpiece of design. If there was a way I could have put that in this computer, I probably would have. I bet the $4K Mac Studio actually performs better on these benchmarks (but it's twice the cost). But with 10 or 16 or whatever cores I'm ok with it.

#### Proof of my assertion, by contradiction

I said I could prove that 2022 is the year of the Linux desktop, and this proof will mainly have to be done by contradiction. It can't be the year of the Windows desktop, because I can't even get the damn operating system running on this computer, and I tried for hours. And I can't find a Mac Studio to save my life for at least two months, and I bet even the top model's Docker performance wouldn't be as good as what I can get on this system (but that's an unfair benchmark since Docker basically runs at native speed on Linux).

Quod erat demonstrandum. ∎