# 伽利略导航系统SDK使用说明

为了降低用户开发和使用伽利略导航系统的难度，我们提供了伽利略导航系统SDK。目前对以下语言提供了支持

[C++](https://github.com/BluewhaleRobot/GalileoSDK) [![Build Status](https://travis-ci.org/BluewhaleRobot/GalileoSDK.svg)](https://travis-ci.org/BluewhaleRobot/GalileoSDK)

[Python](https://github.com/BluewhaleRobot/pygalileo) [![Build Status](https://travis-ci.org/BluewhaleRobot/pygalileo.svg)](https://travis-ci.org/BluewhaleRobot/pygalileo)

[C#](https://github.com/BluewhaleRobot/SharpGalileo) [![Build Status](https://travis-ci.org/BluewhaleRobot/SharpGalileo.svg)](https://travis-ci.org/BluewhaleRobot/SharpGalileo)

[Java & Android](https://github.com/BluewhaleRobot/javagalileo) [![Build Status](https://travis-ci.org/BluewhaleRobot/javagalileo.svg)](https://travis-ci.org/BluewhaleRobot/javagalileo)

所有语言的SDK都是基于 `C++ SDK` 进行开发。下面对 `C++ SDK` 进行说明。其他语言可以进行参考，SDK的使用方法都是一致的。对于不同语言SDK的详细信息可以参考每个SDK的README文件。

## 快速开始

### 获取当前局域网内所有机器人

```cpp
#include "GalileoSDK.h"

int main(){
    GalileoSDK::GalileoSDK sdk; // 实例化SDK,注意只能实例化一个SDK对象
    while (true)
    {
        auto servers = sdk.GetServersOnline(); // 获取当前局域网内所有机器人
        if(servers.size() == 0){
            std::cout << "No server found" << std::endl;
        }
        for (auto it = servers.begin(); it < servers.end(); it++)
        {
            std::cout << it->getID() << std::endl; // 输出机器人ID
        }
        Sleep(1000);
    }
}

```

### 连接机器人

下面这种连接方式是阻塞的，sdk会等待10000ms连接局域网内任意一个机器人。如果局域网内有多个机器人，程序会返回MULTI_SERVER_FOUND错误。当局域网内有且仅有唯一的一个机器人时，sdk自动连接。
如果10000ms没有成功连接机器人，Connect方法会返回超时TIMEOUT错误。

```cpp
#include "GalileoSDK.h"

int main(){
    GalileoSDK::GalileoSDK sdk; // 实例化SDK
    auto res = sdk.Connect("", true, 10000, NULL, NULL); // 连接局域网内任意一个机器人
    if(res == GalileoSDK::GALILEO_RETURN_CODE::OK)
        std::cout << "Connect Success" << std::endl;
    std::cout << "Connect Failed" << std::endl;
}

```

### 连接机器人，异步方式

下面的连接方式是异步的，可以通过设置回调函数处理机器人连接状态。这个方法是非阻塞的，Connect方法会立即返回。当连接成功或超时时OnConnect回调会被调用。具体的状态信息可以通过回调函数的第一个参数获取

```cpp
#include "GalileoSDK.h"

int main(){
    GalileoSDK::GalileoSDK sdk; // 实例化SDK
    sdk.Connect("", true, 10000,
        // OnConnect回调函数
        [](GalileoSDK::GALILEO_RETURN_CODE res, std::string id) -> void {
            std::cout << "OnConnect Callback: result " << res << std::endl;
            std::cout << "OnConnect Callback: connected to " << id << std::endl;
        },
        // OnDisconnect回调函数
        [](GalileoSDK::GALILEO_RETURN_CODE res, std::string id) -> void {
            std::cout << "OnDisconnect Callback: result " << res << std::endl;
            std::cout << "OnDisconnect Callback: server " << id << std::endl;
    });
    while(true){
        Sleep(10000);
    }
}
```

### 获取机器人当前电压

在连接机器人后，机器人的状态信息可以通过`GetCurrentStatus`方法获取。具体有哪些状态信息可以参照`galileo_serial_server/GalileoStatus.h`文件定义。

```cpp
#include "GalileoSDK.h"
#include "galileo_serial_server/GalileoStatus.h"

int main(){
    GalileoSDK::GalileoSDK sdk; // 实例化SDK
    sdk.Connect("", true, 10000, NULL, NULL); // 连接局域网内任意一个机器人
    galileo_serial_server::GalileoStatus status; // 创建伽利略状态对象
    while (true)
    {
        if (sdk.GetCurrentStatus(&status) == GalileoSDK::OK) // 获取当前系统状态
        {
            std::cout << status.power << std::endl; // 输出系统电压
        }
        else
        {
            std::cout << "Get status failed" << std::endl;
        }
        Sleep(1000);
    }
}
```

### 控制机器人前进

```cpp
#include "GalileoSDK.h"

int main(){
    GalileoSDK::GalileoSDK sdk; // 实例化SDK
    sdk.Connect("", true, 10000, NULL, NULL); // 连接局域网内任意一个机器人
    sdk.SetSpeed(0.1, 0); // 设置机器人速度，以0.1m/s速度前进
}

```

### 控制机器人移动到特定坐标

假设机器人已经建立好地图且绘制好相关路径，我们可以通过MoveTo方法控制机器人移动到特定坐标位置。下面代码中的posx和posy即为目标点坐标。你可以替换成自己的实际目标点坐标。

```cpp
#include "GalileoSDK.h"
#include "galileo_serial_server/GalileoStatus.h"

int main(){
    GalileoSDK::GalileoSDK sdk; // 实例化SDK
    sdk.Connect("", true, 10000, NULL, NULL); // 连接局域网内任意一个机器人
    sdk.StartNav();
    galileo_serial_server::GalileoStatus status;
    sdk.GetCurrentStatus(&status);
    // 等待正常追踪,正常追踪时visualStatus和navStatus都为1
    while (status.visualStatus != 1 || status.navStatus != 1)
    {
        std::cout << "Wait for navigation initialization" << std::endl;
        sdk.GetCurrentStatus(&status);
        Sleep(1000);
    }
    uint8_t goalNum;
    sdk.MoveTo( posx, posy, &goalNum); // 移动到特定目标点
    // 等待移动开始
    sdk.GetCurrentStatus(&status);
    while (status.targetStatus != 1)
    {
        std::cout << "Wait for goal start" << std::endl;
        sdk.GetCurrentStatus(&status);
        Sleep(1000);
    }
    std::cout << "Goal started" << std::endl;
    // 等待移动完成
    while (status.targetStatus != 0 || status.targetNumID != goalNum)
    {
        std::cout << "Wait for goal complete" << std::endl;
        sdk.GetCurrentStatus(&status);
        Sleep(1000);
    }
}
```

## SDK 说明

```cpp
GALILEO_RETURN_CODE
Connect(std::string targetID, bool auto_connect, int timeout,
        void (*OnConnect)(GALILEO_RETURN_CODE, std::string),
        void (*OnDisconnect)(GALILEO_RETURN_CODE, std::string));
```

连接机器人

输入

`std::string targetID` 目标机器人ID

`bool auto_connect` 是否开启自动连接功能。当targetID为空字符串，且auto_connect为true时。机器人会自动连接局域网内的机器人。当局域网内有多台机器人时，此方法会返回MULTI_SERVER_FOUND错误。

`int timeout` 超时时间，单位毫秒。当在此时间内没有成功连接机器人则此方法返回超时错误。

`void (*OnConnect)(GALILEO_RETURN_CODE, std::string)` 连接回调函数。当此值为NULL时，Connect会以同步阻塞方式执行。Connect会等待连接成功直至超时。当此值不是NULL时，Connect会以非阻塞方式执行。当机器人成功连接或连接超时时，SDK会调用此回调函数。

`void (*OnDisconnect)(GALILEO_RETURN_CODE, std::string)` 连接断开回调函数。当此值非NULL,机器人连接断开时SDK会调用此回调函数。

返回

GALILEO_RETURN_CODE

```cpp
std::vector<ServerInfo> GetServersOnline()
```

获取当前局域网内所有机器人信息

输入 无

输出

局域网内所有机器人列表

```cpp
ServerInfo *GetCurrentServer();
```

获取当前连接的机器人信息

输入 无

输出 当前连接的机器人信息

在SDK未连接机器人时，此方法返回NULL。在SDK成功连接机器人后此方法返回机器人信息

```cpp
GALILEO_RETURN_CODE SendCMD(uint8_t[] cmd, int length);
```

发送伽利略指令，具体信息参考伽利略串口说明文档

输入

`uint8_t[] cmd` 伽利略导航指令

`int length` 指令长度

输出

`GALILEO_RETURN_CODE`

```cpp
GALILEO_RETURN_CODE StartNav();
```

开启导航服务。此方法只会发送开始导航指令，并不会等待导航开启完成。只能在机器人未处于导航状态且未处于建图状态下才可以执行此方法。

输入 空
输出 `GALILEO_RETURN_CODE`

```cpp
GALILEO_RETURN_CODE StopNav();
```

关闭导航服务。此方法只会发送关闭导航指令，并不会等待导航关闭完成。只能在机器人处于导航状态下才可以执行此方法。

输入 空
输出 `GALILEO_RETURN_CODE`

```cpp
GALILEO_RETURN_CODE SetGoal(int goalIndex);
```

设置导航目标点。此方法只会发送设置导航点指令，并不会等待目标点完成。只能在机器人处于导航状态下执行此方法。

输入

`int goalIndex` 目标点标号

输出 `GALILEO_RETURN_CODE`

```cpp
GALILEO_RETURN_CODE PauseGoal();
```

暂停当前导航任务

输入 无

输出 GALILEO_RETURN_CODE

```cpp
GALILEO_RETURN_CODE ResumeGoal();
```

继续当前导航任务

输出 无

输出 GALILEO_RETURN_CODE

```cpp
GALILEO_RETURN_CODE CancelGoal();
```

取消当前导航任务

输入 无

输出 GALILEO_RETURN_CODE

```cpp
GALILEO_RETURN_CODE InsertGoal(float x, float y);
```

插入导航点

输入

`float x` 插入点x坐标，此坐标为机器人map坐标系下坐标
`float y` 插入点y坐标，此坐标为机器人map坐标系下坐标

输出 `GALILEO_RETURN_CODE`

```cpp
GALILEO_RETURN_CODE ResetGoal();
```

重置目标点。所有通过InsertGoal插入的目标点都会重置。只保留原始目标点。

输入 无

输出 `GALILEO_RETURN_CODE`

```cpp
GALILEO_RETURN_CODE SetSpeed(float vLinear, float vAngle);
```

设置机器人速度

输入

`float vLinear` 直线运动速度，单位m/s
`float vAngle` 转动速度， 单位 rad/s

输出 `GALILEO_RETURN_CODE`

```cpp
GALILEO_RETURN_CODE Shutdown();
```

关闭机器人主机

输入 无

输出 `GALILEO_RETURN_CODE`

```cpp
GALILEO_RETURN_CODE SetAngle(uint8_t sign, uint8_t angle);
```

设置机器人角度

输入

`uint8_t sign` 转向0表示正向旋转，1表示反向旋转
`uint8_t angle` 转动角度

输出 `GALILEO_RETURN_CODE`

```cpp
GALILEO_RETURN_CODE StartLoop();
```

开始循环导航。开启后机器人会按照导航点的顺序依次执行。

输入 无

输出 `GALILEO_RETURN_CODE`

```cpp
GALILEO_RETURN_CODE StopLoop();
```

停止导航循环

输入 无

输出 `GALILEO_RETURN_CODE`

```cpp
GALILEO_RETURN_CODE SetLoopWaitTime(uint8_t time);
```

设置循环等待时间。机器人在循环过程中到达目标点的等待时间

输入 无

输出 `GALILEO_RETURN_CODE`

```cpp
GALILEO_RETURN_CODE StartMapping();
```

开始建图服务。机器人只能在不处于导航状态或建图状态下才能执行此方法。此方法只是发送开启建图指令，并不会等待建图服务开始运行。

输入 无

输出 `GALILEO_RETURN_CODE`

```cpp
GALILEO_RETURN_CODE StopMapping();
```

停止建图服务。机器人只能在建图状态下才能执行此方法。此方法只是发送停止建图指令，并不会等待建图服务停止。

输入 无

输出 `GALILEO_RETURN_CODE`

```cpp
GALILEO_RETURN_CODE SaveMap();
```

保存当前地图。当前地图会被保存在机器人的 `slamdb` 文件夹，但是在Windows客户端并不会显示。想要在Windows客户端显示需要把当前地图复制到 `saved-slamdb` 文件夹

输入 无

输出 `GALILEO_RETURN_CODE`

```cpp
GALILEO_RETURN_CODE UpdateMap();
```

更新地图

输入 无

输出 `GALILEO_RETURN_CODE`

```cpp
GALILEO_RETURN_CODE StartChargeLocal();
```

开始局部充电功能。局部充电采用惯导定位，利用充电桩传感器对准充电桩。局部充电只能在机器人在充电桩附近使用。此方法只是发送开启局部充电指令，并不会等待充电动作完成

输入 无

输出 `GALILEO_RETURN_CODE`

```cpp
GALILEO_RETURN_CODE StopChargeLocal();
```

停止局部充电功能

输入 无

输出 `GALILEO_RETURN_CODE`

```cpp
GALILEO_RETURN_CODE SaveChargeBasePosition();
```

保存充电桩位置

输入 无

输出 `GALILEO_RETURN_CODE`

```cpp
GALILEO_RETURN_CODE StartCharge(float x, float y);
```

开始充电。此方法采用融合导航定位算法，可以在保证机器人处于导航状态下，在地图任意位置执行此指令。此指令首先会控制机器人移动到充电桩附近,然后再启动局部充电指令。此指令是一个阻塞指令，程序会等待机器人运动到充电桩后再退出。

输入 无

输出 `GALILEO_RETURN_CODE`

```cpp
GALILEO_RETURN_CODE StopCharge();
```

取消充电指令

输入 无

输出 `GALILEO_RETURN_CODE`

```cpp
GALILEO_RETURN_CODE MoveTo(float x, float y, uint8_t *goalNum);
```

控制机器人移动到指定坐标

输入

`float x` 目标点x坐标
`float y` 目标点y坐标
`goalNum` 新插入的目标点id，此参数相当于一个返回值。在成功插入点后此值会被设置为新插入点的ID

输出 `GALILEO_RETURN_CODE`


```cpp
GALILEO_RETURN_CODE GetGoalNum(uint8_t *goalNum);
```

获取当前目标点总数

输入

`uint8_t *goalNum` 相当于返回值，成功调用后此值会被设置为当前目标点总数

输出 `GALILEO_RETURN_CODE`

```cpp
GALILEO_RETURN_CODE GetCurrentStatus(galileo_serial_server::GalileoStatus *);
```

获取当前机器人伽利略状态

输入

`galileo_serial_server::GalileoStatus * status` 相当于返回值，成功调用后此值会被设置为当前伽利略系统状态

输出 `GALILEO_RETURN_CODE`

```cpp
void SetCurrentStatusCallback(void (*callback)(
        GALILEO_RETURN_CODE, galileo_serial_server::GalileoStatus));
```

设置状态更新回调函数，当伽利略系统状态更新时会执行设置的回调函数并将最新的系统状态传递给此回调函数。

```cpp
void SetGoalReachedCallback(
        void (*callback)(int goalID, galileo_serial_server::GalileoStatus));
```

设置到达目标点函数。当机器人到达任意目标点时会执行此处设置的回调函数。并且将目标点ID和当前系统状态传递给此回调函数

```cpp
GALILEO_RETURN_CODE WaitForGoal(int goalID);
```

等待到达目标点。此方法会阻塞直至到达目标点或目标点被取消。

输入

`int goalID` 目标点ID

输出 `GALILEO_RETURN_CODE`

```cpp
bool CheckServerOnline(std::string targetid);
```

检查目标机器人是否在线

输入

`std::string targetid` 目标机器人ID

输出

`bool` 目标机器人是否在线

```cpp
void Dispose();
```

释放SDK占用的资源

## 注意事项

1. 在机器人刚启动时可能SDK会连接失败。机器人需要在启动后等待初始化完成才能成功连接（可能要十秒左右）。连接失败时可以重试连接。