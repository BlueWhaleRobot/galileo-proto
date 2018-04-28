# <a href="#" id="start"></a>伽利略视觉导航系统ROS通信协议

除了使用windows客户端或者串口来控制、使用伽利略视觉导航系统外，还可以通过ros的topic机制来实现。

在[伽利略视觉导航系统串口通信协议](/serial.html)里面，导航串口可以发送和接收串口数据，这是通过galileo_serial_server包实现的，而这个包的实现原理就是将导航串口接收的数据转换成ros话题后以“/galileo/cmds”话题发布给伽利略系统，同时通过订阅伽利略系统状态话题“/galileo/status”然后封装后下发给导航串口。

因此在ros系统里面，我们可以绕开串口直接发布“/galileo/cmds”话题就可以控制伽
利略系统，直接订阅“/galileo/status”就可以获取伽利略系统状态。

## <a href="#" id="pub"></a>1.命令发布话题 “/galileo/cmds”

伽利略视觉导航系统会实时订阅这个话题，给这个话题发送数据就可以对应控制伽利略系统的状态。

这个话题所属类型为galileo_serial_server::GalileoNativeCmds，具体定义在/home/xiaoqiang/Documents/ros/src/galileo_serial_server/msg/GalileoNativeCmds.msg文件。

话题内容有三个成员：header、length、data。这个话题是为了封装串口数据设计的，在《伽利略视觉导航系统串口通信协议》里面，串口指令是由“包头+数据长度+数据内容”构成的。

因此根据串口通信协议，把包头用标准的std_msgs/Header替换成header，数据长度赋
值给length，数据内容赋值给data，就可以构造出有效的/galileo/cmds话题。

例如，下面cpp代码将构造一个开启导航系统的话题数据，数据内容请参考串口通信协议。

```cpp
#include"galileo_serial_server/GalileoNativeCmds.h"
galileo_serial_server::GalileoNativeCmdscurrentCmds;
currentCmds.header.stamp=ros::Time::now();
currentCmds.header.frame_id="galileo_serial_server";
currentCmds.length=0x02;
currentCmds.data.resize(0x02);
currentCmds.data[0]=0x6d;
currentCmds.data[1]=0x00;
```

## <a href="#" id="status"></a>2.伽利略视觉导航系统状态话题“/galileo/status”
伽利略视觉导航系统会持续自动发布/galileo/status话题，订阅这个话题查询系统状态。

这个话题所属类型为galileo_serial_server::GalileoStatus，具体定义在/home/xiaoqiang/Documents/ros/src/galileo_serial_server/msg/GalileoStatus.msg文件。

本话题里面数据成员的意义请参考《伽利略视觉导航系统串口通信协议》中第一部分“导航串口下发的数据包”中的内容定义。

使用“rostopicecho/galileo/status”可以直接打印输出话题内容。
