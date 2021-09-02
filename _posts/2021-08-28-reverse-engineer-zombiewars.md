---
layout: post
title:  "Detective work on Reverse Engineering an obscure 16-bit Windows game"
date:   2021-09-2 01:30:00 -0700
categories: gaming
---

![zwarsposter1](/public/images/zw-strip.jpg "Zombie Wars poster 1"){: height="auto" width="auto"}

Zombie wars (also known as Halloween Harry 2) isn’t exactly a household name in the video gaming world. Its origins are a confusing mess and despite being a pretty decent game for its time, its own claim to fame is that it is a sequel to Alien Carnage or Halloween Harry. Released in 1995 for Windows 3.1 and DOS, Zombie Wars is a straight action platformer with puzzle elements. LGR did a good [video](https://www.youtube.com/watch?v=at-zjB1KjM8) reviewing the game. It is largely forgotten even by the retro gaming community, there are no fan clubs for it, or community keeping it alive. The company which created it  [Gee Whiz](https://www.mobygames.com/company/gee-whiz-entertainment) ceased to exist by the end of last millenium and the franchise is now abandonware category. That’s a shame because it is a fun and fantastic game and I love it.    
Here is LGR's review of Zombie Wars
[![LGR youtube video on Zombie Wars](https://img.youtube.com/vi/at-zjB1KjM8/0.jpg)](https://www.youtube.com/watch?v=at-zjB1KjM8){: height="auto" width="50%"}

# Zombie wars in my life
I grew up playing this and even without the rose-tinted nostalgia glasses, there is a lot of fun to be found here, even 20 years later. As a kid growing up in the early 2000s in India in a tier-2 city ([Ranchi, Jharkhand](https://en.wikipedia.org/wiki/Ranchi)), gaming avenues were limited for me. There were barely any stores which sold video games and those which did the games were priced as much as someone’s monthly rent. As a result, pirated game CDs were the only way to experience video games. As Gabe Newell has famously said : Piracy is an issue of service, not price.

This game is special because it was my first legally bought game. My sister had gifted it to me on my birthday from her own pocket money, from a music store. It costed `₹ 250` back in 2002 (`₹ 843` in 2021, adjusted for inflation), costly but not as costly as other modern games of the time which costed around `₹ 1500- 2000`. Given, Zombie Wars came out in 1995 and by 2002, original company was gone but still the game was being sold by publishers in India, `₹ 250` was a decent price. Once I got my hands on it, I loved it. I played it again and again, figured out its secrets, level designs etc. The artwork and music were top notch. The sewer levels, the 3d-esque sprites and the view of city skyline in all their pixel art glory, were memorable. I still have the original CD in my parents house though it no longer boots because of decay.

I always wanted to revisit this game and know more about it. The lack of will and experience prevented me. In the last 2 years, I started contributing features (webassembly port, widescreen etc) to Rigel Engine : an open source reimplementation of Duke Nukem II. It is done be [@lethal_guitar](https://twitter.com/lethal_guitar) who did it after reverse engineering the [system](https://lethalguitar.wordpress.com/2019/05/). I wanted to explore something similar for Zombie Wars, find how it works and revive it for the modern times. As said earlier, there are no fanclubs for doing it , so I guess,  I could be that one fan who does it. 😁

![zwarsposter2](/public/images/zw-cv2.gif "Another Zombie Wars poster"){: height="auto" width="50%"}


# Running the game
First step was to get the game to run on modern hardware and OS. It doesn’t run on modern Windows 10 / Windows 11. It doesn’t get installed or It doesn’t run properly (put screenshot of garbled screen). The only suggestion I found on the internet was to use a Dos version of the game in Dosbox which was cumbersome. Also, not all game copies are created equal. I found multiple entries of the game on [archive.org](https://archive.org) but only a couple even ran. Others flat out refused to even run the installer. For those which ran, even garbled, provided a ray of hope. I needed to get it running on modern Windows with the least effort.

I stumbled across otvdm or winevdm. [Otvdm](http://www.columbia.edu/~em36/otvdm.html) is a port of WINE on Windows. This is some next level inception stuff. WINE was created to run Windows software in Linux. And now, it is being used to run old Windows software in Windows itself. 🥴  After days of trying to run the game through various means, following guides, the moment I found otvdm, it booted the game up like magic. No configuration needed. The game ran perfectly, sounded perfectly and worked. I played 2 levels straightway and it was still fun. (post new screenshots). Updated [pcgamingwiki](https://www.pcgamingwiki.com/wiki/Zombie_Wars) entry for it.


![zwarsgameplay1](/public/images/zw-gplay1.png "Zombie Wars running in Windows 11 thanks to otvdm"){: height="auto" width="50%"}

![zwarsgameplay2](/public/images/zw-gplay2.png "Zombie Wars running in Windows 11 thanks to otvdm"){: height="auto" width="50%"}

# Historical context

Now that I had the game running, I wanted to look under the hood : how the game was coded, what kind of assets it has, where I can find the awesome game music etc. Since it is not a popular game, quick Google searches resulted in nothing important.     
The first step I took was to gather as much historical context about the game as possible. When it was made, where it was made, what was the history of its developers, what previous games they made. A lot of good guesses can be made about the technology if you know the historical context.     

![geewhiz](/public/images/gw_logo2.gif "Gee Whiz logo"){: height="auto" width="50%"}

Here, the **Internet Archive Wayback Machine** came handy. The game’s [website](https://web.archive.org/web/19970327033257/http://www.geewhiz.com.au/gwzombie.html) was down even when I got the game in 2002 so I knew the website closed way before that.  I searched and found the [website](https://web.archive.org/web/19970327033257/http://www.geewhiz.com.au/gwzombie.html) existed till 1999 (screenshot from archive.org). There was a lot of cool artwork for Zombie wars (which they called Halloween Harry 2 but never used that name officially). There were a lot of artwork for the planned **Halloween Harry 3** but it never came to fruition. The company also planned to do some new games in 3D in spirit of Monkey Island and Super Mario 64. [Mobygames]((https://www.mobygames.com/company/gee-whiz-entertainment)) provides a better insight there. 

![diane](/public/images/diane-hh3.jpg "Unreleased Diane artwork from Halloween Harry 3"){: height="auto" width="50%"}

Few insights I got from this historical context (eventually, not immediately) :    

1. Halloween Harry 1 (HH 1) was an old game from 80s which was updated and re-released as Alien Carnage in 1993 by Apogee. It is now freely distributed by 3D realms. It means that the tech for HH 2 might not be an extension of HH 1's tech because it was old which seemed to be true later as image formats don’t match, neither do audio formats.

2. The game was developed by 1-2 developers only so expecting a high level language and esoteric formats - non standardized
The game was first released for DOS and then used Wing (Windows Graphics library, precursor to DirectX days) to bring it to *Windows 3.10*. Thus, I could expect a lot of DOS code and standard practices.
The sign on the asset file and their extension is `sub0` : a nod to the previous name of the company which was Sub zero software (and maybe a nod to Street Fighter games ?). Thus, I didn't need to dig around the internet trying to find `sub0` format because it was entirely custom.

3. Which music synthesizer was used to create the midi files - [Yamaha OPL 3](https://www.polynominal.com/yamaha-opl3/) from 1993

# Getting started with reverse engineering
Armed with this knowledge, I started on the journey. I started focusing on tools for reverse engineering dos games. A lot of people have guides and tools for it. [Moddingwiki](https://moddingwiki.shikadi.net/wiki/Main_Page) is a good resource, so is [r/ReverseEngineering](https://www.reddit.com/r/ReverseEngineering/) subreddit.

There are user-made tools which can “unpack” game resources or atleast find out what compression algorithms are used in packing those DOS games (like Universal Unpacker, UPX). In my case, they mostly didn’t work or could find signatures of DOS era popular compression algorithms/software like ASPack, LZEXE, PKLite. Most popular RE tools (decompilers, debuggers, unpackers) don’t support this niche of games - 16 bit Windows game, they either support later 32 bit games or older 16-bit DOS games. Thus, tools were not useful largely. The only hint I got was the format of the game binary through its signature : *NE* or *New Executable* i.e. it is a 16-bit Windows binary (which I already knew because of knowing its historical context). This is pure 16-bit Window 3.10 goodness. 

Then through some modding site, I stumbled across [Wombat](https://www.szevvy.com/) - a multi game asset extractor tool which could extract music , graphics and other assets from a dozen games. It supported Alien Carnage and yes.. Zombie Wars too.  It listed partial support only but better than nothing.  Wombat underestimated its own capabilities : it actually was able to extract almost all the graphics of the game by unpacking GFX.SB0 and audio from SFX.SB0. It only had difficulty with certain sprites and level files (`.h95`, maybe for Harry ‘95 ?) for the game. I knew SB0 was a custom file type but this helped me understand it even better. I got the list of images in [HSI RAW format](http://fileformats.archiveteam.org/wiki/HSI_Raw)

And music files in midi format and wave. I got the Yamaha OPL 3 sf file to play it from 1993, also got the date right because of historical context. . Crazy how a midi sound [changes](https://www.youtube.com/watch?v=JiZOxUhQj10) depending on the audio hardware. A lot of fun.    

Listen this to the peppy [main track ▶️](https://archive.org/details/zombiewars/GAME1.mp3).


The game’s assets are :

* HSI RAW files for images used in cutscenes and dialogues, credits and menu
* SPR sprite atlas files
* WAV files for game’s sound effects
* Midi files for game’s music  
* .H95 level files which it cannot decode

![wombat](/public/images/wombat1.png "Wombat is a multi game asset extraction tool"){: height="auto" width="auto"}

I was excited. I found the treasure of my childhood games which I could play on and on, especially the music. I wanted to know how to do it myself. Wombat is closed source and didn’t provide anything on how it is doing what it is doing (though its script folder provided a lot of hints).  

# Reverse engineering the binary

I knew these packed files were custom and could be compressed, I started with .EXE file ZWARS.exe to see how it is extracting resources. The first step of RE is to find how to extract resources. String utility (https://docs.microsoft.com/en-us/sysinternals/downloads/strings) came very handy. It just dumps out all strings it could find in a binary but that is important. Many compilers leave their name as a string in binary so I could get that information. And yes, I found it. Delphi 2.0  in the strings. Delphi !! That meant it uses Object Pascal with [Delphi IDE](https://en.wikipedia.org/wiki/Delphi_(software)). Success …

My excitement dampened when  I found that none of the modern state of the art RE tools can reverse engineer 16-bit delphi code properly yet. It was kind of sad, I tried IDA Free, NSA Ghidra, Reko and few others to limited success. Even with these limitations, I was able to identify the main game loop. Given that my assembly and binary experience is rudimentary at best, I was very happy with myself. 

I gave up on trying to reverse engineer the binary and started to focus on the asset file. I kind of had the key to solving that mystery, the output from wombat. I just had to reverse engineer how Wombat was reverse engineering and extracting the asset files `.SUB0`.  Wombat has a scripts folder to select a script for each game and its filetype. It had the learnings of the author on how to decode each file. I used it as a reference even though it was incomplete. 
The game creator used a very simple technique to pack the game and reverse engineering it was a lot of fun. HxD is a good binary viewing tool and Wombat helped in verifying my assumptions.    

The asset binary file followed this pattern :
* A signature (12 bytes)
* A list comprising of file names (exact 12 bytes), its offset in memory to access it (4 bytes), its size (4 bytes), 1 bit of unknown data (i don’t know what it is used for but can be safely ignored)
* Ending signature (8 bytes)
* Binary blob to be accessed by offset

Example : if file names are `harry.RAW`, offset 4, size 20 and `diane.RAW`, offset 10, size 20 the file names need to be 12 bytes long. If a filename is shorter than 12 bytes, filler content is used. In this case, harry.raw becomes harry.RAWRAW and diane.RAWRAW. And stored like
* `harry.RAWRAW` [12 in big endian] [20 in big endian] [1 bit of unknown data]

* `diane.RAWRAW` [12 in big endian] [20 in big endian] [1 bit of unknown data]  

* [8 byte of signature to indicate end of the list]

* Binary blog starting with contents of `harry.RAWRAW` starting with signature mhwanh because HSI RAW format start with that sign


The offset as well as size were stored in big endian than usual little endian format. Maybe it was an obfuscation technique ?  It was a surprising discovery which was missing in scripts. Once found, I was quickly able to develop an algorithm to extract it. 
Once the algorithm became clear, I wanted to test it out using my own script. I wrote a python script to extract all the files. Used `deark` to turn RAW images to something else. These are non standard raw so more guesswork was needed but it was done.

You can access the python script here : [https://gist.github.com/pratikone/f0fbd11c3e16a4e852e9c0bbef891b73](https://gist.github.com/pratikone/f0fbd11c3e16a4e852e9c0bbef891b73)

> There are still image formats which I am unable to decode (`.SPR`, `.BIN`) as well as game level files (`.H95`). If interested,  people are welcome to contribute to this open-source script and help in digital preservation of this game.

# My learnings :
This whole project has beena super fun experience. This is my first foray into the world of reverse engineering retro games and I have gained a lot of knowledge. I became aware of specialized tools, modding communities and multiple video game archiving sites. I also got a good historical insight to the state of dos game development in early 90s.   
On technical side of things, my hex reading got stronger 😁. For the future, I will be increasing my assembly code understanding skills to mount another attempt at reverse engineering the game logic. 

I have uploaded all the files to [**Internet Archive**](https://archive.org/details/zombiewars) for digital preservation, including original game files (as they are now abandonware software, you cannot buy them anywhere) as well as game assets, tools and the [ asset extraction script](https://gist.github.com/pratikone/f0fbd11c3e16a4e852e9c0bbef891b73) I have developed.

## Links

* My asset extraction script : [https://gist.github.com/pratikone/f0fbd11c3e16a4e852e9c0bbef891b73](https://gist.github.com/pratikone/f0fbd11c3e16a4e852e9c0bbef891b73)
* My Internet Archive page for Zombie Wars and extracted assets : [https://archive.org/details/zombiewars](https://archive.org/details/zombiewars)
* [http://blog.ssokolow.com/archives/2018/12/02/resources-for-reverse-engineering-16-bit-applications/](http://blog.ssokolow.com/archives/2018/12/02/)
* Modding wiki page : [https://moddingwiki.shikadi.net/wiki/Main_Page](https://moddingwiki.shikadi.net/wiki/Main_Page)
* PCGamingWiki : [https://www.pcgamingwiki.com/wiki/Zombie_Wars](https://www.pcgamingwiki.com/wiki/Zombie_Wars)
* Wayback machine to gee whiz website : [https://web.archive.org/web/19970327033257/http://www.geewhiz.com.au/gwzombie.html](https://web.archive.org/web/19970327033257/http://www.geewhiz.com.au/gwzombie.html)
