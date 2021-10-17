# 方案设计 #

[TOC]

## 编排模型 ##

有向无环图，节点代表一个步骤。

<img src="https://static-1256056882.cos.ap-guangzhou.myqcloud.com/seven%2Fjobflow.DAG.png">

## 编排语法 ##

见于 [使用手册](/doc/manual.md)

## 编排通知机制 ##

### 下发状态 ###

见于 [数据设计](/doc/model.md) **状态消息**章节

### 上报结果 ###

见于 [接口文档](/doc/interface.md) **上报结果**章节

## 编排引擎架构 ##

<img src="https://static-1256056882.cos.ap-guangzhou.myqcloud.com/seven/jobflow.frame.png">

- OSS : API接口。负责编排的创建、查询、触发任务、任务状态等
- DB : 存储编排、历次触发任务、任务进度、定时触发策略等
- MQ : 消息队列
- Dispatcher : 状态流转调度器
- Timer : 定时触发器

## 编排运行机制 ##

名词引入 :
- pipeline : 将编排的一次触发运行，称作一个pipeline

### 正常顺序 ###

先解释一下，MQ中有以下几类消息 :

| 消息 | 生产者 | 消费者 | 描述 |
| ---- | ---- | ---- | ---- |
| 调度命令 | OSS | Dispatcher | 哪个任务要被调度（希望更新一下它的状态流转） |
| 状态流转 | Dispatcher | 业务方Consumer上的Agent | 编排的步骤进度流转 |

如此，对编排运行的状态机的维护，就从OSS抽离出来，专门让Dispatcher负责。

从编排的生命周期来看，分以下阶段来具体论述模块间的调用关系

**创建编排**

- **OSS** 将编排写入 **DB**

**任务触发**

- **OSS** 创建一个新任务，写入 **DB**
- **OSS** 封装一个 **"调度命令"的消息**（表示哪个任务将要被调度），扔给 **MQ**

**任务执行**

- **Dispatcher** 从 **MQ** 拿到 **调度命令**，从中得到 pipeline-id
- **Dispatcher** 调用 **OSS** 接口，获取 任务进度 + 编排DAG
- **Dispatcher** 根据DAG拓扑关系，计算现在应该执行哪些节点，并封装 **"状态流转"的消息**，扔给 **MQ**
- 业务方消费者，会集成我们提供的 **Agent**，它会订阅自己关注的 **"状态流转"的消息**
- 执行完业务逻辑后，**Agent** 调用 **OSS**接口上报 本步骤的结果
- **OSS** 更新任务进度到 **DB**，若本步骤是成功的，则应该继续下一步，做法是再次封装一个 **"调度命令"的消息**，扔给 **MQ**，这就循环到了第一步

### 异常处理 ###

#### 中断 ####

在pipeline上设置一个**中断标记**，当 **Agent** 调用 **OSS**接口上报某步骤的结果后，若发现有这个中断标记，则不会给**MQ**发送调度命令，而是将pipeline设为**已终止**

限制 :
- 中断不会影响当前正在执行的步骤。即不会让**Agent**立即杀死业务逻辑

#### 中断恢复 ####

前提 :
- pipeline是**已终止**状态
- pipeline已执行的步骤中，最后一步是成功的

**OSS** 发送"调度命令"给 **MQ**，重新激活该pipeline

#### 失败继续 ####

场景 : 某步骤失败了，但是经过一些人为的挽救措施，搞好了。这时候想让引擎继续往下执行
前提 : 与中断恢复类似，但要求已执行的步骤中，最后一步是失败的

### pipeline状态变化 ###

<img src="https://static-1256056882.cos.ap-guangzhou.myqcloud.com/seven/jobflow.pipeline.state.png">

## 编排的Agent ##

<img src="https://static-1256056882.cos.ap-guangzhou.myqcloud.com/seven/jobflow.agent.png">

| 角色 | 数量 | 职责 |
| ---- | ---- | ---- |
| receiver | 本消费者关心的routing_key的数量 | 获取routing_key匹配的消息 |
| PV | 1 | 信号量（初始资源 = 本消费者允许的并发数），本质是排队自旋锁 |
| reporter | 1 | 上报编排中单个步骤的执行结果 |

消费过程 :

- **receiver** 拿到消息后，对 **信号量** 执行P操作，抢不到锁则忙等
- **receiver** 抢到锁后，执行对应的业务逻辑（不同receiver所监听的routing_key，对应不同的业务函数）
- **receiver** 对 **信号量** 执行V操作
- **receiver** 调用 **reporter** 上报结果

## 编排定时触发机制 ##

如**编排引擎架构**章节，原本打算由**Timer**模块实现定时触发逻辑，但是定时任务应该有另外专门一个定时引擎来承载。

<img src="https://static-1256056882.cos.ap-guangzhou.myqcloud.com/seven/jobflow.timer.png">
