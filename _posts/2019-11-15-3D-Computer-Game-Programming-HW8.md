---
layout: post
title: UI系统
date: 2019-11-15 10:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Unity3D]
---

### UI系统

血条（Health Bar）的预制设计。具体要求如下

- 分别使用 IMGUI 和 UGUI 实现
- 使用 UGUI，血条是游戏对象的一个子元素，任何时候需要面对主摄像机
- 分析两种实现的优缺点
- 给出预制的使用方法

##### 演示视频

<video id="video" width="500" height="300" controls="controls">
        <source src="https://bentsai7.github.io/assets/assets/演示hw8.mp4" type="video/mp4">
  </video>

##### IMGUI实现

`IMGUI`的实现比较简单，利用的是`HorizontalScrollbar`来表示血条，`Mathf.Lerp`通过插值裱花控制血条的变化快慢，使得血条变化较为平滑。

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class IMGUI_HP : MonoBehaviour
{
	private float HP = 0.0f;
	private float newHP = 0.0f;
    void OnGUI()
    {

		if (GUI.Button(new Rect(30, 60, 100, 50), "Add", new GUIStyle("button") { fontSize = 15 }))
		{
			newHP = newHP + 0.1f;
			if (newHP > 1) newHP = 1f;
		}
		if (GUI.Button(new Rect(30, 120, 100, 50), "Sub", new GUIStyle("button") { fontSize = 15 }))
		{
			newHP = newHP - 0.1f;
			if (newHP < 0) newHP = 0f;
		}
		HP = Mathf.Lerp(HP, newHP, 0.01f);
		GUI.HorizontalScrollbar(new Rect(30, 30, 200, 20), 0.0f,HP, 0.0f, 1.0f);
	}
}
```
##### UGUI实现
`UGUI`的实现与`IMGUI`类似。只不过`UGUI`是通过在物体上绘制`Canvas`和`Slider`实现的，通过调整`Slider`的`value`值控制`FillArea`的大小以实现血条效果。我们使用一个`Cube`作为物体，为其绘制血条，具体配置如下。

![1572161490962]({{site.baseurl}}/assets/assets/1572161490962.png)

![1572161504929]({{site.baseurl}}/assets/assets/1572161504929.png)

![1572161594960]({{site.baseurl}}/assets/assets/1572161594960.png)

此外我们还增加了一个脚本用于控制摄像头旋转时，血条始终面对着摄像头。

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class UGUI_HP : MonoBehaviour
{
	private float HP = 0.0f;
	private float newHP = 0.0f;
	// Start is called before the first frame update
	void Start()
    {
        
    }

    // Update is called once per frame
    void Update()
    {
		transform.rotation = Camera.main.transform.rotation; //保持面对摄像头
		GetComponent<Slider>().value = HP * 100;
	}

	void OnGUI()
	{
		if (GUI.Button(new Rect(900, 60, 100, 50), "Add", new GUIStyle("button") { fontSize = 15 }))
		{
			newHP = newHP + 0.01f;
			if (newHP > 1) newHP = 1f;
		}
		if (GUI.Button(new Rect(900, 120, 100, 50), "Sub", new GUIStyle("button") { fontSize = 15 }))
		{
			newHP = newHP - 0.01f;
			if (newHP < 0) newHP = 0f;
		}
		HP = Mathf.Lerp(HP, newHP, 0.01f);
	}
}
```

##### 优缺点
IMGUI的优点是使用方便，无需绑定物体，渲染快，兼容传统的UI渲染，缺点是不支持可视化开发，难以调试。

UGUI的优点是所见即所得，屏幕自适应。缺点是需要Canvas渲染，如果血条太多可能渲染较慢。

##### 预制件制作

对于IMGUI，由于其只需要一个脚本，那么将脚本绑定在Empty物体上，就制成了预制件。

而对于UGUI，这需要把Canvas支持预制件，使用时拉出来作为需要有血条的物体的子对象就可以了。

![1572162132117]({{site.baseurl}}/assets/assets/1572162132117.png)


