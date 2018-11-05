title: Google MapReduce
date: 2017-01-17 15:34:08
tags: MapReduce
---
<!-- more -->
# 原理
利用一个输入key/value pair集合来产生一个输出key/value pair集合
* 用户自定义Map函数接受一个输入的key/value pair值，产生一个中间key/value pair值集合；
* MapReduce库把所有具有相同中间key值的value值集合在一起传递给reduce函数；
* 用户自定义Reduce函数接受一个中间key/value的集合，并合并这些value值，形成一个较小的value值的集合。

# 实现

## 执行过程
MapReduce模型有多种不同的实现方式，一般实现方式为：用交换机连接、由普通PC机组成的大型集群。
当用户调用MapReduce函数时，将发生下面的一系列动作：
* 用户程序调用MapReduce库将输入文件分成M个数据片段；
* 用户程序在集群中创建大量的程序副本，包括一个master、多个worker。由master分配任务，共有M个Map任务和R个Reduce任务，master将一个任务分配给一个空闲worker；
* 被分配了map任务的worker程序读取输入数据，解析出key/value pair，传递给用户自定义Map函数，生成中间key/value pair缓存在内存中；
* 缓存中key/value pair通过分区函数分成R个区域，周期性写入到本次磁盘，并将存储位置回传给master；master将这些存储位置传送给Reduce worker；
* 当Reduce worker程序获取到master发送来的数据存储位置信息后，使用RPC从Map worker所在的磁盘上读取这些缓存数据。读取所有数据后，根据key进行排序；
* Reduce worker遍历排序后的中间数据，对每个中间key值，将key值和相关的value值集合传递给用户自定义Reduce函数，并输出到文件；
* 所有Map和Reduce任务完成后，master唤醒用户程序。

成功完成任务后，MapReduce将输出存放在R个输出文件中。

## Master数据结构
存储每一个Map和Reduce任务的状态（空闲、工作中、完成），以及worker机器的标识；Master类似一个数据管道，传递中间文件存储区域位置信息。

## 容错
* worker故障：master周期性ping每个worker。若worker连不上，则master将此worker标记为失效，由此worker完成的Map任务置为空闲状态，之后将安排给其他worker，正在运行的任务也时如此。Map任务需要重新执行，Reduce任务的输出存储在全局文件系统上，不需要再次执行。
* master失效：简单的解决办法，让master周期性将数据结构写入磁盘，即检查点。若此master任务失败，可以从最后一个检查点开始启动另一个master进程。

## 存储位置
尽量将输入数据存储在集群机器的本地磁盘来节省网络带宽，GFS把每个文件按64MB一个block分割，每个Block保存在多台机器上，存在多份拷贝。MapReduce的master在调度Map任务时会考虑输入文件的位置信息，尽量从本地读取。

## 任务粒度
理想情况下，M和R和数量应当比集群中worker的机器数量多很多；实际上，选择合适的M值使得每个独立任务处理的数据为16M到64M。

## 备用任务
解决落伍者问题，在一个MapReduce操作接近完成的时候，master调度备用任务进程来执行剩下的处于处理中的任务，无论谁完成任务，都把这个任务标记为已完成。

# 技巧
* 分区函数：hash(key) mod R；
* 顺序保证：在给定分区中，中间key/value pair数据处理顺序时按照key值增量顺序处理；
* Combiner函数：Map产生中间key值的重复数据会比较多，指定combiner函数在本地将这些记录进行一次合并；
* 输入和输出类型：默认输入类型可满足大部分要求，也可以定义Reader接口实现新的输出类型。类似的，也可以自定义输出类型；
* 副作用：增加一个辅助输出文件；
* 跳过损坏记录：每个worker进程都设置了信号处理函数捕获内存段异常（segmentation violation）和总线错误（bus error），若触发了一个信号，则消息处理函数向master发送处理最后一条记录的序号。master将在跳过特定记录的处理；
* 本地执行：调试中使用MapReduce本地实现版本；
* 状态信息：使用内置HTTP服务器，监控各种状态信息；
* 计数器：统计不同的时间发生次数；