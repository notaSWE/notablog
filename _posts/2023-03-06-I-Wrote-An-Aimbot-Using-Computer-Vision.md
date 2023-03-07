---
title: "I Wrote An Aimbot Using Computer Vision"
date: 2023-03-06
---
I built an [Aim Trainer](https://github.com/notaSWE/aimtrainer) in Python earlier this year.  Problem is, my aim is bad...so I decided to cheat.

![Computer Vision Aimbot](/notablog/docs/assets/2023_03_06_aimbot01.jpg "Computer Vision Aimbot")

# Aim Trainer

Before we jump into the details about how I used computer vision, specifically OpenCV, it is important to understand how the aim trainer works.  Essentially the player starts the game, which then initiates a countdown.  During the default 5 second timeframe, red squares are spawned at random in the playing field.  The player is then able to move the mouse cursor over to the target, click it, and remove it from play.  This act mimics aiming in a First Person Shooter (FPS) game, or any game really, that involves aiming with a mouse cursor.

Blahblahblah