---
layout: post
title: 离散仿真引擎基础
date: 2019-9-10 10:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Unity3D]
---

### 1.简答题

#### 解释游戏对象（GameObjects） 和 资源（Assets）的区别与联系。

根据Unity 2019.2官方文档，对GameObjects 和 Assets的解释如下：

![1567341850090]({{site.baseurl}}/assets/assets/1567341850090.png)

![1567341792518]({{site.baseurl}}/assets/assets/1567341792518.png)

Asset：是可以在游戏或项目中使用的任何物件的表示。 其可能来自Unity外部创建的文件，例如3D模型，音频文件，图像或Unity支持的任何其他文件类型。 可以在Unity中创建一些Asset类型，例如ProBuilder Mesh，动画控制器，音频混合器或渲染纹理等。

GameObjects：Unity中的基本对象，可以是角色，道具和风景等。 它们本身并不需要多少实现，但它们充当了组件的容器以实现真正的物体功能。

区别：Asset是项目资源，是多种对象如音频，3D模型的集合，可以被项目中的任何组成部分使用以构建复杂对象。GameObjects是游戏场景的基本单位，GameObject表示游戏场景中的一个实现某种作用的一个的实体,GameObject之间可以独立的存在,也可以存在层级关系,隶属与某一对象的子对象，成为项目的组成部分。

联系：可以利用Asset的多种资源为项目生成新的GameObjects，而GameObjects可以打包添加入Assets中供其他项目的资源使用。



#### 下载几个游戏案例，分别总结资源、对象组织的结构（指资源的目录组织结构与游戏对象树的层次结构）

游戏名：Creator Kit - FPS

![1567345205643]({{site.baseurl}}/assets/assets/1567345205643.png)

![1567345238095]({{site.baseurl}}/assets/assets/1567345238095.png)

可以看到Assets资源的目录组织结构中包含了设计，场景，模型，数据，脚本等多种资源

游戏对象树的层次结构包含了预制件，Game Objects 和 层级关系。



#### 编写一个代码，使用 debug 语句来验证 MonoBehaviour 基本行为或事件触发的条件

- 基本行为包括 Awake() Start() Update() FixedUpdate() LateUpdate()
- 常用事件包括 OnGUI() OnDisable() OnEnable()

简单创建一个Cube的GameObject对象，并为其绑定编写好的C#行为脚本。

```c#
using UnityEngine;
using System.Collections.Generic;
using System.Collections;
public class testBehaviour : MonoBehaviour {
    void Start () {
        Debug.Log("Start");
    }
    void Update () {
        Debug.Log("Update");
    }
    void Awake() {
        Debug.Log("Awake");
    }
    void FixedUpdate() {
        Debug.Log("FixedUpdate");
    }
    void LateUpdate() {
        Debug.Log("LateUpdate");
    }
    void OnGUI() {
        Debug.Log("OnGUI");
    }
    void OnDisable() {
        Debug.Log("OnDisable");
    }   
    void OnEnable() {
        Debug.Log("OnEnable");
    }
}
```

根据C#对象的生命周期进行调试测验

![img]({{site.baseurl}}/assets/assets/1284555-20171129220355167-1445128134.jpg)

在控制台中可以看到如下输出：

![1567342752865]({{site.baseurl}}/assets/assets/1567342752865.png)

awake：当一个脚本实例被载入时被调用
start：在所有update函数之前被调用一次
update：当行为启用时，其update在每一帧被调用
fixedupdate：当行为启用时，其fixedupdate在每一时间片被调用
lateUpdate：在所有Update函数调用后被调用
OnGUI：渲染和处理GUI事件时调用
OnEnable：当对象变为可用或激活状态时被调用
OnDisable：当对象变为不可用或非激活状态时被调用



#### 查找脚本手册，了解 GameObject，Transform，Component 对象
- 分别翻译官方对三个对象的描述（Description）
- 描述下图中 table 对象（实体）的属性、table 的 Transform 的属性、 table 的部件
  - 本题目要求是把可视化图形编程界面与 Unity API 对应起来，当你在 Inspector 面板上每一个内容，应该知道对应 API。
  - 例如：table 的对象是 GameObject，第一个选择框是 activeSelf 属性。
  - 用 UML 图描述 三者的关系（请使用 UMLet 14.1.1 stand-alone版本出图）

![workwork]({{site.baseurl}}/assets/assets/ch02-homework.png)

##### GameObjects：
*GameObjects are the fundamental objects in Unity that represent characters, props and scenery. They do not accomplish much in themselves but they act as containers for Components.*
Unity中的基本对象，可以是角色，道具和风景等。 它们本身并不需要多少实现，但它们充当了组件的容器以实现真正的物体功能。

##### Transform：
*The Transform component determines the Position, Rotation, and Scale of each object in the scene. Every GameObject has a Transform.*
Transform组件决定了场景中物体的位置，旋转和大小。每一个GameObject都有一个Transform.

##### Component：
*A functional part of a GameObject. A GameObject can contain any number of components. Unity has many built-in components, and you can create your own by writing scripts that inherit from MonoBehaviour..*

GameObject的功能部分。 GameObject可以包含任意数量的组件。 Unity有许多内置组件，可以通过编写从MonoBehaviour继承的脚本来创建自己的组件。

从Inspector视图可以看到，

table是GameObject对象，其属性有很多，有activeSelf，Layer，Tag，Prefab等。

Transform 的属性在Transform面板中显示，Position和Rotation都为0，Scale和X，Y，Z都为1

table的部件可以从Hierarchy视图中看到，组件有 Transform，Mesh等。

**用UML图描述三者的关系**

![1567677435038]({{site.baseurl}}/assets/assets/1567677435038.png)



#### 整理相关学习资料，编写简单代码验证以下技术的实现：

- 查找对象
  ```c#
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  public class findBehaviour : MonoBehaviour {
      void Start () {
          var objectfind = GameObject.Find("objectfind");
          if(objectfind != null){
              Debug.Log(cube);
          }
  	}
  	void Update () {
         
  	}
  }
  ```
- 添加子对象
  ```c#
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  public class addBehaviour : MonoBehaviour {
      void Start () {
          GameObject newobj = GameObject.CreatePrimitive(PrimitiveType.Cube);
          newobj.name = "objectfind";
          newobj.transform.parent = this.transform;
          var objectfind = GameObject.Find("objectfind");
          if(objectfind != null){
              Debug.Log(cube);
          }
  	}
  	void Update () {
         
  	}
  }
  ```
- 遍历对象树
  ```c#
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  public class findallBehaviour : MonoBehaviour {
      void Start () {
          foreach(Transform subobj in transform){
              Debug.Log(subobj);
          }
  	}
  	void Update () {
         
  	}
  }
  ```
- 清除所有子对象
  ```c#
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  public class destroyBehaviour : MonoBehaviour {
      void Start () {
          foreach(Transform subobj in transform){
               Destroy(subobj.gameObject);
          }
  	}
  	void Update () {
         
  	}
  }
  ```



#### 资源预设（Prefabs）与 对象克隆 (clone)

- **预设（Prefabs）有什么好处？**

  预设体的是组件的集合体 , 预制物体可以实例化成游戏对象.

  创建预设体的作用: 可以重复的创建具有相同结构的游戏对象。

  有如下优点：

  1：它可以被置入多个场景中，也可以在一个场景中多次置入。
  2：当你在一个场景中增加一个Prefabs，你就实例化了一个Prefabs。
  3：所有Prefabs实例都是Prefab的克隆，所以如果实在运行中生成对象会有(Clone)的标记。
  4：只要Prefabs原型发生改变，所有的Prefabs实例都会产生变化。

- **预设与对象克隆 (clone or copy or Instantiate of Unity Object) 关系？**

  两者都能产生新的游戏对象，但不同的是，只要Prefabs原型发生改变，所有的Prefabs实例都会产生变化。

  而对于对象克隆产生的多个游戏对象，它们都具有独立的属性和方法，一个改变不会影响其它的对象。

- **制作 table 预制，写一段代码将 table 预制资源实例化成游戏对象**

  ```c#
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  public class initializationBehaviour : MonoBehaviour {
  	void Start () {
          Object tableobj = Resources.Load("table");
          GameObject table = Instantiate(tableobj) as GameObject;
  	}
  	void Update () {
  		
  	}
  }
  ```
  

### 2、 编程实践，小游戏

- 游戏内容： 井字棋 或 贷款计算器 或 简单计算器 等等
- 技术限制： 仅允许使用 **IMGUI** 构建 UI
- 作业目的：
  - 提升 debug 能力
  - 提升阅读 API 文档能力



#### 井字棋的实现

环境：Unity 2019.2.2f1 (Windows  64-bit)

使用Unity UI组件进行简单的构建，这里只需要简单的用到Button，表示棋盘，Canvas为画布显示，Camera为摄像机。

[完整代码](https://github.com/BenTsai7/3D-Computer-Game-Programming/tree/master/HW1/Tic-Tac-Toe)

演示效果：

![f862df1a-0879-4deb-8664-691c53672433]({{site.baseurl}}/assets/assets/f862df1a-0879-4deb-8664-691c53672433.gif)

如图所示，

9个Button组成一个Board棋盘，Text类型的Result用于展示结果，ResetButton用于重新开始游戏

![1567342472253]({{site.baseurl}}/assets/assets/1567342472253.png)

9个Button作为Prefab预制件方便统一管理。

制作完视图后，主要进行的是C#脚本编写，将其绑定到Board对象上。

```c#
public int[,] board;
public bool begin;
public bool turn=true;
public int count = 0;
```

二维数组用来表示棋盘，begin表示游戏是否开始，turn用来区分玩家的回合，count表示已经下的棋子数

```c#
void Awake(){
        Button btn;
        resetBtn = GameObject.Find("ResetButton");
        btn = resetBtn.transform.GetComponent<Button>();
        btn.onClick.AddListener(ResetButtonOnClick);
        hideResetButton();
        board = new int[3,3];
        cleanBoard();
    }
```

在Awake()函数中对以上进行初始化，同时为ResetButton绑定ResetButtonClick，使得当点击ResetButton时游戏可以重新开始。

setStatus和getStatus用来设置和获得棋盘状态，Button通过名字最后的A，B，C......进行区分

```c#
void setStatus(char c){
        switch(c){
            case 'A':
                board[0,0]=turn?1:2;
                break;
            ......
        }
        turn = !turn;
    }
    int getStatus(char c){
        switch(c){
            case 'A':
                return board[0,0];
            ......
        }
        return board[0,0];
    }
```

update函数首先判断游戏是否开始，同时检查是否应该结束，若结束，则输出结果，如果不是，如果用户点击了棋盘下棋，则更改棋盘状态，同时获得对应Button的子对象Text，对其的text值进行修改，使其显示出棋子。

这里使用的是Ray Cast 3D射线碰撞技术来检测鼠标点了哪一个按钮，原理是射线是在三维世界中从一个点沿一个方向发射的一条无限长的线。在射线的轨迹上，一旦与添加了碰撞器的模型发生碰撞，将停止发射。我们可以利用射线实现子弹击中目标的检测，鼠标点击拾取物体等功能。

为了使Ray Cast生效，需要在Button上添加一层碰撞器组件。

```c#
		GameObject obj;
        if (begin && checkFinish()) {
            begin=false;
            showResetButton();
        }
        if(begin==false) return;
        if(Input.GetMouseButtonDown(0)){
            Text text;
            //使用Ray Cast 射线碰撞来检测鼠标点了哪一个按钮
            Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);
            RaycastHit2D hit = 		Physics2D.Raycast(Camera.main.ScreenToWorldPoint(Input.mousePosition), Vector2.zero);
            if (hit.collider != null)
            {                  
                    string name = hit.collider.name;
                    char c = name[name.Length-1];
                    obj = GameObject.Find(name);
                    text = obj.transform.Find("Text").GetComponent<Text>();
                    if(getStatus(c)!=0) return;
                    if(turn){
                        text.text="X";
                    }
                    else{
                        text.text="O";
                    }
                    setStatus(c);
                    ++count;
                    }
        }
```

最后的相关的代码主要就是用于操纵重新开始按钮和结果文本的显示，以及检查是否结束的代码，由于井字棋只有9格，相关代码的实现比较简单，这里不做赘述。

```c#
void showResetButton(){
        resetBtn.SetActive(true);
    }
    void showResult(bool isdraw,bool X){
        Text text = GameObject.Find("Result").GetComponent<Text>();
        if(isdraw){
            text.text ="A Draw";
        }
        else{
            if(X) text.text ="X Wins";
            else text.text = "O wins";
        }
    }
    void hiddleResult(){
        Text text =GameObject.Find("Result").GetComponent<Text>();
        text.text="";
    }
    void ResetButtonOnClick(){
        cleanBoard();
        hiddleResult();
        hideResetButton();
        begin = true;
        turn = true;
        count = 0;
    }
bool checkFinish(){
        if(board[1,1]==0) return false;
        if((board[1,1]==board[0,0]&&board[1,1]==board[2,2]) ||
           (board[1,1]==board[0,2]&&board[2,0]==board[1,1]) ||
           (board[1,1]==board[0,1]&&board[2,1]==board[1,1]) ||
           (board[1,1]==board[1,0]&&board[1,2]==board[1,1])
        ){
            showResult(false,board[1,1]==1);return true;
        }
        else if(((board[0,0]==board[0,1]&&board[0,0]==board[0,2]) ||
            (board[0,0]==board[1,0]&&board[0,0]==board[2,0])) &&board[0,0]!=0
        ){
            showResult(false,board[0,0]==1);return true;
        }
        else if(((board[2,2]==board[2,0]&&board[2,2]==board[2,1]) ||
            (board[2,2]==board[1,2]&&board[2,2]==board[0,2])) && board[2,2]!=0
        ){
            showResult(false,board[2,2]==1);return true;
        }
        if(count==9) {showResult(true,true);return true;}
        return false;
    }
    void cleanBoard(){
        for(int i=0;i<3;++i)
            for(int j=0;j<3;++j)
                board[i,j] = 0;
        Button btn;
        foreach (Transform child in transform)
            {
                btn=child.GetComponent<Button>();
                btn.transform.Find("Text").GetComponent<Text>().text = "";
            }
    }
    void hideResetButton(){
        resetBtn.SetActive(false);
    }
```

### 思考题【选作】

- 微软 XNA 引擎的 Game 对象屏蔽了游戏循环的细节，并使用一组虚方法让继承者完成它们，我们称这种设计为“模板方法模式”。

  - **为什么是“模板方法”模式而不是“策略模式”呢？**

    模板方法模式定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。

    而在策略模式中，一个类的行为或其算法可以在运行时更改。这种类型的设计模式属于行为型模式。

    而对于XNA引擎架构，其提供的API是固定的，如Update和Draw，其算法的骨架已经定义好，不可能在运行时进行更改，但可以重定义其API的实现。

  

- 将游戏对象组成树型结构，每个节点都是游戏对象（或数）。

  - **尝试解释组合模式（Composite Pattern / 一种设计模式）。**

    ![1567342569762]({{site.baseurl}}/assets/assets/1567342569762.png)

组合模式（Composite Pattern），又叫部分整体模式，是用于把一组相似的对象当作一个单一的对象。组合模式依据树形结构来组合对象，用来表示部分以及整体层次。这种类型的设计模式属于结构型模式，它创建了对象组的树形结构。

这种模式创建了一个包含自己对象组的类。该类提供了修改相同对象组的方式。


  	- **使用 BroadcastMessage() 方法，向子对象发送消息。你能写出 BroadcastMessage() 的伪代码吗?**
```c#
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  public class BroadcastBehaviour : MonoBehaviour {
  	void Start () {
        //调用子物体或者父级物体的上脚本的subobj方法，并传递Broadcast参数
          this.BroadcastMessage("subobj", "Broadcast");;
  	}
  	void Update () {
  		
  	}
  }
```


- 一个游戏对象用许多部件描述不同方面的特征。我们设计坦克（Tank）游戏对象不是继承于GameObject对象，而是 GameObject 添加一组行为部件（Component）。

  - 这是什么设计模式？

    组合模式（Composite pattern），其将对象组合成树型结构来表示部分-整体（part-whole）的层次结构。

  - 为什么不用继承设计特殊的游戏对象？
  
    继承的松紧性差，耦合性高，难以适应游戏对象的变化。
    
    而组合模式的可扩展性强，解耦了客户程序与复杂元素的内部结构。由于组件间是充分解耦的，可以轻松的更新，替换组件。