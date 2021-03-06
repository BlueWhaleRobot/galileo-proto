# <a href="#" id="start"></a>伽利略视觉导航系统串口通信协议

导航系统主机带一个usb转串口模块(可以选ttl电平或者rs232电平，默认ttl电平)，通过这个串口用户可以使用导航系统的功能和获取导航系统的状态，下文将这个串口命名为导航串口。

导航串口波特率为115200，8个数据位，1个停止位，无奇偶校验。

## <a href="#" id="send"></a>1. 导航串口下发的数据包

发布频率固定为30hz，即导航系统每秒自动通过导航串口下发30个数据包。

数据包格式：包头+长度+数据内容+结束符0x00

包头：占3个字节，0xcd 0xeb 0xd7

长度：占1个字节，长度不包括包头和长度本身字符，这个长度还包括字符串结束符0x00，当前值固定为21*4+1=85=0x55。

内容：由12个4字节小端模式二进制表示的数字串联在一起构成。

|包头|长度|nav_status|visual_status|map_status|…|数据n|…|busy_status|结束符|
|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
|0xcd0xeb0xd7|0x55|4个字节|4个字节|4个字节|…|4个字节|…|4个字节|0x00|

完整数据包内容构成一个c语言结构体，结构体具体构成如下所示：

```cpp
typedef struct{
  int nav_status;// 导航服务状态，0表示没开启closed，1表示开启opened。
  int visual_status;// 视觉系统状态，-1标系视觉系统处于关闭状态，0表示没初始化uninit，1表示正在追踪tracking,2表示丢失lost,1和2都表示视觉系统已经初始化完成。
  int map_status; // 建图服务状态，0表示未开始建图，1表示正在建图
  int gc_status; // 内存回收标志，0表示未进行内存回收，1表示正在进行内存回收
  int gba_status; // 闭环优化标志，0表示未进行闭环优化，1表示正在进行闭环优化
  int charge_status; // 充电状态，0 free 未充电状态, 1 charging 充电中, 2 charged 已充满，但仍在小电流充电, 3 finding
                     // 寻找充电桩, 4 docking 停靠充电桩, 5 error 错误
  int loop_status; // 是否处于自动巡检状态，1为处于，0为不处于。
  float power;// 电源电压【946】v。
  int target_numID;// 当前目标点编号,默认值为-1表示无效值，当正在执行无ID的任务是值为-2，比如通过Http API 创建的导航任务。
  int target_status;// 当前目标点状态，0表示已经到达或者取消free，1表示正在前往目标点过程中working,2表示当前目标点的移动任务被暂停paused,3表示目标点出现错误error,默认值为-1表示无效值。
  float target_distance;// 机器人距离当前目标点的距离，单位为米，-1表示无效值，该值的绝对值小于0.01时表示已经到达。
  int angle_goal_status;// 目标角度达到情况，0表示未完成，1表示完成，2表示error,默认值为-1表示无效值。
  float control_speed_x;// 导航系统计算给出的前进速度控制分量,单位为m/s。
  float control_speed_theta;// 导航系统计算给出的角速度控制分量,单位为rad/s。
  float current_speed_x;// 当前机器人实际前进速度分量,单位为m/s。
  float current_speed_theta;// 当前机器人实际角速度分量,单位为rad/s。
  unsigned int time_stamp;// 时间戳,单位为1/30毫秒，用于统计丢包率。对于ROS API时间戳在状态的header里面
  float current_pose_x; // 当前机器人在map坐标系下的X坐标,此坐标可以直接用于设置动态插入点坐标
  float current_pose_y; // 当前机器人在map坐标系下的Y坐标
  float current_angle; // 当前机器人在map坐标系下的z轴转角(yaw)
  int busy_status; //当busy为true时系统将仍然后接收新指令，但是不会立即处理。当系统退出busy状态后再处理消息
}Galileo_Status;
```

## <a href="#" id="receive"></a>2. 导航串口可以接收的指令

最大支持100hz上传频率，每条命令由包头+数据长度+数据内容构成。指令可以
串联一起发送但是不推荐这样使用，因为每条指令是否有效还要取决于导航系统状态。

### 2.a. 开启、关闭、重载导航系统

|0xcd|0xeb|0xd7|0x02|0x6d|0xXX|
|:--:|:--:|:--:|:--:|:--:|:--:|
|包头|包头|包头|数据长度|“m”|值为4表示关闭，0表示为开启|

通过本命令可以开启、关闭导航系统，导航系统开启后需要对视觉系统进行初始化，视觉系统初始化过程机器人可能会原地旋转（当前视角无法初始化时需要调整视角），视觉初始化后才能使用导航系统的功能。

因为视觉系统初始化时间较长，所以不建议频繁地进行关闭、开启导航系统操作。导航系统的状态可以由nav_status获得、视觉系统的状态可以由visual_status获得。

现在导航系统支持地图的动态切换，不再需要重启导航系统便可以载入不同的导航地图。同时地图也支持自动切换，导航系统可根据当前环境自动切换至对应的地图。在地图切换后需要重新载入对应地图的导航路径，此时即可用到重载功能。

|0xcd|0xeb|0xd7|0x02|0x6d|0xXX|
|:--:|:--:|:--:|:--:|:--:|:--:|
|包头|包头|包头|数据长度|“m”|9|

重载时不会对导航地图进行处理，只重新载入当前的路径。所以调用重载之前需要设置好地图和路径。

### 2.b. 发布下一个目标点

|0xcd|0xeb|0xd7|0x02|0x67|0xXX|
|:--:|:--:|:--:|:--:|:--:|:--:|
|包头|包头|包头|数据长度|“g”|值为0到255，表示目标点编号|

发布前需要检查视觉系统有没有完成初始化，如果没有完成初始化，系统会忽略这条指令。

视觉系统如果已经初始化了，主机接收这条指令后，先根据当前目标点状态判断是否要先执行取消当前移动任务的操作。当前目标点状态为free或者error时不执行任何动作直接继续下一步，当前目标点状态为working或paused时会自动取消当前目标点然后继续下一步。

接着系统会将当前目标点编号更新成设定值，再验证目标点编号是否有效。

#### 2.b.1 如果目标点有效，立即控制机器人往目标点移动，同时目标点的状态更新成working

#### 2.b.2 如果目标点无效，反馈的目标点状态更新成error，同时机器人保持不动

#### 2.b.3 目标到达后，目标点状态会自动更新成free，机器人退出移动任务同时保持不动

### 2.c 暂停、继续、取消当前目标点

|0xcd|0xeb|0xd7|0x02|0x69|0xXX|
|:--:|:--:|:--:|:--:|:--:|:--:|
|包头|包头|包头|数据长度|“i”|值为0表示暂停，值为1表示继续，值为2表示取消|

发布暂停指令后，系统会检查目标点状态，状态为working时本命令才有效，为其它状态则忽略。指令判断有效后机器人会停止运动、保存移动任务同时目标点状态会更新成paused。

发布继续指令后，系统会检查目标点状态，状态为pause时本命令才有效，为其它状态则忽略。指令判断有效后机器人继续往目标点运动同时反馈的目标点状态变成working，

发布取消指令后，系统会检查目标点状态，状态不时free时本命令才有效，为其它状态则忽略。指令判断有效后机器人停止运动，退出移动任务,目标状态更新成free。

#### 2.d 动态添加目标点

添加目标点

|0xcd|0xeb|0xd7|0x0a|0x67|0x69|0xXX|0xXX|0xXX|0xXX|0xXX|0xXX|0xXX|0xXX|
|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
|包头|包头|包头|数据长度|“g”|"i"|x和y的float32坐标，共八位|

接收到此命令后，系统会在当前目标点的最后添加一个目标点。可以利用动态添加目标点功能实现移动到某个坐标位置。此坐标为map坐标系下坐标。

重置目标点

|0xcd|0xeb|0xd7|0x03|0x67|0x72|0x00|
|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
|包头|包头|包头|数据长度|“g”|"r"|0|

接收到此命令后，系统将清空所有动态添加的目标点，只保留最初的目标点。此功能可以用于动态点的初始化。保证使用正确的目标点id。

### 2.e.手动遥控速度指令

|0xcd|0xeb|0xd7|0x02|0x66|0xXX|
|:--:|:--:|:--:|:--:|:--:|:--:|
|包头|包头|包头|命令长度|前进指令|速度大小，为最大速度百分比，数值范围为0到100|

|0xcd|0xeb|0xd7|0x02|0x62|0xXX|
|:--:|:--:|:--:|:--:|:--:|:--:|
|包头|包头|包头|命令长度|后退指令|速度大小，数值范围为0到100|

|0xcd|0xeb|0xd7|0x02|0x63|0xXX|
|:--:|:--:|:--:|:--:|:--:|:--:|
|包头|包头|包头|命令长度|左转指令|速度大小，数值范围为0到100|

|0xcd|0xeb|0xd7|0x02|0x64|0xXX|
|:--:|:--:|:--:|:--:|:--:|:--:|
|包头|包头|包头|命令长度|右转指令|速度大小，数值范围为0到100|

|0xcd|0xeb|0xd7|0x02|0x73|0xXX|
|:--:|:--:|:--:|:--:|:--:|:--:|
|包头|包头|包头|长度|停止指令|制动量大小，数值范围为0到100|

可以控制机器人的移动，这些指令独立于视觉导航系统，导航系统工不工作都会执行这些指令，因此可以用来实现机器人的遥控功能。

这些指令生效后不会影响目标点状态，即遥控的速度指令和导航系统的速度指令是并行的会相互覆盖。如果导航运动过程中机器人发生错误需要切换到遥控，用户应该先发布目标点取消指令，否则可能会出现机器人还是不受自己控制的情况。

### 2.f 主机关机指令

|0xcd|0xeb|0xd7|0x02|0xaa|0x44|
|:--:|:--:|:--:|:--:|:--:|:--:|
|包头|包头|包头|数据长度|数据1|数据2|

主机已经配置成通电后自动开机，这样可以避免按键的不方便。使用本条指令可以实现串口关机。

### 2.g 角度设定指令

|0xcd|0xeb|0xd7|0x03|0x61|0xXX|0xXX|
|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
|包头|包头|包头|数据长度|“a”|角度值符号，0表示正，1表示负|角度值，范围为0到180|

系统收到指令后，会控制机器人原地旋转到设定角度值。

参考角度需要视觉定位系统提供，因此这条指令需要视觉系统已经初始化完成，即视觉系统状态不为uninit，否则指令无效。

发布这条指令前还需要检查目标点状态，如果目标点状态不为free或者error,本指令也无效，这样可以防止和导航系统的运动控制冲突。

本指令结果由angle_goal_status状态反映。

### 2.h 自动巡检状态控制

自动巡检功能即为机器人按照当前所有的目标点循环移动。在到达目标点时停靠特定时间，此时间可以通过api进行设置。

开启自动巡检功能

注意只有在开启导航服务之后才可以启动自动巡检功能。当此功能成功开启之后galileo_status中的loopFlag会设置成True。

|0xcd|0xeb|0xd7|0x02|0x6d|0x05|
|:--:|:--:|:--:|:--:|:--:|:--:|
|包头|包头|包头|数据长度|“m”|5|

关闭自动巡检功能

|0xcd|0xeb|0xd7|0x02|0x6d|0x06|
|:--:|:--:|:--:|:--:|:--:|:--:|
|包头|包头|包头|数据长度|“m”|6|

设置自动巡检目标点停靠时间

|0xcd|0xeb|0xd7|0x03|0x6d|0x05|0xXX|
|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
|包头|包头|包头|数据长度|“m”|5|目标点停靠时间，单位为秒|

### 2.i 调度系统导航控制

调度导航状态是配合拉格朗日调度系统使用的导航状态，在此状态下机器人的导航任务由调度系统发布。

开启调度导航

|0xcd|0xeb|0xd7|0x02|0x6d|0x07|
|:--:|:--:|:--:|:--:|:--:|:--:|
|包头|包头|包头|数据长度|“m”|7|

关闭调度系统导航

|0xcd|0xeb|0xd7|0x02|0x6d|0x08|
|:--:|:--:|:--:|:--:|:--:|:--:|
|包头|包头|包头|数据长度|“m”|8|

重载调度系统程序，不关闭导航地图，用于动态切换地图。在调用前需要设置好当前的地图和路径。
|0xcd|0xeb|0xd7|0x02|0x6d|0x0a|
|:--:|:--:|:--:|:--:|:--:|:--:|
|包头|包头|包头|数据长度|“m”|10|

### 2.j 开始建立地图，结束建立地图,保存和更新地图

|0xcd|0xeb|0xd7|0x02|0x56|0xXX|
|:--:|:--:|:--:|:--:|:--:|:--:|
|包头|包头|包头|数据长度|“V”|0表示为开启，值为1表示关闭，2表示保存,3表示更新地图|

### 2.k 自动充电相关指令

|0xcd|0xeb|0xd7|0x02|0x6a|0xXX|
|:--:|:--:|:--:|:--:|:--:|:--:|
|包头|包头|包头|数据长度|“j”|0表示为开始充电，值为1表示停止充电，2表示保存充电桩位置|

## <a href="#" id="history"></a>3. 版本历史

|版本|时间|更新说明|
|:--:|:--:|:--:|
|V1.0|2018-3-15|初稿，覆盖基本功能|
|V1.2|2018-3-22|修改开启导航系统指令|
|V1.3|2018-7-11|添加动态目标点功能|
|V1.4|2018-11-8|增加自动巡检功能|
|V1.5|2018-11-23|增加地图创建相关状态，增加创建地图控制指令|
|V1.6|2019-1-25|添加自动充电指令|
|V1.7|2019-10-11|添加调度系统导航控制指令|
|V1.8|2020-04-02|增加动态导航载入控制|
|V1.9|2020-08-07|增加无ID目标点任务状态|
