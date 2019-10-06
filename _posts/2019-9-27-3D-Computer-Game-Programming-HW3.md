---
layout: post
title: 游戏对象与图形基础
date: 2019-9-23 10:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Unity3D]
---

## 游戏对象与图形基础

### 1、操作与总结

- **参考 Fantasy Skybox FREE 构建自己的游戏场景**

  首先在Asset Store上将Fantasy Skybox FREE资源下载到本地，并导入项目中

  将导入的资源中Material文件夹中的Sunny_01A贴图拖动到场景中应用于Sky Box上

  ![1567838020530]({{site.baseurl}}/assets/assets/1567838020530.png)

  接着创建一个Terrain对象，将资源Material中的Groud地表贴图应用于其Setting的Material属性其上，可以构建出一片地表。使用Terrain对象的工具可以改变地形，同时可以使用笔刷种树。可以构建如以下的简单的游戏场景。
  
  ![1567940570263]({{site.baseurl}}/assets/assets/1567940570263.png)
  
- **写一个简单的总结，总结游戏对象的使用**
  
  **游戏对象的定义：**
  
  游戏对象出现在游戏的场景中，是资源整合的具体表现对象。一般有玩家、敌人、环境、摄像机等非实体虚拟父类，但子类常为游戏内的实体；
  资源一般包含对象、材质、场景、声音、预设、贴图、脚本、动作等子文件夹，通常可进一步划分。
  
  >游戏对象分三种：
  (1) 将物体模型等资源由Project工程面板拖拽到Hierarchy层次面板中，即Prefab
  (2) 由GameObject菜单创建Unity自带的游戏对象，如Cube、Camera、Light等 
  (3) 利用脚本动态创建或删除游戏对象
  
  **游戏对象的操作**
  
  1.创建游戏对象
可以通过在编辑器里直接添加
	或使用`GameObject.CreatePrimitive()`在脚本中创建

	2.预设件的使用
游戏对象可以封装为预制件Prefab，可以被储存在Assets文件夹内，以方便在其他scene中重复利用。
	通过将.预设资源由Project工程面板拖拽到Hierarchy层次面板中或使用`GameObject.Instantiate()`来产生预制件。

	3.寻找游戏对象

	- 通过对象名称（Find方法）
	- 通过标签获取单个游戏对象（FindWithTag方法）
	- 通过标签获取多个游戏对象（FindGameObjectsWithTags方法）
	- 通过类型获取单个游戏对象（FindObjectOfType方法）
	- 通过类型获取多个游戏对象（FindObjectsOfType方法）
	
	4.删除游戏对象
	
	`Destroy()`
	
	5.添加和修改组件
  
  `GameObject.AddComponent(className:string)`
  
  `GameObject.GetComponent(type:Type)
  
  6.消息广播
  
  `GameObject.SendMessage`: 发送消息
  
  `GameObject.BroadcastMessage`:广播消息
  
  `GameObject.SendMessageUpwards`:向上发送消息


### 2、编程实践

- **牧师与魔鬼 动作分离版**

  演示视频
  
  <video id="video" width="500" height="300" controls="controls">
        <source src="https://bentsai7.github.io/assets/assets/演示.mp4" type="video/mp4">
  </video>
  
  ![UML]({{site.baseurl}}/assets/assets/687474703a2f2f696d67302e706.jfif)

动作分离指的是将不同动作的实现分离开来并继承SSAction，由SSActionManager进行统一管理。

参考上次编写的代码，之前的Move动作类是绑定于对象的，这里可以将Move的实现与使用分离并继承SSAction，对象调用动作时通过动作管理器进行调用管理，这样减少了Move动作的冗余存储。

定义一个Action.cs，namespace为PAD，包含新增的所有动作管理相关的类。

首先按UML图实现SSAction，其继承了ScriptableObject，ScriptableObject是一个允许你存储大量独立于脚本实例的共享数据的类。
```c#
 public class SSAction : ScriptableObject
    {
        public bool destory = false;
        public bool enabled = true;
        public GameObject gameObject { get; set; }
        public ISSActionCallBack callback { get; set; }
        protected SSAction() { }
        public virtual void Start() { throw new System.NotImplementedException(); }
        public virtual void Update() { throw new System.NotImplementedException(); }
    }
```

接着定义回调接口的类型以及方式，这个游戏中不需要回调函数起真正作用。

```c#
 public enum SSActionEventType : int { Started, Completed };
    public interface ISSActionCallBack
    {
        void SSActionEvent(SSAction sourse, SSActionEventType events = SSActionEventType.Completed);
    }
```

定义动作管理的基类，其实际上就是在维护一个动作队列，如果动作完成，则删除SSAction对象，如果没有完成，则调用SSAction的update方法

```c#
public class SSActionManager : MonoBehaviour
    {
        private Dictionary<int, SSAction> actions = new Dictionary<int, SSAction>();
        private List<SSAction> AddList = new List<SSAction>();
        private List<int> DeleteList = new List<int>();

        protected void Update()
        {
            foreach (SSAction ac in AddList) actions[ac.GetInstanceID()] = ac;
            AddList.Clear();
            foreach (KeyValuePair<int, SSAction> kv in actions)
            {
                SSAction ac = kv.Value;
                if (ac.destory)
                {
                    DeleteList.Add(ac.GetInstanceID());
                }
                else if (ac.enabled)
                {
                    ac.Update();
                }
            }

            foreach (int key in DeleteList)
            {
                SSAction ac = actions[key];
                actions.Remove(key);
                Object.Destroy(ac);
            }
            DeleteList.Clear();
        }

        public void Start() { }

        public void RunAction(GameObject gameObject, SSAction action, ISSActionCallBack manager)
        {
            action.gameObject = gameObject;
            action.callback = manager;
            AddList.Add(action);
            action.Start();
        }
    }
```

具体实现CCActionManager，其定义了物体通用的Move操作

```c#
public class CCActionManager : SSActionManager, ISSActionCallBack
    {
        public FirstController sceneController;

        public void MoveObject(GameObject obj,Vector3 dst)
        {
            CCMove MoveAction = CCMove.GetSSAction(dst);
            this.RunAction(obj, MoveAction, this);
        }
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
    }
```

重新实现Move为SSAction的扩展，这里并不需要重新实现Click，因为SSAction是对持续性动作的模拟，不需要实现Click。

```c#
  public class CCMove : SSAction
    {
        public Vector3 dst;
        public float speed = 15; //移动速度；

        public static CCMove GetSSAction(Vector3 dst)
        {
            CCMove action = ScriptableObject.CreateInstance<CCMove>();
            action.dst = dst;
            return action;
        }

        public override void Start()
        {
            gameObject.transform.position = Vector3.MoveTowards(gameObject.transform.position, dst, speed * Time.deltaTime);
        }

        public override void Update()
        {
            if (gameObject.transform.position == dst)
            {
                this.destory = true;
                this.callback.SSActionEvent(this);
            }
            else
            {
                gameObject.transform.position = Vector3.MoveTowards(gameObject.transform.position, dst, speed * Time.deltaTime);
            }
        }
    }

```

最后只需要更改前面First Controller控制器对应的move调用为管理器调用就完成动作分离了，以及为FirstContler添加ActionManager控件并初始化。

```c#
 void Awake()
    {
        SSDirector director = SSDirector.GetInstance();
        director.CurrentSceneController = this;
        director.CurrentSceneController.LoadResources();
        usergui = gameObject.AddComponent<UserGUI>() as UserGUI;
        actionManager = gameObject.AddComponent<CCActionManager>() as CCActionManager;
        actionManager.Start();
    }
```

具体代码可见于github中

