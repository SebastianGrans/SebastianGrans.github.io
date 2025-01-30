---
layout: post
title: Rigging industrial robots in Blender
img: /images/20201023-Blender-rigging/fancy.png
---

<style>
.mono {
  font-family: monospace;
}

</style>


<center>
<img src="{{ site.baseurl }}/images/20201023-Blender-rigging/fancy.png" alt="KUKA Agilus rigged in Blender" style="display: block;"/>
</center>

<div markdown="1" style="display:block; background-color:#eff0f1; border-radius: 3px; padding: 5px 20px; margin: 10px;">
**TL;DR:** The Blender file is available on my [Github](https://github.com/SebastianGrans/KUKA-Agilus-in-Blender).
</div>

I've recently started to use Blender in my research and it is just really fun tool to play with! 

After finding [this](https://machinekoder.com/animating-industrial-robots-with-blender/) post by Alex Rössler where he rigged an industrial robot in blender, I got inspired to replicate it with a robot I have access to. One of the robots we have is a [KUKA Agilus](https://www.kuka.com/en-de/products/robot-systems/industrial-robots/kr-agilus), in particular the KR6 R900 sixx model. Meshes and URDF files for this robot (and many other common industrial robots) is available at the [ROS-Industrial](https://github.com/ros-industrial) Github page.

As Alex suggested in his post, I watched [this](https://www.youtube.com/watch?v=eF4CuIX40XE) rigging tutorial. I also found [this](https://www.youtube.com/watch?v=1-5ZR45y9RM) and [this](https://www.youtube.com/watch?v=S-2v_CKmVE8) video useful.

Except for the URDF file, I also used the [specification sheet](https://www.kuka.com/-/media/kuka-downloads/imported/6b77eecacfe542d3b736af377562ecaa/0000205456_en.pdf) provided by KUKA as a reference for joint limits.

There are a ton of rigging tutorials out there, but none of them focuses on rigging of industrial robots. So I might write a blog post or make a video tutorial on it in the future.



<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/ProD6t-96L4" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>