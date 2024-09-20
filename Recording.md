## 2024.8.5~2024.8.9

1.配置Carla和carla-simulator/scenario_runner的运行环境，并成功运行项目。

2.初步了解scenario_runner项目中os2模块的基本结构，理解项目运行的基本流程。

3.开始学习ASAM的OpenScenario2标准以及Carla的相关接口调用。



下周计划：

1. 完成OpenScenario2标准的学习
2. 尝试完善项目中的pedestrian模型





## 2024.8.12~2024.8.16

openscenario2的标准：

https://publications.pages.asam.net/standards/ASAM_OpenSCENARIO/ASAM_OpenSCENARIO_DSL/latest/index.html

制作一个简单的脑图：





下周计划：

测试scenario_runner/examples中的样例，确定项目中未支持os2标准的地方。





## 2024.8.19~2024.8.23

1.当前车辆支持情况：

| 车辆品牌    | 具体型号              |
| ----------- | --------------------- |
| tesla       | tesla.model3          |
| lincoln     | lincoln.mkz2017       |
| carlamotors | carlamotors.carlacola |
| jeep        | jeep.wrangler_rubicon |

目前来看，项目中对carla中车辆描述均使用车辆具体型号的名称。具体的实现则是通过根据osc2文件中提及的车辆名称，确定参数model，通过调用carla的API，初始化车辆模型。



现已经完善。

方法：对vehicle.py进行扩写，补充carla所支持的原始车辆模型。

完善后可支持的车辆：

https://carla.readthedocs.io/en/0.9.13/bp_library/#vehicle

如上链接，carla自带的原始的车辆类型，在项目中均可在osc2中实现。

但目前只可指定车辆类型，对于颜色等其他属性的修改尚未支持，可作为后续任务。



2.对项目现存全部osc2文件进行测试

除wait_elapsed.osc外均可正常运行。

其中，对change_speed()函数的测试中，在修改目标车辆的速度时，只可加速，而减速效果并不明显。可能的原因：项目中对该函数的编写尚未完善。



下周计划：

着手完善pedestrian领域模型



## 2024.8.26~2024.8.30

完善了pedestrian.py。通过Carla官方文档，确定了walker的属性，并在项目中对pedestrians类进行重新编写，设置相关函数。

 

下周计划：

继续完善pedestrians的领域模型。使其可在OSC文件中表示，并且编译成功。



## 2024.9.2~2024.9.6

对osc2_scenario_configuration.py等文件进行修改，对其中visit node模块进行完善。使其在原有读取编译vehicle基础上，增加对osc2文件中person的解释。但仍存在识别不了关键字的问题。经过测试得知，在项目中，有部分方法是针对vehicle模型，而对pedestrians模型并不兼容。后续需要解决此类问题。



![s](IMG/Snipaste_2024-09-06_16-53-41.png)

下周计划：

继续解决pedestrians在osc2中的编译问题。借助Carla提供的python API，连接Carla，使osc2描述的行人可以在设定的场景中展示，并表现出相关行为。



## 2024.9.9~2024.9.13

通过对osc_scenario_configuration.py、pedestrian.py的修改，行人模型以person在osc文件中表示，现已经可以通过编译。

![Snipaste_2024-09-13_15-54-06](IMG/Snipaste_2024-09-13_15-54-06.png)

但又出现新的问题，无论选择哪种模型，程序始终默认设置pedestrian为0001号模型，应该是carla_data_provider.py中的request_new_actor函数中的model参数传输出现问题。

通过阅读代码得知，如果设置行人的速度，位置等信息，需要对osc2_scenario.py中的process_speed_modifier等函数进行改写，因为目前这些函数以车辆模型为主，相关属性的调用均是vehicle类中的属性。



下周计划：

解决行人模型选择的问题，可自由选择行人模型。对相关函数重写，使其适用于行人模型，可以借此设置行人的行为动作。



## 2024.9.16～2024.9.20

完成行人领域模型的完善，现已经可以在osc2文件中以peron:Walkerxx（xx表示选择的行人模型，支持01～48）来在场景中设置行人，并且可以通过speed、、position、lane等设置其行走的位置、速度等属性。

![walker1](IMG/walker1.gif)

下周计划：

开始着手完善运动修饰符

