---
title: Running classic Windows game on modern MAC OS using wine
date: 2016-11-17T00:00:00+08:00
tags: ['classic game', 'diablo 2', 'wine', 'Windows', 'Mac']
type: post
---

It is relatively straightforward. All you need is __[wine](https://www.winehq.org/)__, which is a free open source software that allows you to run Windows applications (\*.exe files) on unix systems (Linux, Mac OSX etc).

In this post, I’m going to use the classic game Diablo II as an example. The latest patch of Diablo II is 1.14d, which can function on modern Mac OS. However, the experience is very awful. The most annoying thing to me is that the mouse cursor freezes for ~0.5 seconds whenever you open your character screen or quest log screen.

Eventually, I gave up on the Diablo II Mac client, and instead, using wine to run Windows client on Mac.

To do this, you need to be familiar and comfortable with using command line (Terminal app).

First, you need to install wine. This is an excellent post: __[Installing Wine on Mac OS X](https://www.davidbaumgold.com/tutorials/wine-mac/)__. Then, get Diablo II and Diablo II LOD Windows client from Battle.net. After that, go to the client folder in Terminal, and type:

```bash
wine Installer.exe
```

From now on, everything is like Windows, just follow the instruction, and install the game in C:\Program Files. When you do that, the game is actually located at `~/.wine/drive_c/Program\ Files/Diablo\ II/`. If you want to play runeword mode, download the runeword mode patch from __[here](http://www.chrisclarke.co.uk/D2stuff/PDFs/Mac%20Files/)__. Make sure, you download 1.13d RWM.zip.

Unzip it, and put the folder (Runewords) into your Diablo II folder. Now, to play runeword mode, you need to go to the Runewords folder to run the game, like this:

```bash
cd ~/.wine/drive_c/Program\ Files/Diablo\ II/Runewords/
 
wine ../Game.exe -direct -txt
```

If everything goes right, you should be able to play Diablo II. However, to this stage, the graphic is not very satisfying. One solution is to use Sven’s glide-wrapper. Go to download the latest __[glide-wrapper](http://www.svenswrapper.de/english/downloads.html)__. Again, unzip it into your Diablo II folder.

To configure the glide-wrapper, go to you Diablo II folder, and run glide-init.exe:

```bash
cd ~/.wine/drive_c/Program\ Files/Diablo\ II/
 
wine glide-init.exe
```

Then, the configuration window will pop up, and click “settings” at the left hand side, tick things like this:

![](/images/000/glide1.png)

If you want to play window-mode, simply tick it. Then click “Extensions” at the left hand side, and tick the options like this (the following options work the best for me, but it might be dependent on machines):

![](/images/000/glide2.png)

After that is done, simply click “Quit” at the left hand side.

Now, to run the game with rune word mode and glide-wrapper, run the game like this:

```bash
cd ~/.wine/drive_c/Program\ Files\Diablo\ II/Runewords/
 
wine ../Game.exe -direct -txt -3dfx
```

In the game, check the graphic by typing `/fps` in the dialogue, then you should see that the top right corner shows `Video: Glide`, meaning it is working. If you still see `Video: DDraw`, the glide is not configured properly.

![](/images/000/screenshot011.jpg)

Have fun!
