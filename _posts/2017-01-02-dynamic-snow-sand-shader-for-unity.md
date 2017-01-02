---
layout: post
title:  "Dynamic snow/sand shader for Unity"
date:   2017-01-02 13:10:00 +0100
categories: ['gamedev', 'programming', 'art']
---

A while ago I shared a small project I was working on. I finally had the time to clean up a bit and try a few more things. The goal was to achieve an effect similar to the sand in [Journey](https://en.wikipedia.org/wiki/Journey_(2012_video_game)). Even though in my case it's snow at the moment. 

{% include inline-image.html src="2017-01-snow/snow.png" %}
<!--more-->

The project includes a few example shaders and it's easy to write your own. The effect is achieved by drawing into a height map texture using a stamp or brush texture. A custom material allows for full control over how we overlay that stamp texture. In the shader we have access to the height map pixels as well as the stamp pixels. 


<p>
    <div class="inline-image" style="display: block;"><div class="video-container">
        <iframe src="https://www.youtube.com/embed/REIcwL8hE-w" frameborder="0"></iframe>
        </div>      
        <p>Included stamp shader variants.</p>
    </div>
</p>


I like how it turned out so far, but there are still a few things I want to do:

* Clean up some more, make it easier to use.
* Optimize.
* Make the effect work with Unity's terrain system.

I haven't really thought about how to get this to work with Unity's terrain yet. One option might be to use a seperate tiling system for the snow tiles overlaying the terrain. The shader is already using distance based tessellation, so LOD shouldn't be an issue. However this approach will most certainly lead to stitching issues along LOD seams. This sounds like a completely separate terrain system already, ugh... Maybe I could augment Unity's terrain shader and add some tesselation to it? Then just use a big enough texture atlas for the dynamic height map tiles. I guess I have some research to do.

**Source code:**
{% include github-repo-card.html user="wacki" repo="Unity-IndentShader" %}  


<p>
    <div class="inline-image" style="display: block;"><div class="video-container">
        <iframe src="https://www.youtube.com/embed/WugpdLUP8A8" frameborder="0"></iframe>
        </div>      
        <p>First video about this project.</p>
    </div>
</p>

