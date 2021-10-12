# 使用手册 #

[TOC]

## 编排语法 ##

### 串行 ###

### 并行 ###

### 环境变量 ###

### 条件 ###

### 循环 ###

### 综合示例 ###

以 [编排模型](/static/jobflow.DAG.png) 为例 :

```yaml
env:
  - desc: 环境变量描述
    key: XYZ
    value: 默认值

steps:
  - id: 1
    name: A
    
  - id: 2
    name: B
    upstream:
      - 1
      
  - id: 3
    name: C
    upstream:
      - 1
      
  - id: 4
    name: D
    upstream:
      - 3
      
  - id: 5
    name: E
    upstream:
      - 2
      - 4
    if:
      key: XYZ
      value: aaa
      
  - id: 6
    name: F
    upstream:
      - 5
    while:
      key: XYZ
      value: bbb
```

## 消费者 ##

### SDK ###
#### Go ####
### API ###