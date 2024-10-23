## 此文档记录我对该项目运行过程的理解，也是防止我忘性大，后期维护不知如何下手。

### 项目的整体运行

​	不理清一下，真让人头大。数不清的类，数不清的继承，数不清的调用。哎哎哎……

​	首先我们只针对支持Openscenario2的部分，从`scenario_runner.py`开始，从主函数main开始，根据输入命令，先一步处理参数，其中最重要的就是osc文件的位置。然后，就是一切的开端。先获取`scenario_runner = ScenarioRunner(arguments)`，之后再运行场景`result=scenario_runner.run()`。

​	分为两步来说。先说获取阶段：初始化ScenarioRunner类，其中init时构造Scenariomanager类，可以理解为对生成的场景的管理。

​	第二步，run()，这里在、run的函数就可以看到，需要选择run_osc2()。转到run_osc2的函数后，正式进入osc2的模块。先初始化`config=OSC2SecenarioConfiguration`类，之后运行`_load_and_run_scenario(config)`。

#### config初始化

​	初始化config就涉及到`osc2_scenario_configuration.py`，这里初始化的时候就运行这一函数`self._parse_osc2_configuration()`，以及之后还初始化 了ConfigInit类。先看`self._parse_osc2_configuration()`，直接写明是解释osc2文件。既然都这么说了，那就结下往下看。

​	因为ConfigInit继承了ASTVisitor，那么开始遍历抽象语法树吧。

> [!CAUTION]
>
> 重要部分



#### 运行config

​	运行这部分又涉及到OSC2Scenario类，初始化`scenario = OSC2Scenario()`,转入`osc2_scenario.py`。

> [!CAUTION]
>
> 重要部分

​	接下来，轮到已构造的Scenariomanager类登场，进行场景载入和场景运行。场景载入部分`load_scenario()`，好像没什么说的。场景运行部分`run_scenario`，貌似也没啥说的。



#### 后面就是撤销场景了，暂时不管

​	待定……



### 车辆模型全部支持部分

​	这部分没什么难度，主要就是针对于`vehicle.py`进行修改，其实观察之前的车辆模型的定义就可知。先定义最初的基类，再在基类的基础上设置各个车辆类型的定义。项目目前对车辆的定义也仅仅停留在设置`model`参数上，根据carla官方文档提供的各类已存在车辆模型的`model`来进行补充。最后对`osc2_congifuration.py`中的vehicle_type进行补充，就OK了。

### 行人领域模型完善部分

​	代码量实际并不大，但是弄懂确实花费了不少功夫。就目前我所知，通过osc2语言来在场景中构建物理模型，本质上还是通过Carla的API调用。Carla的官方文档中写明，blueprint提供了目前carla已支持的模型。其中就有行人模型，从01~48号人物模型。那么，对于`pedestrians.py`的编写，借鉴`vehicle.py`不失为一个好主意。同样设置一个基类，再通过继承，再编写每个行人模型不同model参数。这里不难。

​	坑人的是接下来，如何让这个”编译器“能够识别定义行人的osc2语言，然后在场景中构建出来呢？先在`osc2_configuration.py`中把pedestrians_type写一下吧……

​	主要还是在`osc_configuration.py`里面编写，老样子，照葫芦画瓢，参考对车辆模型的处理。在函数中写个基本的函数来处理。

​	其实在看了`carlaprovidedata.py`，发现这里已经处理了osc2中对物理模型的语言的编译。所以实际上我需要做的也只有定义行人模型，再加上一些对行人特定的函数来处理罢了。

​	这样来看似乎也挺容易，但真去编写的时候，头都大了。 

### 运动修饰符部分

###### `modifier.py`：

​	Modifier基类初始化参数为actor_name，和modifier_name。提供基本的获取参数的函数，以及设置属性的函数。

​	目前来看，`osc2_scenario.py`中关于`visit_behavior_invocation`函数（line:694），其用于读取行为树节点中的修饰符并记录在`modifier_ins`和 `xxxx_modifiers`中。

​	其中`OSC2Helper.flat_list`是解决参数的多重列表问题，简单俩说就是将列表平铺开来，形成一个一维列表。

​	有意思的是，几个修饰符代码编写逻辑基本上一样，或许可以为接下来的编写提供参考。`keep_lane`还搞特殊。



​	若要完成运动修饰符，需要着重关注`osc_scenario.py`和`atomic_behaviors.py`，可能涉及到`atomic_behaviors.py`的补充修改，需要先了解。

|     修饰符号      | 是否实现 | 是否定义modifier类 | 为何种类型 |            定义            |
| :---------------: | :------: | :----------------: | :--------: | :------------------------: |
|       speed       |    是    |         是         |   speed    | 目前仅用于确定对象运动速度 |
|     position      |    是    |         是         |            |  确定对象在某个时间的位置  |
|       lane        |    是    |         是         |            | 根据别的对象，确定所在车道 |
|   acceleration    |    是    |         是         |   speed    |    确定加速度，进行加速    |
|     keep_lane     |    是    |                    |            |    让对象保持在当前车道    |
|   change_speed    |    是    |         是         |   speed    |     改变速度，可增可减     |
|    change_lane    |    是    |         是         |  location  |      向左向右改变车道      |
|   keep_position   |   是？   |                    |            | 在规定时间内，保持位置不变 |
|    keep_speed     |    是    |                    |            | 在规定时间内，保持速度不变 |
|      lateral      |          |         是         |            | 根据别的对象，确定横向距离 |
|        yaw        |          |         是         |            |      运动角度左右偏向      |
|    orientation    |          |         是         |            | 确定yaw\pitch\roll三个偏角 |
|       alone       |          |         是         |            |     沿着某一条路径运动     |
| along_trajectory  |          |         是         |            |      沿设定的轨迹运动      |
|     distance      |          |         是         |            |      确定对象移动距离      |
| physical_movement |          |         是         |            |  确定对象是否具有物理属性  |
| avoid_collisions  |          |         是         |            |        是否允许碰撞        |



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



​	在`visit_behavior_invocation`函数中，对运动修饰符的处理分为两类。一类针对于没有参数的修饰符，一类是有参数的修饰符，此时，在这里就需要对参数进行处理，即，记录参数的数值。在这之后则是进行`process_location_modifier`和`process_speed_modifier`。

​	对参数是怎么处理的呢？其实就是看osc文件中，osc2语言编写的函数中的参数。关于tuple就是类似于`lane(right_of: ego_vehicle, at: start)`，参数名和数值在一起，有的函数是直接写的，例如`speed(20kph)`，就不为tuple。这也是为什么处理参数时候需要分类讨论



