# 关于OpenScenario2

### 前言

依旧是怕忘记了，另外一方面，记录一下某些地方的必要性。就是说某些语句是否有实现的必要。



### 格式及注释

python类型



### 关键字

|                |           |            |        |          |
| -------------- | --------- | ---------- | ------ | -------- |
| action         | actor     | and        | as     | bool     |
| call           | cd        | cover      | def    | default  |
| do             | elapsed   | emit       | enum   | event    |
| every          | export    | expression | extend | external |
| factor         | fall      | false      | float  | global   |
| hard           | if        | import     | in     | inf      |
| inherits       | int       | is         | it     | K        |
| keep           | kg        | list       | m      | modifier |
| mol            | namespace | nan        | not    | null     |
| of             | offset    | on         | one_of | only     |
| or             | parallel  | rad        | range  | record   |
| remove_default | rise      | s          | sample | scenario |
| serial         | SI        | string     | struct | true     |
| type           | uint      | undefined  | unit   | until    |
| use            | var       | wait       | with   |          |



### 新的变量类型

物理变量：数值+单位

```
physical-literal ::= (float-literal | integer-literal) unit-name
unit-name ::= qualified-identifier
```



### 命名空间

闲得没事搞这玩意



### 单位

在原理已经被收为关键字的单位中，还可以进行拓展，定义新的物理单位。准确来说，涉及的单位都是SI基本单位。

```
unit km of length is SI(m: 1, factor: 1000.0)
```

公式：

```
base_unit_value = unit_value * factor + offset
```

