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

这里我们采用二元锦标赛，每次选出两个（放回）进行比较路径长度大小,将较小的的路径加入到新交配池子
直接看代码实现
```C++
void GA::tournament( int size,int num)//锦标赛选择新交配池
{
    qDebug()<<"tournament";
    Q_ASSERT(size>0);
    QVector<QVector<int>> *newpool=new QVector<QVector<int>>;//新池子
    int time=size;
    while(time--){
        int index_1=QRandomGenerator::system()->bounded(size);//随机数作为下标
        int index_2=QRandomGenerator::system()->bounded(size);
        //比较路径长度
        if(getPathLength(this->pool->at(index_1),num)<getPathLength(this->pool->at(index_2),num)){
            newpool->append(this->pool->at(index_1));//将该路径加入到新池子
        }else{
            newpool->append(this->pool->at(index_2));
        }
    }
    delete this->pool;
    this->pool=newpool;
    debugPool();
}
```
### 4.进行交配和变异
>c)从交配池中随机选择出两个个体,根据交配概率PC交配
d)如果交配则交配后按照变异概率PM进行变异,不交配直接放入新种群

这一步我们需要随机选出两个个体,根据交配概率PC判断是否进行交配,如果不交配,那我们直接将这两个个体放入新种群(交配池)即可;如果交配,则要进行交配策略，进行交配策略之后还要根据变异概率PM是否进行变异

遗传算法中如何进行交配呢? 常用的策略方法有一点交叉和两点交叉
>交叉的步骤如下：
1.从交配池中随机选出两个个体作为需要交叉的个体
2.根据个体基因长度SIZE，随机生成一个或多个范围在[0,size-1]的整数下标点作为交叉点
3.交换两个个体由交叉点指定的基因范围部分‘

一点交叉：
就是只选择一个交叉点进行交换
![](https://cdn.jsdelivr.net/gh/HK560/MyPicHub@master/res/pic/20211128193901.png)
两点交叉：
两个交叉点，将两个交叉点之间的基因部分交换
![](https://cdn.jsdelivr.net/gh/HK560/MyPicHub@master/res/pic/20211128194004.png)
要注意的是，交换后会出现部分基因重复，也就是地点重复，这是不允许的，所以我们要去掉重复地点
方法很简单，分别找到两个个体中重复的部分，然后互相交换即可。

然后再说说变异，变异在tsp问题中我们用一种简单的方法进行处理：随机交换一个个体中两个基因的位置。
也就是随机选取两个地点交换在路径中的位置
例如：1->2->3->4
可以变异为 1->4->3->2(交换了2和4的位置)

理解后我们就可以写代码了：

```C++
//获得两个符合规则的交叉点
void GA::getRandomSwitchPoint(int &p1,int &p2,int num)
{
    //Q_ASSERT(p1>=0&&p2>=0);
    int tmp1,tmp2;
    do{
        tmp1=QRandomGenerator::system()->bounded(num);
        tmp2=QRandomGenerator::system()->bounded(num);
    }while(tmp1>=tmp2&&!((tmp2-tmp1)>=2));
    p1=tmp1;
    p2=tmp2;
}
//去除重复
void GA::removeDuplicates(QVector<int> &path_1, QVector<int> &path_2)
{
    qDebug()<<"removeDuplicates";
    //找出第一个个体的重复基因
    QVector<int> duplicates_1;
    for(auto i=path_1.begin();i!=path_1.end();i++){
        if(path_1.count(*i)>1){
            if(duplicates_1.contains(*i)==false)
                duplicates_1.append(*i);
        }
    }
    //找出第二个个体的重复基因
    QVector<int> duplicates_2;
    for(auto i=path_2.begin();i!=path_2.end();i++){
        if(path_2.count(*i)>1){
            if(duplicates_2.contains(*i)==false)
                duplicates_2.append(*i);
        }
    }
    //交换重复的基因
    if(duplicates_1.size()>0){
        debugPath(duplicates_1);
        debugPath(duplicates_2);
        Q_ASSERT(duplicates_1.size()==duplicates_2.size());
        for(int i=0;i<duplicates_1.size();i++){
            int tmp= path_1[path_1.indexOf(duplicates_1[i])];
            path_1[path_1.indexOf(duplicates_1[i])]=duplicates_2[i];
            path_2[path_2.indexOf(duplicates_2[i])]=tmp;

        }
        debugPath(path_1);
        debugPath(path_2);
    }
}
//二点交叉
void GA::crossover(int size,int num,double pc,double pm)
{
    Q_ASSERT(this->pool->size()>0);
    Q_ASSERT(size>0&&num>0);
    Q_ASSERT(pc>=0&&pc<=1);
    QVector<QVector<int>>* newPool=new  QVector<QVector<int>>;//新池子
    int time=size;
    while (time--) {
        //随机取出两个路径个体
        int rand_1=QRandomGenerator::system()->bounded(size);
        int rand_2=QRandomGenerator::system()->bounded(size);
        QVector<int> tmpPath_1=this->pool->at(rand_1);
        QVector<int> tmpPath_2=this->pool->at(rand_2);
        debugPath(tmpPath_1);
        debugPath(tmpPath_2);
        //生成一个0~1.0的随机数,如果大于给定的PC值则不交配,否则交配
        double randPC=double(QRandomGenerator::system()->bounded(1.0));
        qDebug()<<"randpc:"<<randPC;
        Q_ASSERT(randPC<1&&rand()>=0);
        if(randPC>pc){
            qDebug()<<"不交配";
            newPool->append(tmpPath_1);
            newPool->append(tmpPath_2);
        }else{
            qDebug()<<"交配";
            int crossPos_1,crossPos_2;
            //获得交叉点
            getRandomSwitchPoint(crossPos_1,crossPos_2,size);
            Q_ASSERT(crossPos_2>crossPos_1);
           // QVector<int> tmpp;
            //交换
            for(int i=crossPos_1;i<crossPos_2;i++){
                int tmppp=tmpPath_1[i];
                tmpPath_1[i]=tmpPath_2[i];
                tmpPath_2[i]=tmppp;
            }
            debugPath(tmpPath_1);
            debugPath(tmpPath_2);
            //去除重复
            removeDuplicates(tmpPath_1,tmpPath_2);
            double randPM=double(QRandomGenerator::system()->bounded(1.0));
            if(randPM<pm){
                qDebug()<<"mutations";
                mutations(tmpPath_1,num);
                mutations(tmpPath_2,num);
                debugPath(tmpPath_1);
                debugPath(tmpPath_2);
            }
            newPool->append(tmpPath_1);
            newPool->append(tmpPath_2);
        }
    }
    delete pool;
    pool=newPool;

}
//变异
inline void GA::mutations(QVector<int> &path,int num)
{
    int randIndex_1=QRandomGenerator::system()->bounded(num);
    int randIndex_2=QRandomGenerator::system()->bounded(num);
    if(randIndex_1!=randIndex_2){
        int tmp=path[randIndex_1];
        path[randIndex_1]=path[randIndex_2];
        path[randIndex_2]=tmp;
    }

}
```

这样遗传算法对于TSP问的的实现就基本写好了
接下来我们设计个可视化界面，方便我们查看遗传算法不同参数对于结果的影响

## 三 设计可视化界面

目标：
1.可以设定PM概率，PC概率，种群大小和迭代次数
2.结果用折线图显示，方便查看

### 1.设计.ui文件

简单设计一下即可，留出给设置变量的lineEdit组件，下面是我设计的界面
![主界面](https://cdn.jsdelivr.net/gh/HK560/MyPicHub@master/res/pic/20211128205510.png)

在文本框输入参数值，点击执行结果按钮计算结果，能够将结果中的最短路径长度和路径显示出来，也能显示耗时

### 2.实现显示折线图

QT中有提供实现相关视图的类，要实现折线图需要用到下面几个头文件：
```C++
#include <QLineSeries>
#include <QValueAxis>
#include <QChartView>
#include <QChart>
```
在这里不对使用的类做过多介绍，可以自行查看QT官方文档，大概用法如下
```C++
QLineSeries *series = new QLineSeries()//QLineSeries类能在图表中中显示数据
series->append(X,Y);//将坐标为（X，Y）的点添加到QLineSeries中
QChart *chart = new QChart();//QChart 类管理图数据、图例和轴的图形表示
```






-----
编写中
