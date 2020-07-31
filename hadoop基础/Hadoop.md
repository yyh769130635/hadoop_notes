# Hadoop #

## 简述 ##
### 适用场景 ###
- 高吞吐，对时延没有要求
- 采用流式数据访问方式，一次写入，多次读取
- 廉价
- 高容错
- 为数据存储提供所需的拓展能力

### 不适合场景 ###
- 牺牲时延
- 大量小文件，文件的元数据保存在NameNode中，文件系统数量受限于NameNode内存大小
- 任意文件的修改HDFS采用追加的模式写入数据，不支持文件任意offset的修改，不支持多个写入器

## 架构 ##
四个部分呢，Cient, NameNode, DataNode, Secondary NameNode

### 1.Client ###
-  文件切分成一个个block，存储
-  与NameNode交互，获取文件的位置信息
-  与DataNode 交互，读写数据
-  提供命令管理和访问HDFS,比如启动和关闭

### 2.NameNode ###
- 管理HDFS的名称空间
- 管理数据块block映射信息
- 配置副本的策略
- 处理客户端读写请求

### 3.DataNode ###
- 存储
- 读写操作

### 4.Secondary NameNode ###
并非NameNode的热备，NameNode 挂了，它并不能马上替换NameNode并提供服务
- 辅助NameNode
- 定期合并fsimage和fsedits，并推送给NameNode
- 紧急情况下，可辅助恢复NameNode

## NameNode ##
- 在内存中保存着整个文件系统的名称、空间和文件数据块的地址映射
- 整个HDFS可存储的文件受限于NameNode的内存大小


###### 1.NameNode元数据信息

![nameNode](https://github.com/yyh769130635/hadoop_notes/blob/master/images/namenodepng.png)

