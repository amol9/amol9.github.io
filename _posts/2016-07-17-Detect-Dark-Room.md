---
layout: post
title: Detect Ambient Darkness In The Room
tags: bash, webcam, imaging
---

A small bash script to detect if the room is dark. It may come handy for automating a variety of tasks.

e.g. 
 when dark:

 - shut down computer
 - start a maintenance job
 - stop music
 
 or when not dark:

 - start music
 - bring up your dev environment (if you are that kind of person)


{% gist amol9/89579987949bfb0c651e roomdark.sh %}
{: style="font-size:0; /* to avoid the extra true rendered */"}


What it does:

 1. Takes a snapshot using webcam.
 2. Calculates average RGB value of the image.
 3. Compares it to the threshold for darkness.

I have used 0x212121 as threshold. You can customize it as per your taste and conditions.

