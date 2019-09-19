### 空间与运动

#### 1、简答并用程序验证

- ##### 游戏对象运动的本质是什么？

  游戏对象运动的本质是物体在3D空间或2D平面的平移或旋转所造成的位置的改动，或者游戏对象本身的组成部分发生平移或选择。在Unity3D中，主要表现为Transform部件的属性值发生改变。

- ##### 请用三种方法以上方法，实现物体的抛物线运动。（如，修改Transform属性，使用向量Vector3的方法…）

  抛物线运动的本质是在垂直方向上做加速度运动，在水平方向上，做匀速运动。
  1. **通过修改Transform属性来实现抛物线运动**
  
     ```c#
     public class Parabola : MonoBehaviour {    
         private double a;
         private double speed_x; //水平方向上的速度
         private double speed_z; //垂直方向上的速度
         // Use this for initialization
         void Start()
         {
             speed_x = 0.1;
             speed_z = 0.0;
             a = 0.1; //垂直加速度
         }
         // Update is called once per frame
         void Update()
         {
             //获得x，y，z坐标
             double x = transform.position.x;
             double y = transform.position.y;
             double z = transform.position.z;
             transform.position = new Vector3((float)(x + speed_x),(float) y, (float)(z + speed_z));
             speed_z += a * Time.deltaTime; //增量时间，防止因帧率变动而导致每秒移动的水平距离不一样
         }
     }
     ```
  
  2. **使用向量Vector3**
  
     ```c#
     public class Parabola : MonoBehaviour {
         private Vector3 speed_x;
         private Vector3 speed_z;
         private Vector3 a;
         // Use this for initialization
         void Start()
         {
             speed_x = 0.1f * Vector3.right; //(1,0,0)
             speed_z = Vector3.zero; //(0,0,0)
             a = 0.1f * Vector3.forward; //(0,0,1)
         }
         // Update is called once per frame
         void Update()
         {
             transform.position += speed_x;
             transform.position += speed_z;
             speed_z += Time.deltaTime * a;
         }
     }
     ```
  
  3. **rigid body刚体模拟**
  
     ```c#
     public class Parabola : MonoBehaviour {
         private Vector3 speed_x;
         private Rigidbody rigid;
         // Use this for initialization
         void Start()
         {
             speed_x = 10.0f * Vector3.right; //(1,0,0)
             rigid = GetComponent<Rigidbody>();
             rigid.velocity = new Vector3(0, 0, 10);
         }
         // Update is called once per frame
         void Update()
         {
             transform.position += speed_x * Time.deltaTime;
         }
     }
     ```
  
- ##### 写一个程序，实现一个完整的太阳系， 其他星球围绕太阳的转速必须不一样，且不在一个法平面上。

  首先从网上下载到相关的星球的贴图
  
  然后在场景中对星球进行设置与摆放，星球的本体为Sphere,根据不同行星的大小进行调整，并附上贴图
  
  ![1567316688779]({{site.baseurl}}/assets/assets/1567316688779.png)
  
  接着编写C#脚本使得太阳系能转动，脚本绑定在Sun上, Rotate用于自转，RotateAround用于公转
  
  为了让每个行星在不同法平面上旋转，需要在RotateAround的第二个参数的z轴方向加上偏移量，转速不同只需要简单的修改第三个参数的值。
  
  ```c#
  public class rotate : MonoBehaviour
  {
      // Start is called before the first frame update
      void Start()
      {
   
      }
      // Update is called once per frame
      void Update()
      {
          //Rotate为自转，RotateAround为公转
          this.transform.Rotate(Vector3.up * Time.deltaTime * 10);
          GameObject.Find("Mercury").transform.RotateAround(this.transform.position, new Vector3(0, 1, 0.5f), 60 * Time.deltaTime);
          GameObject.Find("Mercury").transform.Rotate(Vector3.up * Time.deltaTime * 10);
          GameObject.Find("Venus").transform.RotateAround(this.transform.position, new Vector3(0, 1, 0.3f), 50 * Time.deltaTime);
          GameObject.Find("Venus").transform.Rotate(Vector3.up * Time.deltaTime * 10);
          GameObject.Find("Earth").transform.RotateAround(this.transform.position, new Vector3(0, 1, 0.7f), 40 * Time.deltaTime);
          GameObject.Find("Earth").transform.Rotate(Vector3.up * Time.deltaTime * 10);
          GameObject.Find("Mars").transform.RotateAround(this.transform.position, new Vector3(0, 1, -0.1f), 30 * Time.deltaTime);
          GameObject.Find("Mars").transform.Rotate(Vector3.up * Time.deltaTime * 10);
          GameObject.Find("Jupiter").transform.RotateAround(this.transform.position, new Vector3(0, 1, -0.5f), 20 * Time.deltaTime);
          GameObject.Find("Jupiter").transform.Rotate(Vector3.up * Time.deltaTime * 10);
          GameObject.Find("Saturn").transform.RotateAround(this.transform.position, new Vector3(0, 1, -0.7f), 10 * Time.deltaTime);
          GameObject.Find("Saturn").transform.Rotate(Vector3.up * Time.deltaTime * 10);
          GameObject.Find("Uranus").transform.RotateAround(this.transform.position, new Vector3(0, 1, 0), 5 * Time.deltaTime);
          GameObject.Find("Uranus").transform.Rotate(Vector3.up * Time.deltaTime * 10);
          GameObject.Find("Neptune").transform.RotateAround(this.transform.position, new Vector3(0, 1, 0.9f), 3 * Time.deltaTime);
          GameObject.Find("Neptune").transform.Rotate(Vector3.up * Time.deltaTime * 10);
      }
  }
  ```
  
  最后为每个星系添加Trail Render，跟踪轨迹，实现效果如下：
  
  ![1567391753374]({{site.baseurl}}/assets/assets/1567391753374.png)

### 2、编程实践

- 阅读以下游戏脚本

> Priests and Devils
>
> Priests and Devils is a puzzle game in which you will help the Priests and Devils to cross the river within the time limit. There are 3 priests and 3 devils at one side of the river. They all want to get to the other side of this river, but there is only one boat and this boat can only carry two persons each time. And there must be one person steering the boat from one side to the other side. In the flash game, you can click on them to move them and click the go button to move the boat to the other direction. If the priests are out numbered by the devils on either side of the river, they get killed and the game is over. You can try it in many > ways. Keep all priests alive! Good luck!

程序需要满足的要求：

- play the game ( http://www.flash-game.net/game/2535/priests-and-devils.html )

- ##### 列出游戏中提及的事物（Objects）
  - 3个Priests （牧师)
  
  - 3个Devils (魔鬼)
  - two sides of the river （河的两岸） 
  - boat（船）


- ##### 用表格列出玩家动作表（规则表），注意，动作越少越好

  | 动作 | 条件               | 结果 |
| ------ | ------ | ------ |
| 开船 | 船上有牧师或者魔鬼 | 船移动到对岸 |
| 上岸 | 船中有牧师或魔鬼 | 将船中的牧师或魔鬼移上岸|
| 上船 | 岸上有牧师或魔鬼 |将岸中的牧师或魔鬼移上船|
	| 检测游戏状态|每步操作后|检测游戏状态为胜利，失败|

- ##### 请将游戏中对象做成预制

  将游戏对象封装为河流，船，牧师，魔鬼等预制件，河流预制件封装了整个场景

  ![1567417961570]({{site.baseurl}}/assets/assets/1567417961570.png)

- ##### 在 GenGameObjects 中创建 长方形、正方形、球 及其色彩代表游戏中的对象。

  如图，游戏利用长方体制作了水体，河岸，船等游戏对象，为其添加一层material材料贴图以区分河岸和河流，白色球代表了牧师，而红色球则表示的是恶魔。还引入了Store中下载的树进行装饰。

  ![1567417974100]({{site.baseurl}}/assets/assets/1567417974100.png)

- ##### 使用 C# 集合类型 有效组织对象

  牧师和魔鬼游戏中需要集合的地方并不多，这个游戏中直接用数组就可以有效组织对象。

- ##### 整个游戏仅 主摄像机 和 一个 Empty 对象， **其他对象必须代码动态生成！！！** 。 整个游戏不许出现 Find 游戏对象， SendMessage 这类突破程序结构的 通讯耦合 语句。 **违背本条准则，不给分**。请使用课件架构图编程，不接受非 MVC 结构程序。

  演示效果如下：整个游戏仅主摄像机和 一个 Empty 对象。
  
  
  
  ![img]({{site.baseurl}}/assets/assets/gif.gif)
<video id="video" controls="{{site.baseurl}}/assets/assets/演示.avi" preload="none" >
      <source src="" type="video/avi">
</video>
根据课件上的架构图以及从https://github.com/pmlpml/unity3d-learning/tree/motion/Assets上下载到的关于空间与运动的代码，可实现游戏结构。

使用MVC架构来构建整个游戏，根据类图构建类的接口。

**IUserAction接口的定义**

IUserAction主要定义的是用户的行为操作接口

```c#
public interface IUserAction
    {
        void Restart();
        void MoveBoat();
        int CheckGameOver();
        void MoveModel(Character Model);
    }
```

主要有移动船，移动模型（牧师或魔鬼），检查游戏是否结束，重新开始等四个操作。

**ISceneController接口的定义**

ISceneController主要用于场景切换控制器，用于载入资源，这个游戏只有单场景。

```c#
 public interface ISceneController
    {
        void LoadResources();
    }
```

**SSDirector类的定义**

SSDirector是整个游戏的导演，用于控制场景的切换。使用单例模式实现确保游戏中只有一个SSDirector，并且继承System.Object使得该类存在于整个游戏生命周期中。

```c#
public class SSDirector : System.Object
    {
    // singlton instance
    private static SSDirector _instance;
    public ISceneController CurrentSceneController { get; set; }

    // get instance anytime anywhare!
    public static SSDirector GetInstance()
    {
        if (_instance == null)
        {
            _instance = new SSDirector();
        }
        return _instance;
    }
}
```

**UserGU类的定义**

UserGUI 用于显示控制如按钮文本等GUI控件，将其与IUserACtion绑定，其维护一个状态变量state用来表示当前游戏的状态。

```c#
public class UserGUI : MonoBehaviour
    {
        public bool event_ignore;
        public int state;
        private IUserAction action;
        private GUIStyle style1, style2;
        void Start()
        {
            action = SSDirector.GetInstance().CurrentSceneController as IUserAction;
            state = 0;
            event_ignore = false;
            style1 = new GUIStyle() { fontSize = 30 };
            style1.normal.textColor = Color.white;
            style2 = new GUIStyle("button") { fontSize = 15 };
        }

        void OnGUI()
        {

            if (state == 0) return;

            if (state == 2)
            {
                GUI.Label(new Rect(Screen.width / 2 - 90, Screen.height / 2 - 70, 100, 50), "Game Over", style1);
                if (GUI.Button(new Rect(Screen.width / 2 - 70, Screen.height / 2, 100, 50), "Restart", style2))
                {
                    action.Restart();
                    state = 0;
                }
            }
            else if (state == 1)
            {
                GUI.Label(new Rect(Screen.width / 2 - 80, Screen.height / 2 - 120, 100, 50), "You Win", style1);
                if (GUI.Button(new Rect(Screen.width / 2 - 70, Screen.height / 2, 100, 50), "Restart", style2))
                {
                    action.Restart();
                    state = 0;
                }
            }
        }
    }
```
**FirstController类的定义和实现**
FirstController是当前场景的第一个控制器，实现了 ISceneController, IUserAction接口，直接与模型数据进行交互，继承了MonoBehaviour，用于挂载在Empty Object上面以生成其他所有的游戏对象，实现了与对象相关的控制操作。

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

/// <summary>
/// Scene Controller
/// Usage: host on a gameobject in the scene   
/// responsiablities:
///   acted as a scene manager for scheduling actors.log something ...
///   interact with the director and players
/// </summary>
using PAD;
public class FirstController : MonoBehaviour, ISceneController, IUserAction
{

    public GameObject scene;
    public RiverSide leftside;
    public RiverSide rightside;
    public Boat boat;
    public Character[] characters;
    public UserGUI usergui;

    // the first scripts
    void Awake()
    {
        SSDirector director = SSDirector.GetInstance();
        director.CurrentSceneController = this;
        director.CurrentSceneController.LoadResources();
        usergui = gameObject.AddComponent<UserGUI>() as UserGUI;
    }

    // loading resources for the first scence
    public void LoadResources()
    {
        scene = Object.Instantiate(Resources.Load("River", typeof(GameObject)), new Vector3(-2.503f, -8.267982f, 2.844666f), Quaternion.identity) as GameObject;
        characters = new Character[6];
        leftside = new RiverSide(true);
        rightside = new RiverSide(false);
        Vector3 dst;
        for (int i = 0; i < 3; i++)
        {
            Character character = new Character(true, i);
            characters[i] = character;
            dst = leftside.JoinLand(character);
            character.obj.transform.position = dst;
        }
        for (int i = 3; i < 6; i++)
        {
            Character character = new Character(false, i);
            characters[i] = character;
            dst = leftside.JoinLand(character);
            character.obj.transform.position = dst;
        }
        boat = new Boat();
    }
    public void Restart()
    {
        Vector3 dst;
        leftside.Reset();
        rightside.Reset();
        for (int i = 0; i < 6; i++)
        {
            characters[i].Reset();
            dst = leftside.JoinLand(characters[i]);
            characters[i].obj.transform.position = dst;
        }
        boat.Reset();
    }
    public void MoveBoat()
    {
        if (usergui.state != 0) return;
        if (boat.count != 0)
        {
            boat.MoveBoat();
            usergui.state = CheckGameOver();
        }
    }
    public int CheckGameOver()
    {
        if (rightside.DevilCount + rightside.PriestCount == 6) return 1;
        if (boat.left)
        {
            if (leftside.DevilCount + boat.getDevilCount() > leftside.PriestCount + boat.getPriestCount() && leftside.PriestCount + boat.getPriestCount() != 0) return 2;
            if (rightside.DevilCount > rightside.PriestCount && rightside.PriestCount != 0) return 2;
        }
        else
        {
            if (rightside.DevilCount + boat.getDevilCount() > rightside.PriestCount + boat.getPriestCount() && rightside.PriestCount + boat.getPriestCount() != 0) return 2;
            if (leftside.DevilCount > leftside.PriestCount && leftside.PriestCount != 0) return 2;
        }
        return 0;
    }
    public void MoveModel(Character Model)
    {
        if (usergui.state != 0) return;
        if (boat.left != Model.left) return;
        Vector3 dst;
        if (Model.onGround)
        {
            dst = boat.JoinBoat(Model);
            if (dst != Vector3.zero)
            {
                if (Model.left) leftside.LeaveLand(Model.id);
                else rightside.LeaveLand(Model.id);
                Model.LeaveLand();
                Model.JoinBoat(boat);
                Model.move.MoveToPosition(dst);
            }
        }
        else
        {
            Model.LeaveBoat();
            boat.leaveBoat(Model.id);
            if (Model.left)
            {
                Model.JoinLand(leftside);
                dst = leftside.JoinLand(Model);
            }
            else
            {
                Model.JoinLand(rightside);
                dst = rightside.JoinLand(Model);
            }
            Model.move.MoveToPosition(dst);
            usergui.state = CheckGameOver();
        }
    }
    // Use this for initialization
    void Start()
    {
    }
    // Update is called once per frame
    void Update()
    {
    }
}
```

**Model类的定义和实现**

Model文件定义了Character，RiverSide，Boat等多个类，封装了GameObject，分别表示牧师或魔鬼，河岸，船的游戏物体对象。

同时实现了Move，Click等类，继承了MonoBehaviour，成为了动态添加的部件，用于实现物体的运动和点击。

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace PAD
{
    public class Character
    {
        public GameObject obj;
        public bool isPriest;
        public bool onGround;
        public bool left;
        public int id;
        Click click;
        public Move move;
        public Character(bool isPriest, int c_id)
        {
            if (isPriest)
            {
                obj = Object.Instantiate(Resources.Load("Priest", typeof(GameObject)), Vector3.zero, Quaternion.identity) as GameObject;
                this.isPriest = true;
            }
            else
            {
                obj = Object.Instantiate(Resources.Load("Devil", typeof(GameObject)), Vector3.zero, Quaternion.identity) as GameObject;
                this.isPriest = false;
            }
            left = true;
            onGround = true;
            id = c_id;
            click = obj.AddComponent(typeof(Click)) as Click;
            click.SetCharacter(this);
            move = obj.AddComponent(typeof(Move)) as Move;
        }
        public void JoinLand(RiverSide r)
        {
            onGround = true;
            left = r.isLeft;
        }
        public void LeaveLand()
        {
            onGround = false;
        }
        public void JoinBoat(Boat b)
        {
            obj.transform.parent = b.obj.transform;
            onGround = false;
        }
        public void LeaveBoat()
        {
            obj.transform.parent = null;
            onGround = true;
        }
        public void ChangeSide()
        {
            left = !left;
        }
        public void Reset()
        {
            obj.transform.parent = null;
            onGround = true;
            left = true;
        }
    }

    public class RiverSide
    {
        public int PriestCount, DevilCount;
        public Vector3[] positions;
        public bool[] slots;
        public bool isLeft;
        Character[] characters;
        public RiverSide(bool isLeft)
        {
            //不需要额外空间以储存Gameobj，Gameobj已经在Controller中实例化
            float offset = 5.6f;
            if (isLeft)
            {
                positions = new Vector3[] {new Vector3(-1.4f,1,5.37f), new Vector3(-2.08f,1,4.92f), new Vector3(-2.07f,1,4.22f),
            new Vector3(-1.44f,1,2.72f), new Vector3(-2.1f,1,3.66f), new Vector3(-2.08f,1,3.05f)};
                this.isLeft = true;
            }
            else
            {
                positions = new Vector3[] {new Vector3(-1.4f+offset,1,5.37f), new Vector3(-2.08f+offset+0.5f,1,4.92f), new Vector3(-2.07f+offset+0.5f,1,4.22f),
            new Vector3(-1.44f+offset,1,2.72f), new Vector3(-2.1f+offset+0.5f,1,3.66f), new Vector3(-2.08f+offset+0.5f,1,3.05f)};
                this.isLeft = false;
            }
            PriestCount = 0;
            DevilCount = 0;
            slots = new bool[6];
            characters = new Character[6];
            for (int i = 0; i < 6; ++i) { slots[i] = false; characters[i] = null; }
        }
        public void LeaveLand(int id)      //离开陆地
        {
            for (int i = 0; i < 6; ++i)
            {
                if (slots[i] == true)
                {
                    if (characters[i] != null && (characters[i].id == id))
                    {
                        if (characters[i].isPriest) PriestCount--;
                        else DevilCount--;
                        characters[i] = null;
                        slots[i] = false;
                    }
                }
            }
        }
        public Vector3 JoinLand(Character c)
        {
            for (int i = 0; i < 6; ++i)
            {
                if (slots[i] == false)
                {
                    characters[i] = c;
                    if (c.isPriest) PriestCount++;
                    else DevilCount++;
                    slots[i] = true;
                    return positions[i];
                }
            }
            return Vector3.zero;
        }
        public void Reset()
        {
            PriestCount = 0;
            DevilCount = 0;
            for (int i = 0; i < 6; ++i) { slots[i] = false; characters[i] = null; }
        }

    }
    public class Boat
    {
        public GameObject obj;
        public Vector3[] positions_left, positions_right;
        public float offset = 3.0f;
        public Character[] characters;
        public bool left;
        Click click;
        public Move move;
        public int count;
        public Boat()
        {
            obj = Object.Instantiate(Resources.Load("Boat", typeof(GameObject)), new Vector3(-0.21f, 0.57f, 3.65f), Quaternion.identity) as GameObject;
            characters = new Character[2];
            positions_left = new Vector3[] { new Vector3(-0.18f, 1f, 3.99f), new Vector3(-0.18f, 1f, 3.37f) };
            positions_right = new Vector3[] { new Vector3(-0.18f + offset, 1f, 3.99f), new Vector3(-0.18f + offset, 1f, 3.37f) };
            left = true;
            click = obj.AddComponent(typeof(Click)) as Click;
            click.SetBoat(this);
            count = 0;
            move = obj.AddComponent(typeof(Move)) as Move;
        }
        public int getDevilCount()
        {
            int count = 0;
            for (int i = 0; i < characters.Length; i++)
            {
                if (characters[i] != null && (!characters[i].isPriest))
                {
                    ++count;
                }
            }
            return count;
        }
        public int getPriestCount()
        {
            int count = 0;
            for (int i = 0; i < characters.Length; i++)
            {
                if (characters[i] != null && characters[i].isPriest)
                {
                    ++count;
                }
            }
            return count;
        }
        public Vector3 JoinBoat(Character c)
        {
            for (int i = 0; i < characters.Length; i++)
            {
                if (characters[i] == null)
                {
                    characters[i] = c;
                    ++count;
                    if (left) return positions_left[i];
                    else return positions_right[i];
                }
            }
            return Vector3.zero;

        }
        public void leaveBoat(int id)
        {
            for (int i = 0; i < characters.Length; i++)
            {
                if ((characters[i] != null) && (characters[i].id == id))
                {
                    --count;
                    characters[i] = null;
                    return;
                }
            }
        }
        public void MoveBoat()
        {
            if (!left)
            {
                move.MoveToPosition(new Vector3(-0.21f, 0.57f, 3.65f));
            }
            else
            {
                move.MoveToPosition(new Vector3(2.79f, 0.57f, 3.65f));
            }
            for (int i = 0; i < characters.Length; i++)
            {
                if ((characters[i] != null))
                {
                    characters[i].ChangeSide();
                }
            }
            left = !left;
        }
        public void Reset()
        {
            left = true;
            for (int i = 0; i < characters.Length; i++)
            {
                characters[i] = null;
            }
            obj.transform.position = new Vector3(-0.21f, 0.57f, 3.65f);
            count = 0;
        }
    }

    public class Move : MonoBehaviour
    {
        Vector3 dst;
        float speed = 15; //移动速度
        bool move = false;

        void Update()
        {
            if (move)
            {
                transform.position = Vector3.MoveTowards(transform.position, dst, speed * Time.deltaTime);
                if (transform.position == dst) move = false;
            }
        }
        public void MoveToPosition(Vector3 dst)
        {
            this.dst = dst;
            move = true;
        }
    }

    public class Click : MonoBehaviour
    {
        IUserAction action;
        Boat boat;
        Character character;
        public void SetCharacter(Character character)
        {
            this.character = character;
            this.boat = null;
        }
        public void SetBoat(Boat boat)
        {
            this.boat = boat;
            this.character = null;
        }
        void Start()
        {
            action = SSDirector.GetInstance().CurrentSceneController as IUserAction;
        }
        void OnMuseDown()
        {
            if (boat == null && character == null) return;
            if (boat != null)
            {
                action.MoveBoat();
            }
            else if (character != null)
                action.MoveModel(character);

        }
    }

}
```

