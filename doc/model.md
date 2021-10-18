# 数据设计 #

[TOC]

## 类关系图 ##

关注核心对象：任务步骤、编排、pipeline

<img src="https://static-1256056882.cos.ap-guangzhou.myqcloud.com/seven/jobflow.class.pipeline.png">

## 对象 ##

### 数据库设计 ###

#### 编排 ####

库 : db_jobflow
表 : t_jobflow
主键 : jobflow_id

| 字段 | 类型 | 备注 | 示例 |
| ---- | ---- | ---- | ---- |
| jobflow_id | bigint | 编排ID | 101 |
| jobflow_namespace | varchar(255) | 编排名字空间 | dongfeng |
| jobflow_name | varchar(255) | 编排备注 | test |
| created_at | timestamp | | |
| updated_at | timestamp | | |

#### 编排的入参 ####

库 : db_jobflow
表 : t_jobflow_param
主键 : (jobflow_id, param_key)

| 字段 | 类型 | 备注 | 示例 |
| ---- | ---- | ---- | ---- |
| jobflow_id | bigint | 编排ID | 101 |
| param_key | varchar(255) | 参数名 | xyz |
| param_type | varchar(255) | 参数类型 | int、float、string |
| param_value | varchar(255) | 参数默认值 | *** |
| param_desc | varchar(255) | 参数备注 | *** |

#### 编排内的任务步骤 ####

库 : db_jobflow
表 : t_jobflow_step
主键 : (jobflow_id, step_id)

| 字段 | 类型 | 备注 | 示例 |
| ---- | ---- | ---- | ---- |
| jobflow_id | bigint | 编排ID | 101 |
| step_id | int | 步骤ID | 5 |
| step_retry_limit | int | 最大重试次数 | 2 |
| step_namespace | varchar(255) | 步骤所属名字空间 | dongfeng |
| step_name | varchar(255) | 步骤名字 | E |
| step_desc | varchar(255) | 步骤备注 | *** |
| step_upstream | varchar(255) | 步骤上游 | 2,4 |
| step_register | varchar(255) | 结果传递 | fieldA,fieldB |
| step_exp_if | varchar(255) | if表达式 | ${xyz} == "aaa" |
| step_exp_while | varchar(255) | while表达式 | ${xyz} == "bbb" |

#### pipeline ####

库 : db_pipeline
表 : t_pipeline
主键 : pipeline_id

| 字段 | 类型 | 备注 | 示例 |
| ---- | ---- | ---- | ---- |
| pipeline_id | bigint | pipeline ID | 1111 |
| pipeline_status | int | 状态 | |
| created_at | timestamp | 何时创建 | |
| started_at | timestamp | 何时真正启动 | |
| ended_at | timestamp | 何时结束 | |
| pipeline_input | text | pipeline的启动参数 | {"xyz":"aaa"} |

pipeline_status :

| 状态值 | 含义 |
| ---- | ---- |
| 1 | 待调度 |
| 2 | 运行中 |
| 11 | 等待中断步骤执行完 |
| 12 | 已结束-终止 |
| 3 | 已结束-失败 |
| 0 | 已结束-成功 |

#### pipeline的各步骤的状态 ####

库 : db_pipeline
表 : t_pipeline
主键 : (pipeline_id, step_id)

| 字段 | 类型 | 备注 | 示例 |
| ---- | ---- | ---- | ---- |
| pipeline_id | bigint | pipeline ID | 1111 |
| step_id | int | 步骤ID | 1 |
| step_status | int | 步骤状态 | |
| step_retry_limit | int | 最大重试次数 | 2 |
| step_retry_count | int | 当前重试次数 | 0 |
| step_res_code | int | 步骤返回码 | 0 |
| step_namespace | varchar(255) | 步骤所属名字空间 | dongfeng |
| step_name | varchar(255) | 步骤名字 | E |
| step_desc | varchar(255) | 步骤备注 | *** |
| step_upstream | varchar(255) | 步骤上游 | 2,4 |
| step_register | varchar(255) | 结果传递 | fieldA,fieldB |
| step_exp_if | varchar(255) | if表达式 | ${xyz} == "aaa" |
| step_exp_while | varchar(255) | while表达式 | ${xyz} == "bbb" |
| step_input | text | 步骤输入 | {"xyz":"aaa"} |
| step_res_msg | text | 步骤返回报错信息 | |
| step_res_data | text | 步骤返回数据 | {"fieldA":"xxx","fieldB":"yyy"} |

step_status :

| 状态值 | 含义 |
| ---- | ---- |
| 1 | 未调度 |
| 2 | 跳过 |
| 3 | 已下发 |
| 4 | 执行中 |
| 0 | 已结束-成功 |
| 5 | 已结束-失败 |

### 消息设计 ###

#### 调度命令 ####

```json
{
  "pipeline_id": 1111
}
```

#### 状态流转 ####

routing_key = ${namespace}.${name}

```json
{
  "pipeline_id": 1111,
  "step_id": 1,
  "step_namespace": "dongfeng",
  "step_name": "A",
  "params": {
    "xyz": "aaa"
  }
}
```
