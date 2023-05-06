---
layout: post
title:  "Disabling the power button"
date:   2020-09-02 00:00:000 +0200
categories: blog
---

I had a recurring issue with my laptop: when it would enter sleep mode I would attempt to wake it by pressing on the power button. I would wait 1 or 2 seconds and after seeing that the display wasn't lighting up, I would press the power button again. At this point my laptop would come out of hibernation and immediately shut down potentially losing unsaved work.

<!--more-->

An easy remedy for this was simply to disable the power button altogether by editing /etc/systemd/logind.conf and setting `HandlePowerKey=ignore`. I rarely use the power button to shutdown my laptop since I usually run `poweroff` in a terminal anyway.