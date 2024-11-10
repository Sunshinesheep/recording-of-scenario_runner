



# 关于OpenScenario2

### 前言

依旧是怕忘记了，另外一方面，记录一下某些地方的必要性。就是说某些语句是否有实现的必要。



## 基本定义

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



Range类型：

定义：

```
range-constructor ::= 'range' '(' expression ',' expression ')' | '[' expression '..' expression ']'
```

实例：

```
scenario bar:
    aspeed: speed
    afloat: float
    gear: uint
    keep(afloat in [10..20])
    keep(gear in [-1..6])

scenario foo:
    car: vehicle
    do parallel(duration: [10s..20s]):
        car.drive() with:
            speed([80kph..120kph])
        bar(aspeed: [20kph..40kph])
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



### 结构化

struct定义

```
struct traffic_light:
    id: int
    name: string
    pose: pose_3d
    active_colors: list of traffic_light_colors
    country: string
```



actor定义：

```
actor rock:
    kind: string
    weight: mass
    extent: length
    position: position_3d
```



scenario定义：

```
scenario pass_ego:
    ego: vehicle
    passing_car: vehicle

    do parallel:
        ego.drive() with:
            speed(50kph)

        passing_car.drive() with:
            lane(left_of: ego)
            speed(70kph)
```



### 方法

语法：

```
def <method-name>(<argument-list-specification>) [-> <return-type>] <method-implementation>
```

实例：

```
def my_add(x: float, y: float) -> float is expression x+y
```



### 继承

一个场景描述语言，要什么继承？

无条件继承：

```
struct base:
    f1: bool

struct derived inherits base:
    f2: bool          #derived has both f1 and f2 fields
```

有条件继承：

```
extend vehicle:
    is_electric: bool

actor truck inherits vehicle (vehicle_category == truck):
    ...                    # Fields / methods / events unique to truck

actor electric_vehicle inherits vehicle(is_electric == true):
    ...                    # Fields / methods /events unique to electric_vehicle
```



### 事件

定义事件，并且借助`on`、`wait`、`until`等来确定事情发生的条件

```
event <name> [(<argument-list-specification>)] [is <event-specification>]
```

例如

```
on @x.accident:                # When the event happens, do something
     call dut_error(...)
wait distance_to(car2) < 10m   # Wait until the expression is true
event e1 is @e2 if (x > y)     # Emits e1 when e2 happens and the Boolean expression returns true
```



### 覆盖

虽然但是，我还是想吐槽一句，这有必要吗？



## 领域模型

### 实体关系

![](IMG/entity.png)

### 坐标系

![](IMG/coordinate.svg)

### 地图

暂定，涉及到地图导入



### 整体物理模型

![](IMG/all.png)
