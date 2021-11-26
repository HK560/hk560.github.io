---
title: QT实现使用遗传算法解决旅行商TSP问题
date: 2021-11-26 23:13:50
tags:
    - 学习
    - QT
---
# 放在前面
- 来自广州大学人工智能实验
- 仅记录代码实现过程思路，不对涉及知识过多讲解
# 遗传算法解决旅行商问题
## 1.对旅行商问题进行编码
旅行商问题，目标是求一个走过所有目标地点尽量最短的路径。我们可以将这个路径视为一个数组，数组中的每个元素就是经过的城市的编号，元素的排列顺序就是走过地点的顺序。
以遗传算法的视角来看，该数组（路径）中的元素相当于基因，但是这个基因并不重复，因为旅行商问题不会重复走过同一个地点。
所以我们可以将每一条路径视为一个数组，数组中的元素就是地点。种群由这些一条条不同的数组（路径）组成。

在QT中可以用以下这种方式存储路径，元素int类型可以是地点的编号
```C++
QVector<int> path;//路径
```
不过我们还是得创建一个location类，将地点信息（名称，坐标等）存储一下，方便后面操作
```C++
//location.h
#ifndef LOCATION_H
#define LOCATION_H

#include <QObject>
#include<QtDebug>
class location
{
private:
    int X;
    int Y;
    int code;//地点代号
public:
    location();
    location(int code,int X,int Y);
    void qdebugLocation();
    int getX();
    int getY();
    int getCode();
};

#endif // LOCATION_H
```
```C++
//location.cpp 部分代码
location::location(int code,int X,int Y){
    this->X=X;
    this->Y=Y;
    this->code=code;
}

int location::getX()
    return this->X;

int location::getY()
    return this->Y;

int location::getCode()
    return this->code;
```
然后可以用QVector存储所有地点信息
```C++
QVector<location> loc;//地点位置信息
```
## 2.遗传算法实现
TSP的遗传算法总体思路：
- a)随机生成N个不同的路径数组个体,组成初始种群,
- b)使用锦标赛选择策略从种群中选择出新的个体,组成新的交配池
- c)从交配池中随机选择出两个个体,根据交配概率PC交配
- d)如果交配则交配后按照变异概率PM进行变异,不交配直接放入新种群
- e)重复b-d步骤