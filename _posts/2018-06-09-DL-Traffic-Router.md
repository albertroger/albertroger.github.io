---
title:THE DEEP LEARNING VISION FOR HETEROGENEOUS NETWORK TRAFFIC CONTROL: PROPOSAL, CHALLENGES, AND FUTURE PERSPECTIVE

description:在网络路由问题上，首次引入Deep Learning

categories:
 - tutorial

date: 2018-06-06 00:38:00
---
## 背景：

&emsp;&emsp;网络流量控制是当今以移动为中心的异构网络的重要组成部分，随着物联网的不断发展，为用户提供各种服务和体验。
&emsp;&emsp;下一代有线和无线异构网络需要一个智能机制来控制网络流量的大幅增长，这些流量来自不断变化的移动网络，云计算，大数据处理和机器对机器连接等网络环境。

## 难点：

&emsp;准确的表征DL的输入输出；

## 贡献：

&emsp;1、第一次证明了利用深度学习神经网络来显着改善异构网络流量控制的可行性，并证实可显著改善流量控制；

&emsp;2、搭建了一个DL框架；

## 结构及流程：

&emsp;采用的大体结构如下：

![流程图](/Users/asotree/Documents/GitHub/albertroger.github.io/pic/DL-Traffic-Router/DL流程.png)
&emsp;如图，系统的输入为Traffic-Pattern，而输出为Router-path；其网络的具体结构如下：
![网络结构](/Users/asotree/Documents/GitHub/albertroger.github.io/pic/DL-Traffic-Router/DL网络结构.png)
&emsp;其中，输入层共有$N$个神经元，$N$代表的是目标网络的路由数量,用一个向量来表示$\lbrace x_1 , x_2, \cdots, x_N \rbrace$,其中，$x_i$表示在规定时隙内，第i个路由的入包；

&emsp;输出为下一条的路由，也是用一个向量表示$\lbrace y_1,y_2,\cdots, y_N \rbrace$,其中，$y_i$的取值为0或1。在确定以怎样的输入和输出来搭建网络的过程中，文章设N为16进行了多次对比；

![table](/Users/asotree/Documents/GitHub/albertroger.github.io/pic/DL-Traffic-Router/DL网络输入输出.png)

## 训练

&emsp;1、输入：
- N:核心/骨干网中的路由器数量；
- I:内部路由器的数量，$I<N$；

&emsp;2、训练过程中的两个核心步骤：

- 逐层贪心训练：
  - 第一次训练，神经网络包含输入、输出和一个隐层[^1]；
  - 完成第一次训练之后，去掉上一轮结构中的输出层，作为下一次训练的输入，并在其后增加一个隐藏层，其中每层的神经元为16个，共需要4层隐藏层[^2]；
- BP微调参数：
  - 再完成以上的逐层贪心训练之后，再将训练好的整个结构作为一个整体，使用BP进行微调[^3]

&emsp;3、训练的网络不止一个：

- 对于每一个边界路由，他需要训练$N-I-1$个网络；
  - 因为他有$N-I-1$个目的边界路由；
- 对于每一个内部路由，需要训练$N-I$个网络；
  - 因为有$N-I$个目的路由；

### 参数传输：

&emsp;在训练结束后：

- 边缘路由器将其得到的$N-I-1$个权重矩阵以多播的方式发送个其余的边缘路由器；
- 内部路由也将其得到的$N-I$个权重矩阵发送个边缘路由；
- 亦即，每个边缘路由拥有所有的权重矩阵[^4]；

### 预测：
&emsp;使用下图来具体说明：

![示例](/Users/asotree/Documents/GitHub/albertroger.github.io/pic/DL-Traffic-Router/测试示例.png)
- 使用$D_{ij}$来表示路由i到路由j的网络，而$W_{ij}$则表示其权重；
- 当$R_2$要向$R_{12}$发送数据包时，使用输入$x$和$D_{2,12}$$W_{2,12}$,来预测下一跳路由；
- 得到下一跳路由之后，以此类推，直到到达目的节点[^5]；

## 性能测试：
&emsp;模拟网络有16个路由器，并假设只有边界路由产生数据，内部路由只是转发数据，带宽为$8Mb/s$，每个节点的buffer大小不设限，总数据包生成速率为$16.32Mb/s$:

- 不同DL网络结构的性能对比：
  ![网络对比](/Users/asotree/Documents/GitHub/albertroger.github.io/pic/DL-Traffic-Router/DL网络结构测试.png)
  &emsp;从图中可以看出，当每层的神经元个数确定后，层数越多MSE越大且每层的神经元个数为16 与18差不多；考虑到参数矩阵的大小及训练速度，最终选择了4层，每层16个unit；

- 吞吐量测试：

![吞吐量测试](/Users/asotree/Documents/GitHub/albertroger.github.io/pic/DL-Traffic-Router/DL吞吐量测试.png)
&emsp;对比了传统OSPF和DL-Traffic-Router的吞吐量:

&emsp;其中，DL-Traffic-Router的总吞吐量达到了$16.13Mb/s$左右，而总的发包速率为$16.3Mb/s$，因为DL方法得到路径的速度快，可以一定程度上避免拥塞和丢包[^6]
- 平均每跳时延测试：
![平均每条时延测试](/Users/asotree/Documents/GitHub/albertroger.github.io/pic/DL-Traffic-Router/DL每跳时延测试.png)

&emsp;对比了传统OSPF和DL-Traffic-Router之间的每一跳时延；DL方法明显优于OSPF，因为DL的寻找路径更快；

- 信令开销：

  ![信令开销对比](/Users/asotree/Documents/GitHub/albertroger.github.io/pic/DL-Traffic-Router/DL信令测试.png)
&emsp;对比了OSPF和DL-Traffic-Router之间的信令开销；可以看出DL方法远低于OSPF，这是因为，**<u>还不懂</u>**

## 一点想法：
- 每一个路由训练多个DL网络，边缘网络还保存了多个DL网络的权重，能不能整合到一个网络中；























[^1]: 这里的输入与输出，应当就是之前定义的Traffic-Pattern和Router-Path；
[^2]: 整个结构的深度、每层的神经元个数以及连接方式，会在性能测试中说明，也就是说，是试出来的；
[^3]: 不就是整体训练一遍吗？那在增加最后一层隐藏层的时候，不就是一样的吗？
[^4]: 每个边界路由就可以预测整条路径；
[^5]: 能这么做的保障就是，所有的边缘路由保存了所有的权重值;
[^6]: 为什么计算速度快，就能一定程度上避免拥塞和丢包？

