---
layout: post
title:  "VR GUI Input Module for Unity (HTC Vive)"
date:   2016-06-04 08:00:00 +0100
categories: ['gamedev', 'unity', 'vr', 'vive']
---

I created a simple GUI input module for Unity to be used in VR. The project includes an implementation for the HTC Vive controllers. 

{% include inline-image.html src="2016-06-vrgui/gui.png" description="Unity GUI working with the Vive controllers." %}


<!--more-->

<br>

I am fortunate enough to be able to use an HTC Vive at my workplace. After starting out in Unity I soon found myself wanting to use Unity's uGUI system in VR. So of course the first thing I did was head on over to Google and check if someone had already done the work for me. Sure enough I came upon a solution by [vrinflux](http://www.vrinflux.com/using-the-vive-controllers-for-unity-4-6-ui-ugui/) who put their results on Github for everyone to use. I liked their solution but I wanted something not as tightly coupled with the vive and that would support more than just two controllers. I am also not too fond of the cursor they used in their project. What I had in mind was more in the line of how the steam VR overlay did things, using a sort of laser pointer as a visual guide. 

{% include inline-image.html src="2016-06-vrgui/steamvr-750x400.jpg" description="SteamVR Steam overlay." %}
  
<br>

***LaserPointerInputModule***  
This is the actual Unity Input Module. It handles GUI interaction and lets *IUILaserPointerControllers* know if they enter or exit a GUI element.

***IUILaserPointerControllers***  
Is the abstract base class for all controllers that want to interact with *LaserPointerInputModule*. Concrete classes have to provide an implementation for button up and down functions. The class also handles the actual laser pointer visualization. 

***ViveUILaserPointerController***  
Is the (currently) only concrete implementation of the laser pointer controller. It uses the haptic pulse feature of the Vive controllers to let the user know whenever he enters a new GUI element.

<br>

<p>
    <div class="inline-image" style="display: block;"><div class="video-container">
        <iframe src="https://www.youtube.com/embed/mD16LejMc9A" frameborder="0"></iframe>
        </div>      
        <p>Here you see the current state of the project.</p>
    </div>
</p>

You can check it out on Github:
{% include github-repo-card.html user="wacki" repo="Unity-VRInputModule" %}  

<br>

I am pleased with the current state of this project. The only thing I’d like to change whenever I get the time is the laser pointer visualization. Currently the *IUILaserPointerController* handles the visualization of the laser pointer. I do this mostly because I require the *LaserPointerInputModule* to tell me the results of GUI raycasts and this way I can just do it all at one point. It would be much better though if I could have a self contained laser pointer visualization component that can do GUI raycasts without having to rely on an Input module. It shouldn’t be that big of an issue actually and when I get the time to do it I’ll update the project.

