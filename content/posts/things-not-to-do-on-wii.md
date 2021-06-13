---
title: "Things not to do on a Wii"
date: 2021-06-12T18:11:28+02:00
draft: false
toc: true
images:
tags:
  - wii
  - homebrew
  - linux
  - projects
---


During the last few days, I've gone on a trip down a *very deep* rabbit hole: trying to find things to do with the Wii that I have just lying around.

I documented my findings, and reviewed everything I tried, **just so you won't have to go down the same hole as I did.**

## Requirements


- A SD card, at least bigger than 1 GB but higher is recommended. SDXC is not compatible.
- An USB keyboard.
- As a recommendation, an USB stick that you don't mind not using for a while to use it as a swap device. Alternatively, create a swap partition on your SD (slower)

## Homebrew
Okay, first things first: Homebrew is, basically, **the only thing in this list that I recommend**.   
It's not only very easy and beginner-friendly, but also useful in the sense that it lets you load any useful application that you want (be it ripping your Wii games, using an USB loader, playing online or hell, just having a media player on your Wii).  
I recommend you to follow **[wii.guide](https://wii.guide/get-started)** with the **Letterbomb method**, which is not only super well-documented but also one of the easiest methods.     
In case you're worried about bricking: *it won't, trust me, doing this is very safe!*.

Final verdict: **Do it!** It's extremely useful and easy.
## Linux
After homebrewing your Wii, getting Linux installed on it is as easy as downloading the last **[wii-linux-ngx](https://github.com/neagix/wii-linux-ngx)** release and flashing it on a SD card with **[balenaEtcher](https://www.balena.io/etcher/)**. Also, **you should  boot into a Linux system or a GParted LiveCD to change the size of your newly created Linux system partition** before proceeding, since it is only 300-and-something MB by default, and also **format or create a linux-swap partition in your USB** .  
After this, **connect your USB keyboard and your USB drive, and then insert the SD card into your Wii**. Turn it on and launch the Homebrew Channel, then press the Home Button on your Wiimote. Select "Launch BootMii" and you'll be greeted by a GRUB-like Bootloader. Hold the RESET button on your Wii for around one second, and you'll have successfully booted into Linux for the first time! However, the setup hasn't ended yet.

First thing I recommend doing is **activating swap space**, since the Wii only has ~70MB of RAM. Run **`fdisk -l` and identify your USB's linux-swap partition**, and then modify **`/etc/fstab`** to reflect where it is. Finally, run **`swapon -a`**.
You probably want network access, so next thing to do from here, is navigate to **`/etc/network/interfaces.d`** and create a **wlan1** file with nano, and insert the following content:
```
auto wlan1
iface wlan1 inet dhcp
        wpa-ssid YOUR_SSID
        wpa-psk YOUR_PASSWORD
```  
and then run **ifup -a**  
Note: wii-linux-ngx includes a "easy network config" script, however it doesn't work as it will create the interface wlan0, which is not correct  
Note 2: If this doesn't work out of the box, check [this](http://www.gc-linux.org/wiki/WL:Wifi_Configuration#Debian_configuration).

After this, edit your **/etc/ssh/sshd_config** file with **``PermitRootLogin true``** since we only have a root account in this setup.  
Now we can SSH and SFTP from our computer, which makes working with this way easier.

Finally, the last thing we need to do to complete a basic, working Linux setup is to update our **/etc/apt/sources.list** file to this content:
**``deb http://archive.debian.org/debian jessie main``**

Final verdict: Try doing it!  You don't lose anything, though I have yet to find something actually useful to do.
## Minecraft server
So... Minecraft server. Minimum requirements are 512MB, so it must surely not work on the Wii right? Well, it... kinda does? It all depends on what your definition of "working" is!  
So, I opted for running a **[Paper 1.8.8 server](https://papermc.io/)**, which is one of the fastest server forks you can run.   
I modified the config to the minimum settings I could think of: **a view distance of 4, disabled timeouts, no debugging, no Spigot timings, every single Spigot and Paper setting that increases performance**. The result? A frankenstein configuration that probably breaks every single redstone contraption and gameplay element of Minecraft. However, it should also be the most performance that I can squeeze out of this thing. I also ran **[WorldBorder](https://dev.bukkit.org/projects/worldborder)** to pregenerate the world, because generating it on the Wii would most likely take several hours.
Finally, to install this on your Wii:

1. Push the server folder to your Wii through SFTP.
2.  Add this to your **sources.list** ``deb [check-valid-until=no] http://archive.debian.org/debian jessie-backports main``
3.  Get Java 8 with ``apt-get install -t jessie-backports openjdk-8-jdk-headless`` and pray you don't get as many errors as I did...
4. **You should now be able to run the Paper server like any other Minecraft server!** (I recommend using nice to speed it up a bit): ``nice -n -10 java -Xmx2G -jar paper-1.8.8-443.jar``

You might be wondering about the results... Well:

- **Around one hour just to launch the server** (I'm guessing most of this is JVM memory allocation, which, since it is running completely on swap space is slow as hell)
- **It took three attempts to successfully connect**
- **Around 2-3TPS with only one player.**  ![TPS](https://jos.s-ul.eu/6tWPLaMm)

In case you want to see this monstrosity in action, here's a video:

[![Minecraft gameplay on a Wii Server](https://img.youtube.com/vi/Fe5foaQHgvw/0.jpg)](https://www.youtube.com/watch?v=Fe5foaQHgvw)

Final verdict: **Do NOT do this.** It's a seriously stupid idea.

## Wiihole
I thought to myself that maybe *-just maybe-* I could use my Wii as a DNS server to get network-wide adblocking without having to spend $$$ on a Raspberry Pi (I *do* know that they are super cheap but I am a tremendous miser at times)

While getting Pihole to run on your Wii is not as easy as doing it on an actual Raspberry Pi, it's still fairly straightforward.


1. Set your Wii's IP to be static through your router.
2. Pre-install the dependencies (they won't install with the Pihole script because the Debian Jessie repo keys are expired) ``apt-get install curl dnsmasq dhcpcd5 git dnsutils libsqlite3-0 libidn2-0 libperl4-corelibs-perl lsof dns-root-data psmisc sqlite3 unzip idn2 netcat``
3. Get **Pihole 3.3.1** from GitHub. This is the last supported version. ``curl -https://codeload.github.com/pi-hole/pi-hole/tar.gz/refs/tags/v3.3.1 > Pihole.tar.gz``
4. Unzip it ``tar -xf Pihole.tar.gz``
5. Go into the directory that has been just created and run ``PIHOLE_SKIP_OS_CHECK=true bash pihole``
6. In the Pihole installer, **turn off Web interface and logging**. We need to maximize performance as much as possible.
7. Set your computer (or your router's DNS) to the Wii's IP.

So, what are the results? Well... I tried it with Namebench, and I was even more disappointed than I thought I would be.  
![DNS Performance Graph](https://jos.s-ul.eu/rhs6zC0p)  
So, in average, it takes around ~4-5 times more to resolve a DNS query with the Wiihole than with a standard DNS like your ISP's or Google DNS

Final verdict: **Do NOT do this.**

## Conclusions
You could possibly improve the performance of this by **using ZSwap and ZRAM** because the Wii's 70MB of RAM are the biggest bottleneck (whatever you try to do, due to the scarcity of RAM you'll be constantly accessing  swap space which is super slow). However, **neither ZSwap nor ZRAM are included in wii-linux-ngx**.

You could also probably improve Wiihole's performance by **using a Wii ethernet adapter** and **by using a newer Pihole build that uses FTLDNS** (however, you need to compile this yourself, and the debian repos lack some of the necessary dependencies to do so).

Still, the Wii is old and was already underpowered by the date it came out (as Nintendo consoles usually do), **there just isn't much to do**. I'll probably keep trying to find a good purpose for Linux on it, but for now, I recommend you to just homebrew it and marvel at **how well optimized Wii games were to run well on this kind of hardware**.