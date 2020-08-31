---
layout: post
title:  "Week 7: Upstreaming first code"
date:   2020-07-19 23:00:00 +0200
categories: 
---

This week I was expected to upstream the Jailhouse layer, so I have spent most of the time refactoring it. I moved Raspberry Pi support into dynamic-layers, checked if I have all the dependencies (and not anything else), and simplified the recipe so that it contains just the required steps.

During this week, I was also attending the virtual AGL AMM (All Member Meeting). I have seen many presentations from people around AGL, so I have a better overview of the project and its status and goals. I would like to point out the presentation by Leon Anavi called "Getting started with AGL using a Raspberry Pi". It's unbelievable how much useful information he was able to fit in the 30 minutes. It would help me a lot to see this in the very beginning of my project.

It also motivated me to upstream Waveshare RS485 CAN HAT support into meta-raspberrypi, so I introduced a new variable called CAN_OSCILLATOR to set the MCP2515 crystal frequency and sent a pull request on GitHub. It's currently under review.

But the main goal was to upstream my Jailhouse layers into meta-agl-devel. That is already prepared, but not submitted yet, as I'm having some problem with Gerrit or git-review. I believe that will get solved tomorrow and the layer will be submitted in Gerrit.

Next week I want to look into supporting other memory variants of the Raspberry Pi 4, as Jailhouse currently has configurations only for the 1G RAM version of the board and that is not recommended for use with AGL. 