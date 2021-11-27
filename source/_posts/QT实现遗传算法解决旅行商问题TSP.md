---
title: QT实现使用遗传算法解决旅行商TSP问题
date: 2021-11-26 23:13:50
tags:
    - 学习
    - QT
---
# 放在前面
- 来自广州大学人工智能实验
- 仅记录代码实现过程思路,不对涉及知识过多讲解
# 遗传算法解决旅行商问题
## 一 对旅行商问题进行编码
旅行商问题,目标是求一个走过所有目标地点尽量最短的路径。我们可以将这个路径视为一个数组,数组中的每个元素就是经过的城市的编号,元素的排列顺序就是走过地点的顺序。
以遗传算法的视角来看,该数组(路径)中的元素相当于基因,但是这个基因并不重复,因为旅行商问题不会重复走过同一个地点。
所以我们可以将每一条路径视为一个数组,数组中的元素就是地点。种群由这些一条条不同的数组(路径)组成。

在QT中可以用以下这种方式存储路径,元素int类型可以是地点的编号
```C++
QVector<int> path;//路径
```
不过我们还是得创建一个location类,将地点信息(名称,坐标等)存储一下,方便后面操作
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

## 二 遗传算法函数实现
TSP的遗传算法总体思路:
- a)随机生成N个不同的路径数个体,组成初始种群
- b)使用锦标赛选择策略从种群中选择出新的个体,组成新的交配池
- c)从交配池中随机选择出两个个体,根据交配概率PC交配
- d)如果交配则交配后按照变异概率PM进行变异,不交配直接放入新种群
- e)重复b-d步骤

我们创建一个GA类来实现TSP的遗传算法,头文件如下
后面会仔细讲解每个函数的作用
```C++
//ga.h
#ifndef GA_H
#define GA_H

#include "location.h"
#include <QVector>
#include<QRandomGenerator>
#include<QDebug>
class GA
{
public:
    GA();
    void init_races(int size,int num);//初始化种群

    double getPathLength(QVector<int>path, int num);//获取路径总长度
    void tournament(int size,int num);//锦标赛选择,选出交配池
    void getRandomSwitchPoint (int &p1, int &p2,int size);//返回p1,p2两个交叉点,保证p2>p1 距离至少为2
    void crossover(int size,int num,double pc,double pm);//交配
    void removeDuplicates(QVector<int> &path_1, QVector<int> &path_2);//去除重复
    void mutations(QVector<int>&path,int num);//变异
    double getMinPathInPool(int num,QVector<int> &minPath);//取得当前交配池里最短路径长度
    QString outputPath(QVector<int> path);//打印出路径

    void debugPool();
    void debugPath(QVector<int> path);
private:
    QVector<location> loc;//地点位置信息
    QVector<QVector<int>>* pool=nullptr;//交配池
    QVector<int> minPath;//最小路径
};

#endif // GA_H
```
### 1.初始化位置信息
我们在GA的构造函数中编写添加位置信息的代码,这样我们新建GA对象后就已经有位置信息了,在ga.cpp文件中编写代码如下
```C++
//code in ga.cpp
GA::GA()
{//初始化,添加地点信息
    loc.append(location(1,87,7));
    loc.append(location(2,91,38));
    loc.append(location(3,83,46));
    loc.append(location(4,71,44));
    loc.append(location(5,64,60));
    loc.append(location(6,68,58));
    loc.append(location(7,83,69));
    loc.append(location(8,87,76));
    loc.append(location(9,74,78));
    loc.append(location(10,71,71));
    loc.append(location(11,58,62));
    loc.append(location(12,54,62));
    loc.append(location(13,51,67));
    loc.append(location(14,37,84));
    loc.append(location(15,41,94));
    loc.append(location(16,2,99));
    loc.append(location(17,7,64));
    loc.append(location(18,22,60));
    loc.append(location(19,25,62));
    loc.append(location(20,18,54));
    loc.append(location(21,4,50));
    loc.append(location(22,13,40));
    loc.append(location(23,18,40));
    loc.append(location(24,24,42));
    loc.append(location(25,25,38));
    loc.append(location(26,41,26));
    loc.append(location(27,45,35));
    loc.append(location(28,44,35));
    loc.append(location(29,58,35));
    loc.append(location(30,62,32));
}
```
我们把这些信息写入到QVector<int\>
这个位置信息当然是根据给的题目要求来的,参数分别是 地点编号,X坐标,Y坐标。

### 2.生成初始种群
>a)随机生成N个不同的路径数个体,组成初始种群

遗传算法第一步就是要生成初始随机种群,思路上面讲了,生成给定种群大小为**NUM**个的随机不同路径,个体用QVector<int\>表示,元素int代表地点编号。
可以使用启发性的方法生成初始种群，不过我这里就用最简单较差的方法了，直接随机生成。
我们可以随机生成**SIZE**个小于SIZE值的int值存储到到这个QVector<int\>里(SIZE值就是要走过的地点数,个体的基因数)。
举个例子:
```C++
QVector<int> path ={0,4,1,3,2}
//path是一个SIZE为5的路径
//其中0,4,...,2代表城市的编号,
//元素顺序代表路径,这里表示路径为0->4->1->3->2
```
个体路径的表示方法我们解决了,另外我们还需要还要一个变量**pool**种群来存储全部生成的个体,我们可以写一个QVector<QVector<int\>>指针,然后作为GA的私有成员,这样就能方便我们后续迭代,如下:
```C++
//in ga.h 
QVector<QVector<int>>* pool=nullptr;//交配池,可以用来存储种群
```


明白之后我们来看如何实现随机初始种群,直接放上代码:
```C++
//初始化随机种群
void GA::init_races(int size,int num)//size为经过地点数,num是种群大小
{
    Q_ASSERT(this->pool==nullptr);
    this->pool=new QVector<QVector<int>>;//实例化
    for(int i=0;i<size;i++){
        QVector<int> now;//新建一个个体
        int tmprand;//随机数
        QVector<bool> tmp(num,false);//用于判断是否重复
        for(int k=0;k<num;k++){
            do{
                tmprand=QRandomGenerator::securelySeeded().bounded(num)
                //生成一个大于等于0小于num的int值
            }
            while (tmp.at(tmprand)==true);//当该值已重复时循环
            tmp[tmprand]=true;//将该值标记为重复
            now.append(tmprand);//将该值加入到个体
        }
        this->pool->append(now);//将该个体加入到种群pool
    }
}
```
主要用到了QRandomGenerator这个类,这个是QT5.10之后加进来的,可以不需要自己另外设种子了(当然你也可以设),官方文档:[QRandomGenerator Class](https://doc.qt.io/qt-5/qrandomgenerator.html#securelySeeded)

初始化好的种群在**pool**指针里了
### 2.计算适应值(计算路径长度)
在开始选择子代之前，我们需要实现一个函数用于计算适应值判断个体对于目标的适应值高低，从而有方向地选择子代个体。
在TSP问题中计算适应值就变成了计算路径长度之和了。代码如下
```C++
double GA::getPathLength(QVector<int>path, int num)
{
    qDebug()<<"getpathlength";
    Q_ASSERT(num==path.size());
    double pathLength=0;
    for(int k=0;k<path.size()-1;k++){
        //欧氏距离
        pathLength+=sqrt(pow(qAbs((this->loc[path[k]].getX())-(this->loc[path[k+1]].getX())),2)+pow(qAbs((this->loc[path[k]].getY())-(this->loc[path[k+1]].getY())),2));
        //曼哈顿距离
        //pathLength+=qAbs((this->loc[path[k]].getX())-(this->loc[path[k+1]].getX()))+qAbs((this->loc[path[k]].getY())-(this->loc[path[k+1]].getY()));
    }
    //加上最后一个地点回到开始地点的距离
    pathLength+=sqrt(pow(qAbs((this->loc[path.last()].getX())-(this->loc[path.first()].getX())),2)+pow(qAbs((this->loc[path.last()].getY())-(this->loc[path.first()].getY())),2));
    //pathLength+=qAbs((this->loc[path.last()].getX())-(this->loc[path.first()].getX()))+qAbs((this->loc[path.last()].getY())-(this->loc[path.first()].getY()));

    qDebug()<<"nowpath:"<<pathLength;
    return pathLength;
}
```
计算距离有两种方式，依据题目要求使用欧氏距离或曼哈顿距离，这里使用的是欧氏距离。另外注意有些题目要求是要从最后一个城市回到最初城市，所以还得加上这个距离。
### 3.筛选出交配池
>b)使用锦标赛选择策略从种群中选择出新的个体,组成新的交配池
其实筛选个体的方法除了锦标赛算法还有一个轮盘赌算法选择的方法，不过这里用的是锦标赛算法，轮盘赌算法就不介绍了，有兴趣可以自行了解。

>遗传算法中的锦标赛选择策略每次从种群中取出一定数量个体（放回抽样），然后选择其中最好的一个进入子代种群。重复该操作，直到新的种群规模达到原来的种群规模。几元锦标赛就是一次性在总体中取出几个个体，然后在这些个体中取出最优的个体放入保留到下一代种群的集合中。

这里我们采用二元锦标赛，每次选出两个（放回）进行比较
直接看代码实现

-----
编写中
