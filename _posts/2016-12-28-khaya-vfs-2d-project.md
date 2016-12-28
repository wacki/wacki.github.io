---
layout: post
title:  "Khaya - VFS 2D project"
date:   2016-12-28 10:10:00 +0100
categories: ['gamedev', 'programming']
series: "vfs-2dproject-khaya"
---

I am currently working on a 2D platformer as part of a student team. The project is part of the Game Design course at VFS. I am handling the programming side of things as well as some special effects and UI. The prototype version of the game is done and we should hit alpha in about three weeks. 

At its core the game is about speed and skillful traversal of levels. The twist is an aura surrounding the character that shrinks should the player stand still for too long. If this aura gets too small the player will die. 

<p>
    <div class="inline-image" style="display: block;"><div class="video-container">
        <iframe src="https://www.youtube.com/embed/0v06lCwkuHg" frameborder="0"></iframe>
        </div>      
        <p>Current state of the prototype. Note: Art, animations and audio are still only placeholders for now.</p>
    </div>
</p>

<!--more-->

The movement in its current form feels fun and engaging. However there are still quite a few areas where we need to tweak and balance things. The most difficult being the aura mechanic. 

**How the aura works:** 

If the player is moving at an average velocity larger than a set threshold then the aura will grow / stay at maximum size. As soon as the player falls below that velocity threshold a timer will start after which the aura starts to shrink. If the aura gets too small the player will start taking damage. The aura is also responsible to trigger frozen objects in the level should it come in contact with them. 

There are still a bunch of design challanges ahead of us. Most of them involve giving the player proper feedback for their actions. Especially the aura will be hard to get right in this regard. And of course some smaller tweaking and balancing is still necessary as well.
