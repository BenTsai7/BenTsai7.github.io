---
layout: post
title: 模型与动画
date: 2019-10-20 10:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Unity3D]
---

## 模型与动画
### 1、改进飞碟（Hit UFO）游戏：

智能巡逻兵

- 提交要求：
- 游戏设计要求：
  - 创建一个地图和若干巡逻兵(使用动画)；
  - 每个巡逻兵走一个3~5个边的凸多边型，位置数据是相对地址。即每次确定下一个目标位置，用自己当前位置为原点计算；
  - 巡逻兵碰撞到障碍物，则会自动选下一个点为目标；
  - 巡逻兵在设定范围内感知到玩家，会自动追击玩家；
  - 失去玩家目标后，继续巡逻；
  - 计分：玩家每次甩掉一个巡逻兵计一分，与巡逻兵碰撞游戏结束；
- 程序设计要求：
  - 必须使用订阅与发布模式传消息
    - subject：OnLostGoal
    - Publisher: ?
    - Subscriber: ?
  - 工厂模式生产巡逻兵

**演示视频如下**

<video id="video" width="500" height="300" controls="controls">
        <source src="https://bentsai7.github.io/assets/assets/演示hw6.mp4" type="video/mp4">
  </video>

**首先是Model部分巡逻兵的实现和角色的实现**

```c#
 public class Role
    {
        public GameObject obj;
		public int area;
        public Role()
        {
			area = 2;
            obj = Object.Instantiate(Resources.Load("Role", typeof(GameObject)), new Vector3(0f, 0f, 45f), Quaternion.identity) as GameObject;
        }
        public void reset()
        {
            obj.transform.position = new Vector3(0f, 0f, 45f);
        }
    }
    //巡逻兵的Model类
    public class Patrolman
    {
        public GameObject obj;
        public int area;
        public int state;
        public Patrolman(int area)
        {
            obj = Object.Instantiate(Resources.Load("Patrolman", typeof(GameObject)), new Vector3(100f, 100f, 100f), Quaternion.identity) as GameObject;
            setArea(area);
		}
        public void setArea(int area)
        {
            state = 0;
            this.area = area;
            Vector3 new_position = Vector3.zero;
            switch (area)
            {
                case 1:
                    new_position = new Vector3(31f, 0, 22f);
                    break;
                case 2:
                    new_position = new Vector3(-3f, 0, 22f);
                    break;
                case 3:
                    new_position = new Vector3(-36f, 0, 22f);
                    break;
                case 4:
                    new_position = new Vector3(31f, 0, 75f);
                    break;
                case 5:
                    new_position = new Vector3(-3f, 0, 75f);
                    break;
                case 6:
                    new_position = new Vector3(-36f, 0, 75f);
                    break;
            }
            obj.transform.position = new_position;
        }
        public void reset()
        {
            setArea(this.area);
        }
        public void patrol()
        {
            CCMove ccMove;
            ccMove = CCMove.GetSSAction(this);
            var actionManager = ((FirstController)SSDirector.GetInstance().CurrentSceneController).actionManager;
            actionManager.RunAction(this.obj, ccMove, (CCActionManager)actionManager);
        }
    }
```

**巡逻兵工厂用于生成巡逻兵**

```c#
public class PatrolmanFactory
    {
        private static PatrolmanFactory _instance;
        List<Patrolman> used = new List<Patrolman>();
        List<Patrolman> free = new List<Patrolman>();
        public Patrolman getPatrolman(int area)
        {
            Patrolman newPatrolman;
            if (free.Count > 0)
            {
                newPatrolman = free[0];
                free.RemoveAt(0);
                newPatrolman.setArea(area);
            }
            else
            {
                newPatrolman = new Patrolman(area);
            }
            used.Add(newPatrolman);
            newPatrolman.obj.SetActive(true);
            return newPatrolman;
        }
        public void freePatrolman(Patrolman patrolman)
        {
            free.Add(patrolman);
            if (!used.Remove(patrolman))
            {
                return;
            }
            patrolman.obj.SetActive(false);
        }
        // get instance anytime anywhare!
        public static PatrolmanFactory getInstance()
        {
            if (_instance == null)
            {
                _instance = new PatrolmanFactory();
            }
            return _instance;
        }
        public void reset()
        {
            foreach (Patrolman temp in used)
            {
                temp.obj.SetActive(false);
                free.Add(temp);
            }
            used.Clear();
        }
    }
```

**接着是巡逻兵的走动函数，主要是通过三个状态0，1，2分别表示巡逻状态，追踪状态和返回状态**

```C#
public class CCMove : SSAction
    {
        public float speed; //移动速度；
        public Vector3 initialpos;
		public int state; //0表示巡逻，1表示追逐玩家，2表示返回巡逻初始点
		public enum Direction { EAST, NORTH, WEST, SOUTH };
		public Direction direction;//巡逻方向'
		public float movelen = 20f;
		public Vector3 dst;
		public Patrolman p;
	public static CCMove GetSSAction(Patrolman p)
    {
        CCMove action = ScriptableObject.CreateInstance<CCMove>();
        action.p = p;
		action.state = 2;
		action.direction = Direction.EAST;
		action.speed = 10;
		return action;
    }

    public override void Start()
    {
		switch (p.area)
		{
			case 1:
				initialpos = new Vector3(42f, 0f, 17f);
				break;
			case 2:
				initialpos = new Vector3(13f, 0f, 17f);
				break;
			case 3:
				initialpos = new Vector3(-25f, 0f, 17f);
				break;
			case 4:
				initialpos = new Vector3(42f, 0f, 67f);
				break;
			case 5:
				initialpos = new Vector3(13f, 0f, 67f);
				break;
			case 6:
				initialpos = new Vector3(-25f, 0f, 67f);
				break;

		}
		dst = p.obj.transform.position + new Vector3(-movelen, 0f, 0f);

	}

    public override void Update()
    {
		p.obj.GetComponent<Animator>().SetBool("Walk Forward", true);
		if (((FirstController)SSDirector.GetInstance().CurrentSceneController).role.area == p.area) {
			state = 1;
		}
		if (state == 0)
		{
			if (p.obj.transform.position != dst)
			{
				p.obj.transform.position = Vector3.MoveTowards(p.obj.transform.position, dst, speed * Time.deltaTime);
			}
			else
			{
				switch (direction)
				{
					case Direction.EAST:
						direction = Direction.SOUTH;
						dst = p.obj.transform.position + new Vector3(0f, 0f, movelen);
						break;
					case Direction.SOUTH:
						direction = Direction.WEST;
						dst = p.obj.transform.position + new Vector3(movelen,0f,0f);
						break;
						
					case Direction.WEST:
						direction = Direction.NORTH;
						dst = p.obj.transform.position + new Vector3(0f,0f, -movelen);
						break;
					case Direction.NORTH:
						direction = Direction.EAST;
						dst = p.obj.transform.position + new Vector3(-movelen,0f,0f);
						break;
				}
			}
		}
		else if (state == 1)
		{
			if (((FirstController)SSDirector.GetInstance().CurrentSceneController).role.area != p.area) {
				EventManager.Escape();
				state = 2;
			}
			else {	
				Vector3 pos = ((FirstController)SSDirector.GetInstance().CurrentSceneController).role.obj.transform.position;
				p.obj.transform.position = Vector3.MoveTowards(p.obj.transform.position, pos, speed * Time.deltaTime);
				if ((p.obj.transform.position - pos).magnitude < 5)
				{
					EventManager.Gameover();
				}
			}

		}
		else if (state == 2)
		{
			p.obj.transform.position = Vector3.MoveTowards(p.obj.transform.position, initialpos, speed * Time.deltaTime);
			if (p.obj.transform.position == initialpos)
			{
				state = 0;
				direction = Direction.EAST;
				dst = p.obj.transform.position + new Vector3(-movelen, 0f, 0f);
			}
		}
 
    }
}
```

**单例的EventManager通过委托来实现订阅与发布模式传消息**

```c#
public class EventManager : MonoBehaviour
	{
		private static EventManager _instance;
		public delegate void ScoreEvent();
		public static event ScoreEvent ScoreChange;
		public delegate void GameoverEvent();
		public static event GameoverEvent GameoverChange;
		public static void Escape()
		{
			if (ScoreChange != null)
			{
				ScoreChange();
			}
		}
		public static void Gameover()
		{
			if (GameoverChange != null)
			{
				GameoverChange();
			}
		}
	}
```

**通过RayCast防止用户移动时碰到墙，同时每个帧获得用户的键盘输入使得角色可以上下移动**

```c#
public void moveRole(DIRECTION direction)
    {
        //上下左右移动
        RaycastHit hit;
        if (direction == DIRECTION.UP)
        {
            if (Physics.Raycast(role.obj.transform.position, transform.TransformDirection(Vector3.forward), out hit, 2*Time.deltaTime * Rolespeed))
                return;
            Vector3 e_rot = transform.eulerAngles;
            e_rot.x = 0;
            e_rot.y = 180;
            e_rot.z = 0;
            role.obj.transform.eulerAngles = e_rot;
            role.obj.transform.Translate(Vector3.forward * Time.deltaTime * Rolespeed, Space.Self);
        }
        else if (direction == DIRECTION.DOWN)
        {
            if (Physics.Raycast(role.obj.transform.position, transform.TransformDirection(Vector3.back), out hit, 2*Time.deltaTime * Rolespeed))
                return;
            Vector3 eulerAngles = transform.eulerAngles;
            eulerAngles.x = 0;
            eulerAngles.y = 0;
            eulerAngles.z = 0;
            role.obj.transform.eulerAngles = eulerAngles;
            role.obj.transform.Translate(Vector3.forward * Time.deltaTime * Rolespeed, Space.Self);
        }
        else if (direction == DIRECTION.LEFT)
        {
            if (Physics.Raycast(role.obj.transform.position, transform.TransformDirection(new Vector3(-1,0,0)), out hit, 2 * Time.deltaTime * Rolespeed))
                return;
            Vector3 eulerAngles = transform.eulerAngles;
            eulerAngles.x = 0;
            eulerAngles.y = 90;
            eulerAngles.z = 0;
            role.obj.transform.eulerAngles = eulerAngles;
            role.obj.transform.Translate(Vector3.forward * Time.deltaTime * Rolespeed, Space.Self);
        }
        else if (direction == DIRECTION.RIGHT)
        {
            if (Physics.Raycast(role.obj.transform.position, transform.TransformDirection(new Vector3(1, 0, 0)), out hit, 2 * Time.deltaTime * Rolespeed))
                return;
            Vector3 eulerAngles = transform.eulerAngles;
            eulerAngles.x = 0;
            eulerAngles.y = -90;
            eulerAngles.z = 0;
            role.obj.transform.eulerAngles = eulerAngles;
            role.obj.transform.Translate(Vector3.forward * Time.deltaTime * Rolespeed, Space.Self);
        }
    }
```

