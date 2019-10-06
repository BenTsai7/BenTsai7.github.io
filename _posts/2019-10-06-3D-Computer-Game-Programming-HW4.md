---
layout: post
title: 与游戏世界交互
date: 2019-10-06 10:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Unity3D]
---

## 作业与练习

![UML]({{site.baseurl}}/assets/assets/687474703a2f2f696d67302e706.jfif)

  

![1568953285006]({{site.baseurl}}/assets/assets/1568953285006.png)

可以根据前面牧师和魔鬼的UML架构进行改写，这里主要是添加了一个单例模式的DiskFactory类用来创建各类飞碟实例化对象的Prefab，而每个Perfab用一个DiskData进行表示，同时。还是实现了RoundController和ScoreRecorder用于控制轮数的跳转以及分数的获取。

**演示视频**：

<video id="video" width="500" height="300" controls="controls">
        <source src="https://bentsai7.github.io/assets/assets/演示hw4.mp4" type="video/mp4">
  </video>

具体实现如下：

### 动作与模型部分

> Ruler类，用于定义飞碟在不同level下的大小颜色速度和分数等。

```c#
public class Ruler
    {
        public Vector3 scale;
        public float speed;
        public int level;
        public int score;
        public Color color;
        public Ruler(int level)
        {
            switch (level)
            {
                case 1:
                    scale = new Vector3(2f, 0.2f, 2f);
                    speed = 7;
                    color = Color.black;
                    score = 1;
                    break;
                case 2:
                    scale = new Vector3(1.5f, 0.15f, 1.5f);
                    speed = 8;
                    color = Color.white;
                    score = 2;
                    break;
                case 3:
                    scale = new Vector3(1f, 0.1f, 1f);
                    speed = 9;
                    color = Color.red;
                    score = 3;
                    break;
                case 4:
                    scale = new Vector3(0.9f, 0.09f, 0.9f);
                    speed = 10;
                    color = Color.yellow;
                    score = 4;
                    break;
                case 5:
                    scale = new Vector3(0.7f, 0.07f, 0.7f);
                    speed = 11;
                    color = Color.green;
                    score = 5;
                    break;
            }
            this.level = level;
        }
    }
```

>DiskData类，飞碟类，其保存了Instantiate过的预制件飞碟的实体GameObject，以及相应level的Ruler

```c#
public class DiskData
    {
        public GameObject obj;
        public Ruler ruler;
        public DiskData(Ruler ruler)
        {
            obj = Object.Instantiate(Resources.Load("Disk", typeof(GameObject)), new Vector3(100f, 100f, 100f), Quaternion.identity) as GameObject;
            reset(ruler);
        }
        public void reset(Ruler ruler)
        {
            obj.GetComponent<Renderer>().material.color = ruler.color;
            obj.transform.localScale = ruler.scale;
            this.ruler = ruler;
            obj.SetActive(true);
        }
        public void fly()
        {
            CCMove ccMove;
            ccMove = CCMove.GetSSAction(this);
            var actionManager = ((FirstController)SSDirector.GetInstance().CurrentSceneController).actionManager;
            actionManager.RunAction(this.obj, ccMove, (CCActionManager)actionManager);
        }
    }
```

> 重定义过的CCMoveAction类，用于控制飞碟的飞行状态，将飞碟的飞行请求加入到ActionManager的动作管理队列中进行执行。同时在飞行前，随机化飞行的初始位置和终点，使得各轮飞碟的飞行路线不同。当飞碟飞至目的地时，通过调用飞碟工厂类进行回收，以实现资源的重利用。
```c#
public class CCMove : SSAction
    {
        public float speed; //移动速度；
        public Vector3 dst;
        public DiskData disk;

        public static CCMove GetSSAction(DiskData disk)
        {
            CCMove action = ScriptableObject.CreateInstance<CCMove>();
            action.disk = disk;
            action.speed = disk.ruler.speed;
            int side = Random.Range(-1, 2);
            Vector3 dst1 = new Vector3(20, Random.Range(-1f, 3f), Random.Range(-2f, 0f));
            if (side < 1)
            {
                dst1.x = -dst1.x;
            }
            disk.obj.transform.position = new Vector3(-dst1.x, Random.Range(-1f, 3f), Random.Range(-2f, 0f));
            action.dst = dst1;
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
                DiskFactory.getInstance().freeDisk(disk);
            }
            else
            {
                gameObject.transform.position = Vector3.MoveTowards(gameObject.transform.position, dst, speed * Time.deltaTime);
            }
        }
    }
```

### 飞碟工厂类与显示部分

> IUserAction定义了用户通过GUI可以触发的动作，包括开始游戏，重置游戏，获得状态，检测是否击中飞碟。

```c#
public interface IUserAction
    {
        void Reset();
        int getState();
        void GameStart();
        void hitDisk(GameObject disk);
    }
```



> UserGUI，用于控制用户的点击按钮，点击飞碟的操作，将其传递给场景控制器进行处理，同时显示计分控制器的分数。如果用户点击了鼠标左键，通过从摄像机Camera发送RaycastHit射线来判断是否击中了飞碟。

```c#
 public class UserGUI : MonoBehaviour
    {
        private IUserAction action;
        private GUIStyle style1, style2;
        void Start()
        {
            action = SSDirector.GetInstance().CurrentSceneController as IUserAction;
            style1 = new GUIStyle() { fontSize = 30 };
            style1.normal.textColor = Color.black;
            style2 = new GUIStyle("button") { fontSize = 15 };
        }
    void Update()
    {
        if (action.getState() == 0)
        {
            checkHit();
        }
    }

    void checkHit()
    {
        if (Input.GetButtonDown("Fire1"))
        {
            Vector3 mp = Input.mousePosition;
            Camera ca = Camera.main;
            Ray ray = ca.ScreenPointToRay(Input.mousePosition);
            RaycastHit hit;
            if (Physics.Raycast(ray, out hit))
            {
                ((FirstController)SSDirector.GetInstance().CurrentSceneController).hitDisk(hit.transform.gameObject);
            }
        }
    }

    void OnGUI()
    {
        if (action.getState() == 0)
        {
            GUI.Label(new Rect(20, 20, 120, 25), "score: " + ((FirstController)SSDirector.GetInstance().CurrentSceneController).scoreCtrl.score,style1);
            return;
        }
        if (action.getState() == 1)
        {
            if (GUI.Button(new Rect(Screen.width / 2 -50 , Screen.height/2 -25, 100, 50), "Start", style2))
            {
                action.Reset();
                action.GameStart();
            }
        }
    }
}
```

> 飞碟工厂类，在场景控制器需要某种飞碟的时候，飞碟工厂从仓库(List <DiskData> free)的空闲队列中中获取飞碟，如果仓库中没有，则实例化一个新的飞碟，然后添加到正在使用的used飞碟列表中。当场景控制器发现飞碟被打中或者飞碟飞到飞行路线的尽头时，将执行回收飞碟。回收飞碟的操作便是将飞碟从正在使用的used队列中移出，加入free队列中，以供新飞碟的获取。`getHitDisk`函数在飞碟列表中搜素指定的飞碟，以判断被击中的飞碟是否是正在使用中的飞碟。

```c#
public class DiskFactory
    {
        private static DiskFactory _instance;
        List<DiskData> used = new List<DiskData>();
        List<DiskData> free = new List<DiskData>();
        public DiskData getDisk(Ruler ruler)
        {
            DiskData newDisk;
            if (free.Count > 0)
            {
                newDisk = free[0];
                newDisk.reset(ruler);
                free.RemoveAt(0);
            }
            else
            {
                newDisk = new DiskData(ruler);
            }
            used.Add(newDisk);
            newDisk.obj.SetActive(true);
            return newDisk;
        }
        public void freeDisk(DiskData disk)
        {
            free.Add(disk);
            if (!used.Remove(disk))
            {
                return;
            }
            disk.obj.SetActive(false);
        }
        // get instance anytime anywhare!
        public static DiskFactory getInstance()
        {
            if (_instance == null)
            {
                _instance = new DiskFactory();
            }
            return _instance;
        }
        public void reset()
        {
            foreach (DiskData temp in used)
            {
                temp.obj.SetActive(false);
                free.Add(temp);
            }
            used.Clear();
        }
        public DiskData getHitDisk(GameObject obj)
        {
            foreach (DiskData d in used)
            {
                if (d.obj == obj)
                {
                    return d;
                }
            }
            return null;
        }
    }
```

> 计分器类，用于在飞碟被击中时增加分数以及重置分数

```c#
public class ScoreController
    {
        public int score = 0;
        public void addScore(int score)
        {
            this.score += score;
        }
        public void reset()
        {
            score = 0;
        }
    }
```

### 主控制器部分

> FirstController主控制器通过游戏状态的变化来控制游戏运行。游戏状态有两个，state值0和1代表了游戏的开始和结束。控制器发射飞碟时首先从飞碟工厂类获取飞碟，然后为飞碟添加动作，将其放入动作管理器中执行飞行动作。最后若用户成功点击飞碟，则更新计分器的计分值。同时这里新增了一个爆炸效果的粒子系统预制件，如果玩家击中飞碟，则在击中处产生一个0.5秒的爆炸。

```c#
public class FirstController : MonoBehaviour, ISceneController, IUserAction
{
    public int state;
    public UserGUI usergui;
    public CCActionManager actionManager;
    public ScoreController scoreCtrl;
    public DiskFactory diskFactory;
    public int round;
    public Ruler ruler;
    public bool RoundFinished = true;
    public float time;
    public float interval;

    // the first scripts
    void Awake()
    {
        state = 1;
        interval = 7.0f;
        SSDirector director = SSDirector.GetInstance();
        director.CurrentSceneController = this;
        director.CurrentSceneController.LoadResources();
        usergui = gameObject.AddComponent<UserGUI>() as UserGUI;
        actionManager = gameObject.AddComponent<CCActionManager>() as CCActionManager;
        diskFactory = DiskFactory.getInstance();
        scoreCtrl = new ScoreController();
        actionManager.Start();
    }

    public int getState()
    {
        return state;
    }

    // loading resources for the first scence
    public void LoadResources()
    {

    }
    public void GameStart()
    {
        round = 1;
        state = 0;
        time = 10;
    }
    public void hitDisk(GameObject disk)
    {
        DiskData temp = diskFactory.getHitDisk(disk);
        if (temp != null)
        {
            GameObject obj = Object.Instantiate(Resources.Load("ParticleSystem", typeof(GameObject)),disk.transform.position, Quaternion.identity) as GameObject;
            scoreCtrl.addScore(temp.ruler.score);
            diskFactory.freeDisk(temp);
            Destroy(obj, 0.5f);
        }
    }
    public void FlySomeDisk(int round)
    {
        int num = (int) Random.Range(5f, 10f);
        ruler = new Ruler(round);
        for(int i = 0; i < num; ++i)
        {   
            FlyOneDisk();
        }
    }

    public void FlyOneDisk()
    {
        DiskData disk = diskFactory.getDisk(ruler);
        disk.fly();
    }

    public void Reset()
    {
        diskFactory.reset();
        scoreCtrl.score = 0;
    }
   
    void checkIfNeedFly()
    {
        if (time >= interval)
        {
            if (round <= 5)
            {
                FlySomeDisk(round);
            }
            ++round;
            time = 0;
        }
    }

    // Use this for initialization
    void Start()
    {

    }
    // Update is called once per frame
    void Update()
    {
        if (state != 0) return;
        if (round >5)
        {
            if (time > 5.0f)
            {
                state = 1;
                return;
            }
        }
        time += Time.deltaTime;
        checkIfNeedFly();
    }
}
```

