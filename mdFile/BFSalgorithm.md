title: BFS algorithm
date: 2016-04-24 10:55:22
tags:
- 算法
- python
categories: 技术相关

---

一些最近遇到的广度搜索算法题

<!-- more -->

# 几道广度搜索算法
最近在看算法然后就看见了几道BFS的算法题，之前对于算法学的并不好，基本全忘了，正好用着复习一下

## 方块问题

> 在一个4行4列的平面立面，有7个红色块8个蓝色块和一个白色块，你只能移动白色的快，当移动后，白色块会和被移动的块交换位置。
使用上下左右来记录白色块的交换顺序，问：最少需要多少步能让图形从S变为T
![算法图片](/img/algorithm.jpg)

分析：一开始看见这个题目是没有什么思路的，看了提示用BFS,然后采取把BFS相关算法介绍看了一下，才大概有个思路，通过广度搜索暴力求解答案。
因为图比较小，所以花的时间并不是太多。简单的说就是模拟所有可能路径找到最短的那条。
```
def exchange(num, p1, p2):
    # 对num进行位交换操作(位从右数)
    small=p1 if p1<p2 else p2
    big=p2 if p2>=p1 else p1
    return num[:small]+num[big]+num[small+1:big]+num[small]+num[big+1:]
  


def go(node):
    # path: 到达目前状态的路径
    # cursor: 可移动块所处的位置
    # status: 其他块(同色的8个)的位置信息
    (path, cursor, status) = node
    if cursor == 0 and status == '2101101001011010':
        print(path)
    if status in visited:
        return
    visited[status] = True
    if cursor + 4 < 16: # go Up
        patterns.append((path + "D", cursor + 4, exchange(status, cursor, cursor + 4)))
    if cursor - 4 > -1: # go Down
        patterns.append((path + "U", cursor - 4, exchange(status, cursor, cursor - 4)))
    if cursor % 4 < 3: # go Left
        patterns.append((path + "R", cursor + 1, exchange(status, cursor, cursor + 1)))
    if cursor % 4 > 0: # go Right
        patterns.append((path + "L", cursor - 1, exchange(status, cursor, cursor - 1)))


patterns = [("", 0, '2011001100110011')]
visited = {} # 保存到达过的状态
count = 0
while (count < len(patterns)):
    if go(patterns[count]):
        break
    count += 1

```
当然如果用位运算应该效率更高，不过因为不擅长位运算就不用位运算了。

# 二叉树宽度

> 定义二叉树的宽度为二叉树中包含节点最多的层中的节点数。现有一颗二叉树，其深度不大于 N 
基本结构为 
typedef struct tree 
{ 
struct tree * left; 
struct tree * right; 
} * Btree 

求二叉树宽度， ROOT 为此二叉树根节点指针 
