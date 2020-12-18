---
title: Docker and macOS Sierra
date: '2016-07-17T09:48:00-04:00'
description: Issues using Docker with macOS Sierra beta? Read on for a fix!
---
If you were feeling as adventurous as I was, then you decided to update one of your production Macs to macOS Sierra. That's OK, living on the bleeding edge has its issues, but we have cookies. :)

However, it's been mostly smooth sailing, with the occasional slowness in Safari. A couple days ago, I noticed that after using the Docker for Mac beta for awhile, or putting my Mac to sleep, Docker would stop responding (and I rely quite heavily on using Docker as a replacement for everything, Jekyll, apache, nginx, etc.), and then all the development sites that Docker was server would stop responding.

After doing some searches, I came across this forum post:

[Docker Forums - Docker beta for Mac Does Not Work on macOS](https://forums.docker.com/t/docker-beta-for-mac-does-not-work-and-hangs-frequently-on-macos-10-12/18109/6/)

Apparently, ntpd is interfering with Docker for Mac Beta (and the latest build 19 does not fix the issue). By running the following command, ntpd gets disabled. However, to avoid rebooting my Mac (I _did_ say that I was developing, right?), I used pkill to kill the ntpd and Docker for Mac processes, relaunched Docker for Mac Beta, and was good to go. Keep in mind, in the next macOS or Docker for Mac Beta release, you probably want to enable ntpd in order to see if the bug is fixed (ntpd keeps your Mac's clock synced constantly to an NTP server, that's a very important function of all modern OS's).

Here are the commands you should run, in order:

```bash
sudo launchctl unload /System/Library/LaunchDaemons/org.ntp.ntpd.plist
sudo pkill -9 -f ntpd
sudo pkill -9 -f docker
```

Use at your own risk, don't believe everything you read on the internet!
