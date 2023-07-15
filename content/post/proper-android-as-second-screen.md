---
title: "There's always an open source solution for everything."
date: 2023-03-21T19:43:24+01:00
draft: true
toc: false
images:
tags:
  - productivity
  - ffmpeg
  - rtsp
---

TODO: Add links to all of the app mentions

So, if you are a cheapskate like I am, you probably have wondered "Huh, maybe I could just use an old tablet as my second monitor instead of buying an expensive screen?". Now, what seems like a fairly good idea on the surface becomes a fairly complex task. Not because using a device as a second screen is a hard task to pull off; just trick your computer into thinking you have another monitor, and somehow get the image from that monitor into your other devices.

## The issues
All of the software that I have tried to use this issue has, at least 2 of these issues, if not all of them at the same time:
- **Not cross-platform**: Usually, these kinds of software are built for just one SO. The solution I will propose here should work for all Operating Systems (although, I will only go into detail for the Windows case)
- **Requires a companion app on your Desktop**: This is just annoying, specially since all the solutions I tried have some pretty meh software.
- **Based on VNC**: VNC is a remote control protocol. While it has the upside that using VNC allows touch input from your devices, it has the horrible, horrible downside of being slow as balls, both in terms of latency and specially throughput, as it uses a series of JPEGs rather than a video file, which will compress motion in a way worse way (therefore, VNC makes it impossible to i.e watch a series or a tutorial in your second screen). 
- **No USB or WiFi support**: Most solutions are limited to being either wired or wireless. This is annoying since the use-case for a second screen varies and you may find yourself in a position where you may not be able to use whichever software you had.
- **Cost money**: Why would I want to spend 10$ on an app of dubious quality, when I was only trying to save money originally? lmao.
- **Not compatible with older mobiles**  
And finally, the worst crime of them all: 

- **Propietary drivers from untrusted sources**: At least for Windows, every single solution I have come accross needs one of these. Now, this is a very, very bad thing. Even if these drivers aren't necessary malicious (which we cant know due to their closed nature, anyways, but they could very well be), they can expose us to security vulnerabilities on a driver level, and also cause crashes and system instability in general. Oof.

## The idea
Like I said before, it is actually very simple: 
1. Create a dummy monitor 
2. Get the video of this monitor to our device

For the first point, refer to the [Virtual Display Drivers Knowledge Base](https://github.com/pavlobu/deskreen/discussions/86)
For the second point, the easiest thing I could think of is just using a streaming app. However, since we don't want anyone to know about this, we should install a RTSP server and properly secure it.

## The detailed how-to for my method (for Windows)
1. Install the [ge9 IddSampleDriver](https://github.com/ge9/IddSampleDriver)
   1. Unzip everything to C:\IddSampleMonitor
   2. Install the driver
   3. Modify options.txt
2. Install a RTSP server. For this, I used [MediaMTX](https://github.com/aler9/rtsp-simple-server)
   1. Just unzip the release into a easily accesible path, and run simple-rtsp-server.exe
3. Stream your second monitor somehow...
4. Watch this stream through any app; RTSP Viewer, VLC...

## That looks like a job for FFMpeg
![xkcd 2347: "Dependency"](https://imgs.xkcd.com/comics/dependency.png)
You may have seen this image floating around the internet and thought, "Nah, there's no way the Internet works like that". You would be wrong; there are many examples of this, some being, for example, **dnsmasq**, a piece of networking software that runs on pretty much every server in the Internet.

In this case, the project we are talking about is **FFMpeg**. FFMpeg is a video encoding software known for it's efficiency and it's extendability, which is used by sites like YouTube. According to industry expert John Carmack, ["all the huge companies used the open source ffmpeg in their backends"](https://twitter.com/ID_AA_Carmack/status/1258531455220609025); you can basically expect ANY webpage that handles videos in some way (that includes all of social media) to be using ffmpeg.

So, how extensible is ffmpeg? It is so extensible that we can literally record our screen and stream it at the same time, while specifying every single setting in a single command. 

```
ffmpeg -filter_complex ddagrab=1,hwdownload,format=bgra -c:v libx264 -preset ultrafast -tune zerolatency -r 30 -b:v 650k -an -f rtsp -rtsp_transport udp rtsp://localhost:8554/monitor1

```

So, uh, that looks pretty scary. But it is actually very simple:
- ```-filter_complex ddagrab=1,hwdownload,format=bgra```. Taken from [here.](https://trac.ffmpeg.org/wiki/Capture/Desktop) All this is doing is telling ffmpeg that our input is a desktop capture of monitor 1 (which is actually the second monitor), and that this capture should be made with the Windows API for Desktop capture (DDA) as it is faster.
- ```-c:v libx264 -preset ultrafast -tune zerolatency -r 30 -b:v 650k```. We are specifying our video encoding settings: X264 video, optimized for speed, 30FPS and 650kbps bitrate
- ```-an``` No audio, since I use my PC's audio anyways.
- ```-f rtsp -rtsp_transport udp rtsp://localhost:8554/monitor1```. Stream this to the RTSP server we set up before.

## How does it peform?

### Addressing the problems

- Cross-platform: FFMpeg is cross platform, and there are open source display drivers (or in Linux, simply xrandr) for any platform.
- No companion apps for pairing
- Not based on VNC: RTSP serves video instead of JPEGs, so it is way lighter on bandwidth and you can actually play video with no problem!
- Can be used either through USB or WiFi: Using USB tethering, you can put your PC and mobile in a LAN. Also, using adb you can use "adb reverse" to forward the requests through USB.
- Free, as in beer.
- Free, as in freedom.

### Latency

After some testing, I found out VLC has a 1.5 to 2 second delay, but enough throughput and cache to stream a video in perfect quality and not dropping frames even if the connection got a bit iffy.

Using other apps that let you modify settings, you can lower the cache to a low number and get around 0.5 to 0.7 seconds of lag with some glitching.

I tried using a wired connection (through USB Tethering and adb reverse port forwarding), however using wired connections had no impact on latency, so I'm guessing most of the delay is with the RTSP protocol, viewer, and simply how I structured everything.

## The real world
Keen eyed readers may have clicked on the Virtual Display Driver link before and spotted the fact that it is from a project called Deskreen, which does exactly what my method does but even better:
- Better latency; I don't fully understand the black magic that is going on here; however the Deskreen method is actually 5-7x more responsive than mine, measuring just around 100ms of latency! 
- Better security by default: Fully encrypted, doesn't use predictable URLs and requires the host computer accepting the logins.
- Better compatibility: Only requires the phone to have a browser, instead of a media player, making it usable with literally any device you may have laying around.
- Way more user-friendly: It has an interface, uses QRs for pairing instead of needing to know your device's IPs, and doesn't require you to input a bajillion commands or tinker with low level stuff.
  
So yeah, tl;dr tinkering and discovering everything on my own was very cool, however anyone should just do the dummy monitor thing and then use Deskreen! It's an amazing piece of software, and is open-source while peforming way better than any closed or even paid apps in every respect. :)

