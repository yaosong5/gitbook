当时说的greenplum，集群5台，是开会的时候说的，我当时就按照计划提的

![](https://ws2.sinaimg.cn/large/006tKfTcgy1g16xxe2cewj31200hemy7.jpg)





如果测试节点觉得5台多了的话就,最少四台：

1. 中心节点Master：要实现主备Master，必须要两台，当一台Master宕机后能却保有另一台Master提供服务，故至少两台
2. 存储节点Segment：要测试存储的容错机制，当一台存储节点宕机，是否其他备份的数据能继续提供服务，故需要两台

共计4台