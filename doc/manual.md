# 使用手册 #

[TOC]

## 编排语法 ##

### 基础 ###

使用YAML格式来写编排，包含三部分 :

- namespace: 名字空间（常把一类业务的编排归于同一个名字空间）
- params : 编排的入参（编排常作为模板，将可变参数抽离出来）
- steps : 步骤

```yaml
namespace: dongfeng

params:
  - desc: 变量描述
    key: xyz
    value: 默认值

steps:
  - id: 1 // 步骤ID，在编排内要唯一
    name: A // 名字空间内要唯一
    desc: A.步骤描述 // 无唯一性要求
    
  - id: 2
    name: B
    desc: B.步骤描述
    upstream: // 上游步骤的ID
      - 1
```

特别说明 **name** 字段，往往我们多个业务内伴有相同操作，可将这种操作抽象为一个step，多个业务（编排）都用到它

### 高阶 ###

#### 结果传递 ####

当step执行完，希望把它的结果传给下游的step，可在step上添加register进行声明

- 在后续step中，就可以像访问 **params** 一样来读取 fieldA、fieldB
- 变量支持任意类型，可嵌套

```yaml
steps:
  - id: 1
    name: A
    desc: A.步骤描述
    register:
      - fieldA
      - fieldB
```

要求step的输出有固定的结构 :

```json
{
  "code": 0,
  "message": "",
  "data": {
    "fieldA": "***",
    "fieldB": "***"
  }
}
```

#### 并行 ####

B、C的上游都是A，故它俩是并行的

```yaml
steps:
  - id: 1
    name: A
    desc: A.步骤描述
    
  - id: 2
    name: B
    desc: B.步骤描述
    upstream:
      - 1

  - id: 3
    name: C
    desc: C.步骤描述
    upstream:
      - 1
```

#### 条件 ####

在单个step上，可加上if条件表达式，若不满足则跳过

- shell语法，支持与、或运算
- 可引用 **params** 中的变量
- 可引用前面的step中 **register** 的变量

```yaml
steps:
  - id: 5
    name: E
    desc: E.步骤描述
    upstream:
      - 2
      - 4
    if: ${xyz} == "aaa"
```

#### 循环 ####

在单个step上，可加上while循环表达式

```yaml
steps:
  - id: 6
    name: F
    desc: F.步骤描述
    upstream:
      - 5
    while: ${xyz} == "bbb"
```

### 综合示例 ###

以 [编排模型](/static/jobflow.DAG.png) 为例 :

```yaml
namespace: dongfeng

params:
  - desc: 变量描述
    key: xyz
    value: 默认值
    
steps:
  - id: 1
    name: A
    desc: A.步骤描述
    
  - id: 2
    name: B
    desc: B.步骤描述
    upstream:
      - 1
      
  - id: 3
    name: C
    desc: C.步骤描述
    upstream:
      - 1
      
  - id: 4
    name: D
    desc: D.步骤描述
    upstream:
      - 3
      
  - id: 5
    name: E
    desc: E.步骤描述
    upstream:
      - 2
      - 4
    if: ${xyz} == "aaa"
    
  - id: 6
    name: F
    desc: F.步骤描述
    upstream:
      - 5
    while: ${xyz} == "bbb"
```

## 消费者 ##

### SDK ###
#### Go ####
### API ###