## 此文档记录我对该项目运行过程的理解，也是防止我忘性大，后期维护不知如何下手。

### 项目的整体运行



### 车辆模型全部支持部分

#### 

### 行人领域模型完善部分



### 运动修饰符部分

###### `modifier.py`：

Modifier基类初始化参数为actor_name，和modifier_name。提供基本的获取参数的函数，以及设置属性的函数。

目前来看，`osc2_scenario.py`中关于`visit_behavior_invocation`函数（line:694），其用于读取行为树节点中的修饰符并记录在`modifier_ins`和 `xxxx_modifiers`中。

其中`OSC2Helper.flat_list`是解决参数的多重列表问题，简单俩说就是将列表平铺开来，形成一个一维列表。

有意思的是，几个修饰符代码编写逻辑基本上一样，或许可以为接下来的编写提供参考。`keep_lane`还搞特殊。



若要完成运动修饰符，需要着重关注`osc_scenario.py`和`atomic_behaviors.py`，可能涉及到`atomic_behaviors.py`的补充修改，需要先了解。

|     修饰符号      | 是否实现 | 是否定义modifier类 | 为何种类型 |            定义            |
| :---------------: | :------: | :----------------: | :--------: | :------------------------: |
|       speed       |    是    |         是         |   speed    | 目前仅用于确定对象运动速度 |
|     position      |    是    |         是         |            |  确定对象在某个时间的位置  |
|       lane        |    是    |         是         |            | 根据别的对象，确定所在车道 |
|   acceleration    |    是    |         是         |   speed    |    确定加速度，进行加速    |
|     keep_lane     |    是    |                    |            |    让对象保持在当前车道    |
|   change_speed    |    是    |         是         |   speed    |     改变速度，可增可减     |
|    change_lane    |    是    |         是         |  location  |      向左向右改变车道      |
|   keep_position   |          |                    |            | 在规定时间内，保持位置不变 |
|    keep_speed     |          |                    |            | 在规定时间内，保持速度不变 |
|      lateral      |          |                    |            | 根据别的对象，确定横向距离 |
|        yaw        |          |                    |            |      运动角度左右偏向      |
|    orientation    |          |                    |            | 确定yaw\pitch\roll三个偏角 |
|       alone       |          |                    |            |     沿着某一条路径运动     |
| along_trajectory  |          |                    |            |      沿设定的轨迹运动      |
|     distance      |          |                    |            |      确定对象移动距离      |
| physical_movement |          |                    |            |  确定对象是否具有物理属性  |
| avoid_collisions  |          |                    |            |        是否允许碰撞        |



|      修饰符       |                             用法                             |
| :---------------: | :----------------------------------------------------------: |
|   keep_position   |                       keep_position()                        |
|    keep_speed     |                         keep_speed()                         |
|      lateral      | lateral(distance: length, side_of: vehicle, side: side_left_right <br />[, measure_by: lat_measure_by]<br /> [, <standard-movement-parameters>]) |
|        yaw        | yaw(angle: angle <br />[, <standard-movement-parameters>]) <br />yaw(angle: angle, relative_to: physical_object<br /> [, measure_by: yaw_measure_by ] <br />[, <standard-movement-parameters>]) |
|    orientation    | orientation(yaw: angle  [, pitch: angle] [, roll: angle] <br />[, relative_to: physical_object] [, measure_by: orientation_measured_by] <br />[, <standard-movement-parameters>]) <br /><br />orientation(pitch: angle  [, roll: angle] [, yaw: angle] <br />[, relative_to: physical_object] [, measure_by: orientation_measured_by] <br />[, <standard-movement-parameters>]) <br /><br />orientation(roll: angle  [, yaw: angle] [, pitch: angle] <br />[, relative_to: physical_object] [, measure_by: orientation_measured_by] <br />[, <standard-movement-parameters>]) |
|       alone       | along(route: route <br />[, start_offset: length] [, end_offset: length]<br /> [, <standard-movement-parameters>]) |
| along_trajectory  | along_trajectory(trajectory: trajectory <br />[, start_offset: length] [, end_offset: length] <br />[, <standard-movement-parameters>]) |
|     distance      | distance(distance: length)<br /> [, <standard-movement-parameters>] |
| physical_movement |         physical_movement(option: movement_options)          |
| avoid_collisions  |                avoid_collisions(avoid: bool)                 |



在`visit_behavior_invocation`函数中，对运动修饰符的处理分为两类。一类针对于没有参数的修饰符，一类是有参数的修饰符，此时，在这里就需要对参数进行处理，即，记录参数的数值。在这之后则是进行`process_location_modifier`和`process_speed_modifier`。

对参数是怎么处理的呢？其实就是看osc文件中，osc2语言编写的函数中的参数。关于tuple就是类似于`lane(right_of: ego_vehicle, at: start)`，参数名和数值在一起，有的函数是直接写的，例如`speed(20kph)`，就不为tuple。这也是为什么处理参数时候需要分类讨论



