# 流程编排引擎 #

## 简介 ##

管理工作流，运行工作流的引擎。它让接入方将自己的业务逻辑定义成由多个任务组成的一个编排，只需要关心各任务本身的业务逻辑，而不必感知任务之间的调度。

## 能力矩阵 ##

| 类别     | 能力 | 预计第几期 | 是否已实现 |
| ---- | ---- | ---- | ---- |
| 编排能力 | 串行 | 1 |
| 编排能力 | 并行 | 1 |
| 编排能力 | 条件 | 2 |
| 编排能力 | 循环 | 2 |
| 编排运行 | 全流程执行 | 1 |
| 编排运行 | 中断 | 2 |
| 编排运行 | 中断恢复 | 2 |
| 编排运行 | 失败继续 | 2 |
| 编排管理 | 手动触发 | 1 |
| 编排管理 | 定时触发 | 2 |

## 更多 ##

- [设计文档](/doc/design.md)
- [接口文档](/doc/interface.md)
