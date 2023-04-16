---
title: "Change Keyboard Layout iPad"
date: 2023-04-15T22:50:19-06:00
lastmod: 
draft: false
tags: ["howto", "ios", "ipad"]
categories: ["tech"]
---
Although I mostly use the genius [EurKey](https://eurkey.steffen.bruentjen.eu/) Keyboard Layout on my laptop, I frequently switch between layouts, when typing in German for longer texts. Furthermore my wife prefers the Spanish/Latin American Layout. That's why I frequently have to switch between those three keyboard layouts. Fortunately it's quite easy to do on my laptop. By pressing `CTRL + Super` I toggle through my selected layouts and receive feedback by a little country flag (🇦🇹) or by a short letter code (`lat` or `eur`).

Lately I used my wifes iPad for writing stuff and found myself in the trouble of having to long press `S` to be able to write the German letter `ß`. However sometimes the letter was right there in the first layer. Other times it wasn't there, but the Spanish `Ñ` was in the first layer of the layout. I couldn't quite make out a pattern. I knew that in the setting for the virtual keyboard layout English, German and Spanish were set. However I never had found out how to actually change the layout at will.

Since I normally only used it for passive consuming I never had the urge to find out. Until now. So I set out for searching the Internet for a solution, but never found a satisfying solution. To make things short, I recorded this little video/gif to showcase how to switch between the set layout: press and hold the little world symbol on the virtual keyboard and then slide up to select the desired/needed layout.
![Switch Layout on the iPad Keyboard](/img/ipadkeyboardswitcher.gif#center)

**Edit**: I since found the right page on [Apple Support](https://support.apple.com/guide/ipad/add-or-change-keyboards-ipad1aa5a19a/ipados). Searching stuff really became a hit and miss thing.  
Anyways, while creating the little gif above I once again was amazed at how easy it is to find the right tools for quite anything on Linux. I recorded the original video on the iPad itself (the tool is included out of the box! I still remember how hard it was many years ago to find a working tool on Windows to do just that.) and then manipulated it in Avidemux, since it somehow got rotated by 90 degrees and i wanted to show the keyboard only, so crop and rotate. Here I found out, that one has to first [select a video decoder](https://www.avidemux.org/admWiki/doku.php?id=using:main_window_-_qt_version#audio_video_sidebar) to be able to apply filters to a video, since without them the `Video->Filter` option is greyed out. And then I [used ffmpeg to create a gif](https://superuser.com/questions/556029/how-do-i-convert-a-video-to-gif-using-ffmpeg-with-reasonable-quality) out of the `.mkv` file with the following command:  
 `ffmpeg -i ipadkeyboard.mkv -vf "fps=10,scale=640:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" -loop 0 output.gif`
