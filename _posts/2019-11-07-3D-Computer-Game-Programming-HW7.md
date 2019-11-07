---
layout: post
title: 粒子系统
date: 2019-11-07 10:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Unity3D]
---

## 粒子系统
### 完善官方的“汽车尾气”模拟：

使用官方资源资源 Vehicle 的 car， 使用 Smoke 粒子系统模拟启动发动、运行、故障等场景效果

从左到右分别为故障，发动，运行，的smoke场景效果

**演示视频如下**

<video id="video" width="500" height="300" controls="controls">
        <source src="https://bentsai7.github.io/assets/assets/演示hw7.mp4" type="video/mp4">
  </video>
主要通过修改粒子的各个部分来实现不同场景下smoke效果的不同

![1571634176766]({{site.baseurl}}/assets/assets/1571634176766.png)

这里可以通过修改Duration来控制粒子的喷射周期，通过Looping设置是否循环喷射，通过 Start Lifetime和Start Speed分别控制粒子的生命周期和喷射速度。通过Start Color来设置粒子颜色。通过设置MaxParticles来限定一周内发射的例子数,多与此数目停止发射。

![1571634354986]({{site.baseurl}}/assets/assets/1571634354986.png)

Shape属性板用于控制喷射的范围
球体(Sphere) 半球体(HemiSphere)圆锥体 Cone, 盒子(Box)网格(Mesh) 环形(Cricle) 边线(Edge)。
由于这里是汽车尾部气体，我们选用盒型的喷射器模拟汽车尾部粒子效果的形状。



![1571634392942]({{site.baseurl}}/assets/assets/1571634392942.png)

Emission属性板用于控制喷射的速度。如果汽车要产生气体状的尾气，则将Emission rate的速率稍微调低。如果要产生喷射状的火焰效果，则将Emission rate的速率调高。

![1571634532587]({{site.baseurl}}/assets/assets/1571634532587.png)

Renderer属性板用于控制渲染的效果。如果这里要产生黑烟或者火焰，则选取对应的smoke贴图将其作为material属性，以更改喷射效果的贴图。

![1571634683526]({{site.baseurl}}/assets/assets/1571634683526.png)

Size over Lifetime和Color over Lifetime属性板用于控制粒子随时间变换的颜色和大小。