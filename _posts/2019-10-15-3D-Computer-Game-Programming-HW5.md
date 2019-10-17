---
layout: post
title: 物理系统与碰撞
date: 2019-10-15 10:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Unity3D]
---
## 物理系统与碰撞

### 1、改进飞碟（Hit UFO）游戏：
**游戏内容要求：
按 adapter模式设计图修改飞碟游戏
使它同时支持物理运动与运动学（变换）运动**

演示如下：可以看到飞碟多了抛物线运动以及相撞的效果。

<video id="video" width="500" height="300" controls="controls">
        <source src="https://bentsai7.github.io/assets/assets/演示hw5.mp4" type="video/mp4">
  </video>

![1569156878849]({{site.baseurl}}/assets/assets/1569156878849.png)

之前我们实现的飞碟游戏的运动是建立在`CCActionManager`的基础上的，是非物理学变换运动。而如果要实现新的物理学变换的动作管理器，又不改变删除原有的非物理学变换运动动作管理器，可以通过适配器模式来实现。

![1569157824274]({{site.baseurl}}/assets/assets/1569157824274.png)

> 适配器模式（Adapter）的定义如下：将一个类的接口转换成客户希望的另外一个接口
>
> 这里使用的是类适配器模式，即可定义一个适配器类来同时继承当前系统的业务接口和现有组件库中已经存在的组件接口。
>
> 适配器模式（Adapter）包含以下主要角色。
>
> 1. 目标（Target）接口：当前系统业务所期待的接口，它可以是抽象类或接口。
> 2. 适配者（Adaptee）类：它是被访问和适配的现存组件库中的组件接口。
> 3. 适配器（Adapter）类：它是一个转换器，通过继承或引用适配者的对象，把适配者接口转换成目标接口，让客户按目标接口的格式访问适配者。

根据以上定义，首先在原有代码的基础上增加一个`PhysisActionManager`动作管理器类，同时其使用的动作类为`PhysisActionMove`动作类。通过刚体`Rigidbody`组件为游戏对象提供物理属性，让游戏对象在场景中可以受到物理引擎的作用。当游戏对象添加了`Rigidbody`组件后，游戏对象便可以接受外力与扭矩力。具体实现如下：

> 首先定义要实现的接口类IActionManager，以及其主要接口`public virtual void playDisk(DiskData disk)`。该函数接受一个DiskData类，根据IActionManager自身是否为物理运动实现，来调用不同的动作，加入动作管理类中，实现不同的飞碟飞行方式。

```c#
public class IActionManager : SSActionManager, ISSActionCallBack
    {
        public FirstController sceneController;

        public new void Start()
        {
            sceneController = (FirstController)SSDirector.GetInstance().CurrentSceneController;
            sceneController.actionManager = this;
        }

        protected new void Update()
        {
            base.Update();
        }

        public void SSActionEvent(SSAction sourse, SSActionEventType events = SSActionEventType.Completed)
        {

        }

        public virtual void playDisk(DiskData disk)
        {

        }
    }
```

>接着实现PhysisActionManager来是实现接口，其 playDisk函数通过创建一个PhysisActionMove并将其绑定在disk对象上，并加入动作管理器中来实现disk对象的物理运动。
```c#
public class PhysisActionManager : IActionManager
    {
        
        public override void playDisk(DiskData disk)
        {
            PhysisActionMove action = ScriptableObject.CreateInstance<PhysisActionMove>();
            action.disk = disk;
            base.RunAction(disk.obj, action, (this));
        }
    }
```

>PhysisActionMove设置物体的随机位置，同时加入刚体组件，设置物体受到的恒定力，使得物体可以做物理抛物线运动，并且在飞行中可以相互碰撞改变方向。
```c#
    public class PhysisActionMove : SSAction
    {
        public DiskData disk;

        public override void Start()
        {
            int side = Random.Range(-1, 2);
            Vector3 dst1 = new Vector3(19, Random.Range(5f, 7f), Random.Range(-3f, 0f));
            if (side < 1)
            {
                dst1.x = -dst1.x;
            }
            disk.obj.transform.position = dst1;
            if (disk.obj.GetComponent<Rigidbody>() == null)
                disk.obj.AddComponent<Rigidbody>();
            disk.obj.GetComponent<Rigidbody>().velocity = Vector3.zero;
            if (side < 1)
            {
                side = 1;
            }
            else
            {
                side = -1;
            }
            disk.obj.GetComponent<Rigidbody>().useGravity = false;

            disk.obj.GetComponent<Rigidbody>().AddForce(new Vector3(disk.ruler.speed*70*side, 0, 0), ForceMode.Force);
        }

        public override void Update()
        {
            if (disk.obj.transform.position.y < -20)
            {
                Destroy(disk.obj.GetComponent<Rigidbody>());
                this.destory = true;
                this.callback.SSActionEvent(this);
                DiskFactory.getInstance().freeDisk(disk);
            }
        }
    }
```

> 接着改动原来的CCActionManager，使其适配接口。其 playDisk函数通过创建一个CCMove并将其绑定在disk对象上，并加入动作管理器中来实现disk对象的普通线性运动。CCMove的实现和上一个版本相同。

```c#
public class CCActionManager : IActionManager
    {

        public override void playDisk(DiskData disk)
        {
            CCMove action = ScriptableObject.CreateInstance<CCMove>();
            action.disk = disk;
            action.speed = disk.ruler.speed;
            base.RunAction(disk.obj, action, (this));
        }
    }

    public class CCMove : SSAction
    {
        public float speed; //移动速度；
        public Vector3 dst;
        public DiskData disk;
 
        public override void Start()
        {
            int side = Random.Range(-1, 2);
            Vector3 dst1 = new Vector3(20, Random.Range(-1f, 3f), Random.Range(-2f, 0f));
            if (side < 1)
            {
                dst1.x = -dst1.x;
            }
            disk.obj.transform.position = new Vector3(-dst1.x, Random.Range(-1f, 3f), Random.Range(-2f, 0f));
            dst = dst1;
        }

        public override void Update()
        {
            if (gameObject.transform.position == dst)
            {
                this.destory = true;
                this.callback.SSActionEvent(this);
                DiskFactory.getInstance().freeDisk(disk);
            }
            else
            {
                gameObject.transform.position = Vector3.MoveTowards(gameObject.transform.position, dst, speed * Time.deltaTime);
            }
        }
    }
```

最后在主控制器的Awake函数中，对主控制器初始化时调用的接口进行改变。

`actionManager = gameObject.AddComponent<PhysisActionManager>() as IActionManager;`

使其加载物理运动方式的动作管理器