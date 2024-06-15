---
title: Dijkstra算法
date: 2024-04-11 16:41:26
tags:
 - 图
 - Dijkstra
categories: 算法
mathjax: true
---

## Dijkstra算法介绍

Dijkstra算法是求解 **非负权图** 上单源最短路径的算法

 参考 [leetcode-2642.设计可以求最短路径的图类]([2642. 设计可以求最短路径的图类 - 力扣（LeetCode）](https://leetcode.cn/problems/design-graph-with-shortest-path-calculator/submissions/522530035/))

将结点分成两个集合：已确定最短路长度的点集（记为`S`集合）的和未确定最短路长度的点集（记为 `T` 集合）。一开始所有的点都属于`T` 集合。邻接矩阵`arcs[i][j]`表示有向边`i -> j` 的权值，创建一个辅助数组`dist[n]`，其中`n`是图中节点个数，`dist[i]`表示从起点开始到节点 `i` 的最短路径长度，这里设起点为`i`

初始化`dist[i]=0`，其他点的 `dist[j]=arcs[i][i]` 。

然后重复这些操作：

1. 从 `T` 集合中，选取一个最短路长度最小的结点，移到 `S` 集合中。
2. 修改从起点`i`出发到集合`T`上任意一个顶点`k`可达的最短路径长度：若`dist[j] + arcs[j][k] < dist[k]`，则更新`dist[k]=dist[j] + arcs[j][k]`

直到 T集合为空，算法结束。

操作一的时间复杂度为$$O(n^2)$$，操作二时间复杂度为$$O(m)$$，总复杂度为$$O(n^2+m)$$，考虑到`m<=n-1`，总时间复杂度为$$O(n^2)$$

## 代码

代码如下：

```java
public int shortestPath(int node1, int node2) {
        //graph的数据类型为List<int[]>[] graph = new List[n]，graph[i]表示节点以i为有向边起点的所有边和权重，graph[i]中int[]数组的第一个元素表示有向边的终点，第二个元素表示有向边的权重
        int[] dist = new int[n];
        boolean[] visited = new boolean[n];//存放S集合中的节点
        Arrays.fill(dist, Integer.MAX_VALUE/2);//这里除以二是防止溢出
        dist[node1] = 0;
        visited[node1] = true;
        for(int[] nextArr : graph[node1]){//初始化
            dist[nextArr[0]] = nextArr[1];
        }
        for(int i=0;i<n-1;i++){
            int minNode=0, minValue = Integer.MAX_VALUE;
            for(int j=0;j<n;j++){
                if(!visited[j]&&dist[j]<minValue){//算法步骤一：选取最短路径节点
                    minNode = j;
                    minValue = dist[j];
                }
            }
            visited[minNode] = true;//算法步骤一：加入S集合
            for(int[] nextArr : graph[minNode]){
                int next = nextArr[0], ncost = nextArr[1];
                if(dist[next]>dist[minNode]+ncost){//算法步骤二：更新最短路径
                    dist[next] = dist[minNode]+ncost;
                }
                
            }
        }
        return dist[node2]==Integer.MAX_VALUE/2?-1:dist[node2];

    }
```



也可以用优先队列来优化，总时间复杂度为$$O(mlogm)$$

```java
public int shortestPath(int node1, int node2) {
    //graph的数据类型为List<int[]>[] graph = new List[n]，graph[i]表示节点以i为有向边起点的所有边和权重，graph[i]中int[]数组的第一个元素表示有向边的终点，第二个元素表示有向边的权重
        Queue<int[]> queue = new PriorityQueue<>((a1,a2)->(a1[1]-a2[1]));
        int[] dist = new int[graph.length];
        Arrays.fill(dist, Integer.MAX_VALUE);
        dist[node1]=0;
        queue.offer(new int[]{node1, 0});
        while(!queue.isEmpty()){
            int[] array = queue.poll();
            int cost = array[1];
            int cur = array[0];

            if(cur==node2)return cost;
            for(int[] edge : graph[cur]){
                int ncost = edge[1];
                int nnode = edge[0];
                if(dist[nnode] > cost + ncost){
                    dist[nnode] = cost + ncost;
                    queue.offer(new int[]{nnode, ncost+cost});
                }
            }
        }
        return -1;
    }
```

