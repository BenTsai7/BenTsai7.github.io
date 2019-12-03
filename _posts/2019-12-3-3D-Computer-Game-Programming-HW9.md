---
layout: post
title: 游戏智能
date: 2019-11-23 10:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Unity3D]
---

## 游戏智能

P&D 过河游戏智能帮助实现，程序具体要求：

- 实现状态图的自动生成
- 讲解图数据在程序中的表示方法
- 利用算法实现下一步的计算

### 演示效果

<video id="video" width="500" height="300" controls="controls">
        <source src="https://bentsai7.github.io/assets/assets/演示hw9.mp4" type="video/mp4">
  </video>


### 状态图的自动生成

![1572515242429]({{site.baseurl}}/assets/assets/1572515242429.png)

我们要实现的状态图类似于上图。只不过要自动生成以上的状态图并进行搜索路径的计算。

### 图数据在程序中的表示方法

图可以表示为矩阵或邻接表，由于这里每个状态实际上是记录了P和数量，D的数量，和Boat的位置。所以用矩阵无法很好地表示，这里用邻接表来实现。

首先定义每个状态节点State，其包含的属性有P，D，B状态三元组，同时list<> neighbour保存着其邻接的节点列表。这里定义了`Connect`方法用于连接两个节点，加入邻接链表中。`isEqual`用来判定两个节点状态是否相同。

```c#
public class Node
	{
		public int P, D;
		public bool B; //True代表在左
		public List<Node> neighbour;
		public Node(int P, int D, bool B)
		{
			this.P = P;
			this.D = D;
			this.B = B;
			this.neighbour = new List<Node>();
		}
		//深拷贝
		public Node(Node n)
		{
			this.P = n.P;
			this.D = n.D;
			this.B = n.B;
			this.neighbour = new List<Node>(n.neighbour);
		}
		public void Connect(Node node)
		{
			foreach (Node n in neighbour)
			{
				if (isEqual(n, node))
				{
					return;
				}
			}
			neighbour.Add(node);
		}
		public static bool isEqual(Node first, Node second)
		{
			if (first.P == second.P && first.D == second.D
				&& first.B == second.B)
			{
				return true;
			}
			return false;
		}
	}
```

### 解法搜索

首先定义两个结构`StateMove`和`SearchNode`，`StateMove`用于表示状态的转移变化，`SearchNode`用于BFS解法路径的搜索。

```c#
public class StateMove
	{
		public int P;
		public int D;
		public StateMove(int P, int D)
		{
			this.P = P;
			this.D = D;
		}
	}
	//用于BFS搜索
	public class SearchNode
	{
		public Node startNode;
		public Node realNode;
		public SearchNode(Node startNode, Node realNode)
		{
			this.startNode = startNode;
			this.realNode = realNode;
		}
	}
```

接着定义图Graph，其用于生成状态图，并寻找路径。

`construct`函数层层枚举所有可能的状态和状态之间的转移关系，并将生成的状态图保存在List中。通过BFS宽度优先搜索，通过构建一个队列寻找到某个状态节点到最终状态节点的路径的下一步。

```c#
public class Graph
	{
		public List<Node> nodes;
		//可走状态
		public StateMove[] moves = { new StateMove(0, 1), new StateMove(1, 0), new StateMove(1, 1), new StateMove(2, 0), new StateMove(0, 2) };
		public int maxP;
		public int maxD;
		public Node endNode;
		public Graph(int maxP, int maxD)
		{
			this.maxP = maxP;
			this.maxD = maxD;
			nodes = new List<Node>();
			endNode = new Node(0, 0, false);
			construct();
		}
		private void construct()
		{
			//初始点
			nodes.Add(new Node(maxP, maxD, true));
			for (int i = 0; i < nodes.Count; ++i)
			{
				Node curNode = nodes[i];
				foreach (StateMove move in moves)
				{
					Node n = new Node(curNode.P, curNode.D, !curNode.B);
					//一开始在左岸
					if (curNode.B)
					{
						n.P -= move.P;
						n.D -= move.D;
					}
					else
					{
						n.P += move.P;
						n.D += move.D;
					}
					//验证该解是否有效
					if (isValid(n))
					{
						bool flag = false;
						foreach (Node tmp in nodes)
						{
							if (Node.isEqual(tmp, n))
							{
								flag = true;
								break;
							}
						}
						if (flag) continue;
						n.Connect(curNode);
						curNode.Connect(n);
						nodes.Add(n);
					}
				}
			}
		}
		private bool isValid(Node n)
		{
			if (n.D < 0 || n.P < 0) return false;
			if (n.D > maxD || n.P > maxP) return false;
			if (n.D > n.P && n.P != 0) return false;
			int P = maxP - n.P;
			int D = maxD - n.D;
			if (D > P && P != 0) return false;
			return true;
		}

		public StateMove getNextMove(Node curNode)
		{
			Node nextNode = BFS(curNode);
			if (nextNode == null)
			{
				return null;
			}
			else
			{
				return new StateMove(Mathf.Abs(curNode.P - nextNode.P), Mathf.Abs(curNode.D - nextNode.D));
			}
		}

		public Node getNodeFromGraph(Node node) {
			foreach (Node n in nodes) {
				if (Node.isEqual(n, node))
					return n;
			}
			return null;
		}
		private Node BFS(Node begin)
		{
			List<SearchNode> searchList = new List<SearchNode>();
			Node beginnode = getNodeFromGraph(begin);
			searchList.Add(new SearchNode(null, beginnode));
			foreach (Node n in beginnode.neighbour)
			{
				if (Node.isEqual(n, endNode))
				{
					return n;
				}
				else
				{
					searchList.Add(new SearchNode(n, n));
				}
			}
			for (int i = 1; i < searchList.Count; i++)
			{
				foreach (Node n in searchList[i].realNode.neighbour)
				{
					if ((Node.isEqual(n, endNode)))
					{
						return searchList[i].startNode;
					}
					else
					{
						bool flag = false;
						foreach (SearchNode sn in searchList)
						{
							if (Node.isEqual(n, sn.realNode))
							{
								flag = true;
								break;
							}
						}
						if (flag) continue;
						searchList.Add(new SearchNode(searchList[i].startNode, n));
					}
				}
			}
			return null;
		}
	}
```

FirstController维护游戏场景的Node状态，并在物体移动时发生更新。同时实现了一个`tips`函数，通过调用BFS获得的结果构造提示的字符串返回给UserGUI进行渲染显示。

```c#
public string tips() {
		StateMove statemove = GRAPH.getNextMove(STATE);
		string tipstring;
		string number = "";
		int moveP = statemove.P;
		int moveD = statemove.D;
		if (moveP != 0) {
			number += moveP.ToString();
			if (moveP > 1)
			{
				number += " Priests";
			}
			else {
				number += " Priest";
			}
		}
		if (moveD != 0) {
			if (moveP != 0) number += " And ";
			number += moveD.ToString();
			if (moveD > 1)
			{
				number += " Devils";
			}
			else
			{
				number += " Devil";
			}
		}

		if (boat.left)
		{
			tipstring = "Move " + number + " To Right";
		}
		else {
			tipstring = "Move " + number + " To Left";
		}
		return tipstring;
	}
```



