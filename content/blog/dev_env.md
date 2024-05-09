+++
title = "My Development Environment"
+++

## Background
I'm not a professional developer (which will be immediately obvious if you ever read some of my code). As such, I like to use tools that prevent me from doing something stupid, as much as possible.

Additionally, I split my development time between Windows and Linux machines; therefore, I prioritize tools that are cross-platform as much as possible.

With those two facts in mind, here are the tools I use, and some thoughts on why:

### Linux Laptop: System76 Lemur Pro
I found Pop!_OS a while ago and really like it. System76 is the company that maintains Pop, and using their laptops just removes any pain points with making sure that it works seamlessly with the hardware. It's also very light and quite powerful!

### Custom Keyboard: Cyboard Imprint
In 2020, I lost part of my right index finger in a woodworking accident. This makes typing on a normal keyboard pretty annoying, although not impossible. In fact, I didn't switch for several years afterward (mainly because I didn't really know that custom keyboards were even available).

Cyboard is a small company that makes keyboards custom-designed to fit your specific hands - when you order one, you send in a picture of both your hands (next to a credit card for scale), and they use their own proprietary fitting algorithms to design a keyboard that is, in theory, perfectly fitted for your hands.

I can confirm that, in my case at least, it works swimmingly. I can actually reach the keys on my right index finger!

Getting a bespoke keyboard has also allowed me to design custom firmware for the keyboard (based on the open-source QMK project), which allows me to do all sorts of cool things like create an alternative key layout, use custom key combinations, and other cool features. I'll do a blog post in the future about my key map.

### Mouse: Ploopy Thumb
I started using thumb trackballs several years ago. As a civil engineer, I spend a lot of time doing CAD (computer-aided design, aka drafting). I noticed that, after a long CAD session (which consists of a LOT of precision mouse work), my right hand would be exhausted, and my shoulders would be tense and painful. Switching to a thumb trackball got rid of both issues almost instantly.

For a long time, I used Logitech devices. The Ploopy though has two advantages:

- it has an extra button on the right side (under the pinky)
- it runs on QMK, meaning it can be extensively customized

While the customizations aren't as extensive as what I have on my keyboard (fewer buttons, after all), they still make a big difference to productivity. I'll do a future blog post about my CAD setup, which is a little wacky!

### Linux Distro: Pop!_OS
I like Pop!_OS for several reasons. One, while I do like tweaking things in general, I had to try to limit my own madness somehow or I'd never actually do anything productive. Pop!_OS is an Ubuntu-based distro that works smoothly out of the box, and doesn't need any tweaking to be perfectly functional for most use cases.

Two, the developers of Pop!_OS, System76, are one of the few companies out there that make Linux-first machines. Pop!_OS is their baby and they work to ensure that their hardware runs it flawlessly. By combining a System76 laptop with their own distribution of Linux, I minimize the occurrence of bugs.

### Terminal Emulator: WezTerm
This is one thing that I've gone back and forth about repeatedly. There are plenty of worthy terminal emulators out there, and I've tried a lot of them. The Windows Terminal app is actually criminally underrated, in my opinion; Alacritty is a great emulator; but I settled on WezTerm for one main reason: **I really love using tabs and panes.** Alacritty and zellij make a fantastic combo in Linux systems, but zellij currently isn't available on Windows, so if I were to use them, I would either need to use a separate app (and separate config) on my Windows machine, or I would have to get used to never using tabs/panes while working on Windows. I wasn't willing to do either of those things, and WezTerm is - to the best of my knowledge - the only terminal emulator out there with in-built tab/pane support that's also cross-platform.

I will say that there is currently an open PR on GitHub where an enterprising individual is working on making zellij work on Windows. If this project ever comes to fruition - and it seems like it will, the zellij maintainer has been supportive - I will probably switch back to Alacritty/zellij.

### Shell: Nushell
Nushell is the bomb. The developers made what I think was an excellent choice, and explicitly threw POSIX compatibility out the window from the start. This allowed them to essentially start from scratch, and ask the question: what do we actually want out of a shell?

It's fast. It's got modern syntax, unlike bash. It's typed. It's got built in support for a whole grab bag of features that make working with any sort of data in the shell so much smoother - from table operations, to modules when programming, to smooth and consistent piping of operations.

After using Nushell for a while, I don't think I can ever go back to any other shell.

### Text Editor: helix
Like I said: I love tweaking. When I used neovim, I spent hours tweaking my config; not because I needed to, but because it was **fun.** As a hobbyist, it's a real treat that I usually am not on a time crunch to get something done; I can spend as much time as I want playing with my system.

But...I still do want to get *some* things done. helix is a great editor because it has such sensible pre-configured defaults. There's no package manager because there's no plugin system; helix does what it needs to do to be useful from the start.

Dealing with LSPs (a must for someone like me, an idiot who still wants to write code) is absolutely painless. Is the LSP installed? Is it on PATH? It's working in helix.

### Windows-Specific Tools
As much as I try to share my config and tools across ecosystems, there are a few areas where my setups do diverge. I have two Windows-specific tools installed: Flow Launcher, and QtTabBar. Flow Launcher is a great emulation of one of my favorite Linux features: pressing a hotkey, typing in a search query, and having the system launch your selected program when you hit Enter. Unfortunately, due to underlying limitations in Windows itself, you can't start the launcher by pressing the SUPER key, as you can on a Linux system; but other than that, it works great. You can register custom query strings - i.e., type "aaa" and have it automatically type out a long search string for you.

The other that I use is QtTabBar - it enables tabs for File Explorer. Because this feature is actually included in Windows 11, once I finally upgrade this one will go away.
