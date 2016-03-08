---
layout: post
title:  "Using World Machine to create natural looking terrain"
date:   2015-11-29 21:20:02 +0100
categories: ['gamedev', 'ue4', 'terrain']
---
Here it is, the first real post of the blog and already I’m doing something only somewhat related to my main goal, which is learning UE4. I have a game in mind and I’m not going to go into too much detail here, but basically you’re controlling a large vehicle in a desert environment. Well I thought I could try and create small parts of the game as a way to learn how to use the engine. Another idea would be to recreate a simple arcade game like Space Invaders. I might still do that later on but for now I am interested in the topic of terrains. 
<!--more-->
Terrains in UE4 seem to be a big selling point since they released the [kite demo](https://www.youtube.com/watch?v=0zjPiGVSnfI). And their results speak for themselves. So after doing some research I found out that Epic used [World Machine](http://www.world-machine.com/) for an errosion pass over their heightmaps to achieve a natural look. This is why I figured World Machine would be a good starting point for me. Luckily there is a trial version of World Machine to play around with.

After opening up World Machine for the first time it seemed instantly familiar to me. This is because back when I started to learn openGL, some of my first projects involved procedural terrain generation. And although not being anywhere near of what World Machine can do, the basics are the same. Generate and combine some different noises and run a few erosion passes on it.
After playing around with World Machine for a while I decided to try and create a few simple sand dunes for now and then move on to something different. Below you can see my current state. 


{% include inline-image.html src="2015-11-worldmachine/wmdunes.png" description="This is what I came up with after some trial and error in world-machine" %}
{% include inline-image.html src="2015-11-worldmachine/ue4dunes.png" description="This is what it looks like inside UE4" %}


While looking for terrain examples and stuff other people had done, I stumbled upon [this](http://www.planetside.co.uk/forums/index.php?action=dlattach;topic=18400.0;attach=50771;image) ([source](http://www.planetside.co.uk/forums/index.php?topic=18400.0)). which is much more in the vein of what I want my dunes to look. When I revisit this topic I’m going to use this as a resource. For now I’m happy to have the basics of World Machine down and will move on to other topics.