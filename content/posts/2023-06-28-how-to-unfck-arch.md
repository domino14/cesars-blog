---
title: "How to unf*ck Arch Linux"
date: 2023-06-28
tags: ["linux", "arch"]
---

Did I tell you all that I run Arch Linux? The following cartoon should make my
intent clearer:

![](https://i.kym-cdn.com/photos/images/newsfeed/002/243/374/ae2.jpg)

Arch is pretty sweet. But to be honest, I've always been the type of nerd who likes building stuff more than fighting with my OS/code/etc etc. So I always choose simple things: Golang, Mac OS, etc. But when I [built my own computer](/posts/2022-05-13-linux-on-desktop/), I finally was able to get everything the way I wanted to: a very fast coding environment where I can still watch Netflix or whatever once in a while, and stream Scrabble tournaments/coding with OBS. I scratch my gaming itch with the Switch most of the times.

That is, until Ubuntu broke again. I don't know what it is about me and Ubuntu. I literally have never been able to upgrade to the next Ubuntu version without my computer becoming unbootable. Never. It must be because I like installing extra packages here and there and something just must not be compatible. I think it had to do with my NVIDIA graphics card driver or who knows.

And then I saw the light. The wonder that is Arch Linux. And I've been a very happy Arch user for about a year now.

HOWEVER! Arch Linux also breaks sometimes! What is the difference, you're asking? There is something about Arch where it always becomes possible to restore your system. If you broke Arch, it's pretty much always something you did wrong, but instead of trying to debug Ubuntu install scripts or whatever, Arch, as a "rolling upgrade" type of system makes it possible to figure out what went wrong on the last update. 

First, some notes on programs and how stuff can break:

### Pacman

Run `pacman -Syu` every few days. This is how you upgrade your system. And **pay attention to what it says**. If there is an error, if it's not able to update the Linux kernel or the initial ramdisk environment, your system will break next time you reboot. Many long-bearded Arch users will figure out what to do before it even boots (for example, they know the magic incantation `mkinitcpio -p`) and they won't mess up their systems like the newb that I am. 

### Yay

Don't use `yay`. It just does too many things behind the scenes. Just take the time to figure out how to install from the AUR yourself; clone the packages, do a `makepkg -si` or whatever. You get more power this way. 

### Rebooting

Arch isn't the type of system you should just keep up for 200 days. You should reboot relatively often. Definitely every time the `linux` kernel updates, but probably for other reasons too. If the kernel updates, for example, your OBS plugins or your USB camera or whatever might stop working, because the USB modules are new. Just like you'd reboot your Windows/Mac computer if the kernel/OS updated, you should do the same for Arch.

Of course, Arch doesn't make you do anything. Things will just not be optimal if you're changing major parts of the system without rebooting.

### My last few Arch breakages:

- Something with my NVIDIA graphics card, the newer version of the linux kernel, and a certain instruction that was not properly supported. I don't remember exactly what the problem was, but there's a Github issue for it and the people on #archlinux (Libera) were able to help me edit my GRUB config to turn off a boot flag. I guess this is the only one that wasn't really my fault, although I should have been able to foresee that NVIDIA sucks and should have forged my own silicon wafers and circuits from scratch.

- I installed `buf`, a program to build protobuf files, using a way that I shouldn't have. It ended up overwriting some system-level `manfile` directories. When another system program had a new update, pacman was unable to install it because the ownership of the directories had changed and then it gave up and quit. It of course printed this, but I ignored it. You can always find the pacman logs in `/var/log/pacman` so you can see after the fact what dumb thing you did and how it broke.

- I unplugged a USB device right as I was running `pacman -Syu` and some modprobe thing was going on/updating (this also showed up in the logs). Somehow this caused the initial ramdisk image to break and then on reboot I was thrown into an increasingly familiar screen of "WTF I CAN'T BOOT, EMERGENCY SHELL COMING UP".

- There might be 1 or 2 more similar things. I had a `yay` thing that was incompatible with something else, and then the something else couldn't update with pacman, and I again ignored the error message and tried rebooting. DON'T DO THIS. Just get rid of `yay` and do it yourself. You'll figure out more easily what is wrong, what packages are incompatible, and install them one by one. It's fine.

### How to fix it

- Keep your install media around. It is your best friend. I installed Arch with a USB dongle. Just plug it in, reboot, and boot from the USB dongle (you should be able to figure out how to do this with your bios).

- Mount your disks. You can find them with `lsblk`:

```
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
nvme0n1     259:0    0 931.5G  0 disk 
├─nvme0n1p1 259:1    0   512M  0 part /boot
└─nvme0n1p2 259:2    0   931G  0 part /
```

You can mount like:

`mount /dev/nvme0n1p2 /mnt`

`mount /dev/nvme0n1p1 /mnt/boot`

Then:

`arch-chroot /mnt` to change root into the new system.

Now you can do whatever you need to do! In the last case for me it's fix my ramdisk. Run `mkinitcpio -p` for example. Another time I had to fix the linux kernel, somehow (always through something dumb I did) it was missing from my /boot partition. `pacman -S linux` does the trick in this case.

Then exit and reboot. Success! Your system should boot. Next time don't do the dumb thing you did. I love Arch Linux, and this is totally not Stockholm syndrome.