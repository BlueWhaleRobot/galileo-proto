# 伽利略导航系统HTTP协议说明

API的格式为API版本号加上对应的URL，以获取系统状态API为例，实际请求地址为/api/v1/system/status。API返回值都是json格式的数据。下面的文档将省略API前缀。API服务程序默认端口为3546。
机器人在启动后会向局域网发送UDP广播，端口为22002。通过此广播我们能够获取到局域网中的机器人基本信息。下面是一个具体的广播数据例子

{"id": "F072E1BA8162245572D2FAEEB2526C5DD916F5A0D6D0F8A14B67FA43DC501079461280B15BA5", "port": 3546, "mac": "00:e0:4c:68:6f:0f", "version": "5.0.0"}

广播数据包含了机器人ID，机器人Http服务端口号，机器人mac和机器人当前的http服务版本号。

## 跨局域网调用API

通过伽利略网络代理我们可以实现远程跨局域网的机器人API调用

[机器人远程代理设置网址](http://robot.bwbot.org)

## HTTP 服务的参数配置

http服务参数配置文件位于 `/home/xiaoqiang/Documents/ros/src/galileo_api/config.json`

其默认内容如下

```json
{
    "username": "admin",
    "password": "admin",
    "allow_update": true,
    "no_token_check": true,
    "auto_charge": false,
    "battery_low": 0
}
```

|参数|类型|说明|
|--|--|--|
|username|string|获取机器人token时的用户名。默认为admin|
|password|string|获取机器人token时的密码。默认为admin|
|allow_update|bool|是否允许自动更新。当为true时程序有更新时客户端会收到机器人更新提示。反之则不会有更新提示。|
|no_token_check|bool|是否开启token验证功能。默认不开启|
|auto_charge|bool|是否开启低电量自动充电功能，默认不开启。注意开启此功能要保证机器人导航地图中有正确的充电桩位置，同时导航地图能够正常工作|
|battery_low|float|低电量充电时的电压|

## token API

为了系统的安全性，在调用机器人api的时候可以加上token验证的功能。此功能默认关闭，可以通过配置http api参数打开此功能。

URL: /token

请求方式: GET

请求参数:

|参数|类型|说明|
|--|--|--|
|username|string|配置文件中的用户名，默认为admin|
|password|string|配置文件中的密码, 默认为admin|

返回值

|参数|类型|说明|
|result|bool|是否成功获取token|
|token|string|获取的token|

获取token后在调用api时可以在url参数中加入token=xxx,xxx为你的token数据。
对于POST或PUT请求，token参数可以加在url中也可以在body的json数据里面。

下面是一个调用的例子

```text
http://192.168.0.132:3546/api/v1/system/info?token=28b1c500400611ebb805493c9303c705
```

其中192.168.0.132为机器人IP，28b1c500400611ebb805493c9303c705为机器人token。

## 系统状态API

### 获取系统当前状态

URL: /system/status

请求方式: GET

请求参数: 无

返回参数:

|参数|类型|说明|
|--|--|--|
|status|string|状态String,可能的值为 Mapping, Navigating, Busy, Free, Charging|

说明：系统可能处在互斥的几种状态中。通过不同的操作API系统在不同的状态间切换。只有在特定的状态下系统才能执行特定的API。

### 获取系统基本信息

URL: /system/info

请求方式: GET

请求参数: 无

返回参数:

|参数|类型|说明|
|--|--|--|
|battery|int|电池电量百分比|
|camera_rgb|int|rgb摄像头topic发布频率，0代表没有数据|
|camera_depth|int|深度摄像头topic发布频率，0代表没有数据|
|odom|int|编码器topic发布频率，0代表没有数据|
|imu|int|IMU topic的发布频率，0代表没有数据|

***注意第一次调用时，由于需要对数据统计，可能对应的数据返回为0.之后调用返回数据将为正常***

### 获取rgb摄像头原始图像

URL: /system/camera_rgb

请求方式: GET

请求参数: 无

返回数据：返回为当前机器人摄像头视频流

### 获取rgb摄像头处理后图像

URL: /system/camera_rgb_processed

请求方式: GET

请求参数: 无

返回数据: 返回为当前机器人摄像头处理后的视频流

### 获取系统日志

URL: /system/log

请求方式: GET

请求参数: 无

返回参数:

|参数|类型|说明|
|--|--|--|
|log|string|系统日志信息|

### 获取机器人速度

URL: /system/speed

请求方式: GET

请求参数: 无

返回参数:

|参数|类型|说明|
|--|--|--|
|speed_x|float|x方向速度，单位为米每秒|
|speed_y|float|y方向速度，单位为米每秒|
|speed_angle|float|转动角速度, 单位是弧度每秒|

### 遥控机器人

URL: /system/speed

请求方式: PUT

请求参数:

|参数|类型|说明|
|--|--|--|
|speed_x|float|x方向速度|
|speed_y|float|y方向速度|
|speed_angle|float|转动角速度|

返回参数:

|参数|类型|说明|
|--|--|--|
|result|bool|遥控指令是否成功执行|

### 关闭机器人

URL: /system/shutdown

请求方式: GET

请求参数: 无

返回参数:

|参数|类型|说明|
|--|--|--|
|result|bool|是否成功接收关机指令|

### 校正机器人陀螺仪

URL: /system/calibrate_imu

请求方式: GET

请求参数: 无

返回参数:

|参数|类型|说明|
|--|--|--|
|result|bool|是否开始校准陀螺仪|

### 开始校正摄像头角度位置参数

URL: /system/calibrate_camera/start

请求方式: GET

说明：此自动校准方法只对单摄像头系统有效。对于多摄像头情况可能会造成异常结果。执行启动校准方法后需要遥控机器人在环境中运动，使其路径形成一个闭环完成闭环优化。闭环优化完成后，继续移动一段距离，然后调用提交参数方法完成参数校准。

请求参数: 无

返回参数:

|参数|类型|说明|
|--|--|--|
|result|string|是否开始校准陀螺仪|

### 取消摄像头角度位置校正参数

URL: /system/calibrate_camera/cancel

请求方式: GET

请求参数: 无

返回参数:

|参数|类型|说明|
|--|--|--|
|result|string|是否取消校准陀螺仪|

### 获取摄像头角度位置校正状态

URL: /system/calibrate_camera/status

请求方式: GET

请求参数: 无

返回参数:

|参数|类型|说明|
|--|--|--|
|status|string|标定状态，可以是CALIBRATING或CALIBRATED。分别表示标定中和标定完成|
|accuracy|float|在标定完成时候会包含此参数，表明标定准确度，最大值为100|

### 提交摄像头角度位置校正参数

URL: /system/calibrate_camera/complete

请求方式: GET

说明： 只有在处于CALIBRATED状态下才能够提交标定参数

请求参数: 无

返回参数:

|参数|类型|说明|
|--|--|--|
|status|string|标定参数是否提交成功|

### 检查系统更新状态

URL: /system/update/check

请求方式: GET

请求参数: 无

返回参数:

|参数|类型|说明|
|--|--|--|
|status|string|检查更新程序是否成功执行|
|need_update|bool|是否需要更新|

### 开始自动更新程序

URL: /system/update/start

请求方式: GET

请求参数: 无

返回参数:

|参数|类型|说明|
|--|--|--|
|status|string|系统更新程序是否成功启动|

### 取消自动更新程序

URL: /system/update/cancel

请求方式: GET

请求参数: 无

返回参数:

|参数|类型|说明|
|--|--|--|
|status|string|取消系统更新是否成功|

### 获取软件更新日志

URL: /system/update/log

请求方式: GET

请求参数: 无

返回参数:

|参数|类型|说明|
|--|--|--|
|status|string|是否成功获取更新日志|
|log|string|系统更新日志内容|
|is_complete|bool|系统更新是否已经完成|

### 获取当前调度服务器

URL: /system/schedule_manager

请求方式: GET

请求参数: 无

返回参数: 所有schedule manager数据，其中第一个是当前的schedule manager

返回数据示例

```json
[
  {
    "ip": "192.168.0.23",
    "update_time": 1576115449217,
    "version": "1.0.0",
    "id": "9188d590-8508-47bd-a704-d34ddb803265",
    "port": 24958
  }
]
```

### 系统自检

URL: /system/self_test

请求方式: GET

返回参数:

|参数|类型|说明|
|--|--|--|
|motor_driver|bool|底盘驱动状态是否正常|
|camera|bool|摄像头状态是否正常|
|charge|bool|自动充电模块是否正常|
|lidar|bool|雷达是否正常|
|bluetooth|bool|蓝牙是否工作正常|
|battery|bool|获取电池电压是否正常|

### 获取当前io电平情况

注意io仍为电平输出，获取的是输出电平的状态，并不是输入信号

URL: /system/info

请求方式: GET

请求参数: 无

返回参数:

|参数|类型|说明|
|--|--|--|
|status|string|获取状态|
|out1|string|"-1"为未设置，"0"为低电平, "1"为高电平|
|out2|string|"-1"为未设置，"0"为低电平, "1"为高电平|
|out3|string|"-1"为未设置，"0"为低电平, "1"为高电平|

### 控制当前io电平情况

URL: /system/info

请求方式: POST

请求参数:

|参数|类型|说明|
|--|--|--|
|level|string|0 为低电平， 1为高电平|
|port|string|可以为1,2,3分别对应三个IO端口|

返回参数:

|参数|类型|说明|
|--|--|--|
|status|string|获取状态|
|status|string|设置电平状态|

### 获取机器人当前配置参数

URL: /system/config

请求方式: GET

请求参数: 无

返回参数:

|参数|类型|说明|
|--|--|--|
|default_map|string|默认地图名称|
|maps|list|地图对应默认路径配置|
|navigation_speed|float|最大导航速度|
|max_control_speed|float|最大遥控速度|
|bar_distance_min|float|避障距离|
|k2|float|PID控制参数k2|
|kp|float|PID控制参数kp|
|ki|float|PID控制参数ki|
|kd|float|PID控制参数kd|
|look_ahead_dist|float|预估距离|
|theta_max|float|最大角速度|
|plan_width|float|车体宽度|
|path_change|float|避障时是否绕开|
|forward_width|float|预估距离|
|rot_width|float|车体旋转宽度|
|backtime|float|最大后退距离|

### 修改机器人配置

URL: /system/config

请求方式: POST

请求参数:

|参数|类型|说明|
|--|--|--|
|default_map|string|默认地图名称，可选参数|
|maps|list|地图对应默认路径配置，可选参数|
|navigation_speed|float|最大导航速度，可选参数|
|max_control_speed|float|最大遥控速度，可选参数|
|bar_distance_min|float|避障距离，可选参数|
|k2|float|PID控制参数k2，可选参数|
|kp|float|PID控制参数kp，可选参数|
|ki|float|PID控制参数ki，可选参数|
|kd|float|PID控制参数kd，可选参数|
|look_ahead_dist|float|预估距离，可选参数|
|theta_max|float|最大角速度，可选参数|
|plan_width|float|车体宽度，可选参数|
|path_change|float|避障时是否绕开，可选参数|
|forward_width|float|预估距离，可选参数|
|rot_width|float|车体旋转宽度，可选参数|
|backtime|float|最大后退距离，可选参数|

返回参数:

修改后的机器人参数

|参数|类型|说明|
|--|--|--|
|default_map|string|默认地图名称|
|maps|list|地图对应默认路径配置|
|navigation_speed|float|最大导航速度|
|max_control_speed|float|最大遥控速度|
|bar_distance_min|float|避障距离|
|k2|float|PID控制参数k2|
|kp|float|PID控制参数kp|
|ki|float|PID控制参数ki|
|kd|float|PID控制参数kd|
|look_ahead_dist|float|预估距离|
|theta_max|float|最大角速度|
|plan_width|float|车体宽度|
|path_change|float|避障时是否绕开|
|forward_width|float|预估距离|
|rot_width|float|车体旋转宽度|
|backtime|float|最大后退距离|

### 恢复机器人默认参数

URL: /system/config

请求方式: DELETE

请求参数

|参数|类型|说明|
|--|--|--|
|key|string|对应参数的名称,如k2,kp,ki等等|

返回参数:

|参数|类型|说明|
|--|--|--|
|status|string|恢复默认值操作状态|

### 让机器人播放语音

URL: /system/tts

请求方式: GET

请求参数:

|参数|类型|说明|
|--|--|--|
|text|string|需要机器人播放的语音对应的文本|

返回参数:

|参数|类型|说明|
|--|--|--|
|status|string|语音播放状态|

## 创建地图API

### 启动创建地图

URL: /map/start

请求方式: GET

说明: 接受到启动命令后系统会处于Busy状态，线程启动成功后系统进入Mapping状态。

请求参数: 无

返回参数:

|参数|类型|说明|
|--|--|--|
|result|bool|是否成功启动导航|

### 结束创建地图线程

URL: /map/stop

请求方式: GET

说明: 接收到结束命令进入Busy状态。后先自动保存地图，然后开始关闭建图线程，线程关闭后系统进入Free状态。

请求参数: 无

返回参数

|参数|类型|说明|
|--|--|--|
|result|bool|是否成功结束建图|

### 当前机器人位置

URL: /map/pose

请求方式: GET

请求参数: 无

返回参数:

|参数|类型|说明|
|--|--|--|
|x|float|机器人当前x坐标，单位为米|
|y|float|机器人当前y坐标，单位为米|
|angle|float|机器人当前朝向角度， 单位为度|

### 获取当前正在创建的地图信息

URL: /map/current_map_image

请求方式: GET

请求参数: 无

返回参数:

|参数|类型|说明|
|--|--|--|
|status|string|是否成功获取地图信息|
|map_params|object|地图的meta信息|
|map_image|string|获取当前地图图片的URL路径|

返回数据示例:

```json
{
    "status": "OK",
    "map_params": {
        "origin": [
            -7.621091365814209,
            -9.969059944152832,
            0.0
        ],
        "width": 306,
        "resolution": 0.05000000074505806,
        "height": 324
    },
    "map_image": "/api/v1/map/current_map_png"
}
```

### 保存当前创建的地图

URL: /map/current_map_image

请求方式: POST

请求参数:

|参数|类型|说明|
|--|--|--|
|name|string|创建的地图名称|

返回参数:

|参数|类型|说明|
|--|--|--|
|result|bool|地图保存是否成功|
|name|string|保存后的地图名称|

### 获取当前创建地图的png图片

URL: /map/current_map_png

请求方式: GET

请求参数: 无

返回参数: 当前正在创建的地图的png图片

### 获取正在创建地图的图像视频

URL: /map/stream

请求方式: GET

请求参数: 无

返回参数: 正在创建的地图的图片视频

### 获取创建地图时机器人的运动轨迹

URL: /map/track

请求方式: GET

请求参数: 无

返回参数:

|参数|类型|说明|
|--|--|--|
|trajectory|list|创建地图时机器人运动轨迹|

返回数据示例:

```json
{
  "trajectory": [
    {
      "y": 0.3912552,
      "x": 0.3480716,
      "z": -0.0384934
    },
    {
      "y": 0.3972594,
      "x": 0.4466029,
      "z": -0.038688
    },
    {
      "y": 0.403202,
      "x": 0.5998781,
      "z": -0.0389315
    },
    ...
  ]
}
```

### 获取已经保存的地图信息

URL: /map/image

请求方式: GET

请求参数:

|参数|类型|说明|
|--|--|--|
|name|string|地图名称，可选参数。当有此参数时返回目标地图的pgm地图图片。当无此参数时返回所有保存的地图的名称|

返回参数: 如上说明

返回参数示例:

当没有name参数时

```json
[
  "走廊",
  "update3",
  "map1"
]
```

### 重命名已经保存的地图

URL: /map/image

请求方式: PUT

请求参数:

|参数|类型|说明|
|--|--|--|
|name|string|目标地图的名称|
|new_name|string|新地图名称|

返回参数

|参数|类型|说明|
|--|--|--|
|result|bool|是否成功重命名地图|
|name|string|新的地图名称|

## 导航部分

系统处于Navigating状态下才可能执行下面的API

### 启动导航状态

URL: /navigation/start

请求方式: GET

说明： 接收到启动命令后系统会处于Busy状态，线程启动成功后系统进入Navigating状态。启动命令参数中包含导航地图文件名字。

请求参数:

|参数|类型|说明|
|--|--|--|
|map|string|导航所使用的地图名称|
|path|string|导航所使用的路径名称|

返回参数

|参数|类型|说明|
|--|--|--|
|result|bool|是否成功启动导航|
|description|string|若启动失败，其原因说明|

### 启动调度导航

URL: /navigation/start_schedule_mode

请求方式: GET

说明: 接收到启动命令后系统会处于Busy状态，线程启动成功后系统进入Navigating状态。此导航状态不同于普通导航状态。伽利略导航功能将不能使用，机器人将由拉格朗日调度系统控制。

请求参数: 无

返回参数:

|参数|类型|说明|
|--|--|--|
|result|bool|是否成功启动调度导航|
|description|string|若启动失败，其原因说明|

### 停止导航

URL: /navigation/stop

请求方式: GET

请求参数: 无

返回参数:

|参数|类型|说明|
|--|--|--|
|result|bool|是否成功启动调度导航|
|description|string|若启动失败，其原因说明|

### 停止调度导航

URL: /navigation/stop_schedule_mode

请求方式: GET

请求参数: 无

返回参数:

|参数|类型|说明|
|--|--|--|
|result|bool|是否成功启动调度导航|
|description|string|若启动失败，其原因说明|

### 机器人当前位置信息

URL: /navigation/pose

请求方式: GET

请求参数: 无

返回参数:

|参数|类型|说明|
|--|--|--|
|x|float|机器人x坐标，单位为米|
|y|float|机器人y坐标，单位为米|
|angle|float|机器人朝向角度，单位为度|

### 获取当前导航正在使用的地图和路径名称

URL: /navigation/current_path

请求方式: GET

请求参数: 无

返回参数:

|参数|类型|说明|
|--|--|--|
|map|string|当前导航使用的地图名称|
|path|string|当前导航使用的路径名称|

### 获取当前位置到目标点的距离

URL: /navigation/target_distance

请求方式: GET

请求参数: 无

返回参数:

|参数|类型|说明|
|--|--|--|
|distance|float|当前位置到目标点的距离|

### 获取已经保存的详细地图信息

URL: /navigation/saved_maps

请求参数: GET

请求参数:

|参数|类型|说明|
|--|--|--|
|name|string|目标地图的名称，可选参数。当没有此参数时返回所有已保存的地图信息。当有此参数时，返回目标地图的信息|

返回参数:

|参数|类型|说明|
|--|--|--|
|resolution|float|地图分辨率，即每像素点对应的实际距离|
|origin|object|地图原点坐标|
|occupied_thresh|float|占用阈值，一般没什么用|
|free_thresh|float|未占用阈值，一般没什么用|
|negate|int|是否反转占用和未占用，一般没什么用|
|name|string|目标地图名称|
|md5sum|string|目标地图数据的md5sum值，用于判断地图是否是一样的|

### 获取保存的地图中的机器人运动轨迹

URL: /navigation/robot_tracks

请求方式: GET

请求参数:

|参数|类型|说明|
|--|--|--|
|map_name|string|目标路径的地图名称。可选参数，没有此参数时返回所有地图中的机器人路径数据。当有此参数时只返回目标地图的机器人路径数据|

返回参数:

当请求参数有map_name时

|参数|类型|说明|
|--|--|--|
|trajectory|list|机器人运动轨迹点列表|

当请求参数有map_name时

返回所有地图的轨迹点列表，包含地图名称和轨迹点数据

返回数据示例:

当没有map_name参数时

```json
[
  {
    "map_name": "map1",
    "track": [
      {
        "y": 0.0000016,
        "x": -0.0010667,
        "z": -0.0000129
      },
      {
        "y": 0.0084065,
        "x": 0.0657083,
        "z": -0.0001012
      },
      {
        "y": -0.0036619,
        "x": 0.1893717,
        "z": -0.0003028
      },
      ...
    ]
  }
]
```

### 获取目标地图的PGM图片信息

URL: /navigation/map_pgm

请求方式: GET

请求参数:

|参数|类型|说明|
|--|--|--|
|name|string|目标地图名称|

返回目标地图的pgm图片数据

### 获取目标地图的png图片信息

URL: /navigation/map_png

请求方式: GET

|参数|类型|说明|
|--|--|--|
|name|string|目标地图名称|

返回目标地图的png图片数据

### 获取当前正在使用的地图

URL: /navigation/current_map

请求方式: GET

请求参数: 无

返回参数:

|参数|类型|说明|
|--|--|--|
|name|string|目标地图名称|
|md5sum|string|目标地图数据的md5sum值|

### 上传当前机器人地图数据至调度服务器

URL: /navigation/upload_map

请求方式: GET

说明: 此方法是异步方法，调用后会在系统中创建一个上传地图数据的线程。上传进度可以通过task相关api获取。使用前一定要保证调度服务器和机器人可以正常通信，且机器人处于被激活状态。

请求参数:

|参数|类型|说明|
|--|--|--|
|server_id|string|调度服务器id|

返回参数:

地图上传任务数据

示例返回数据:

```json
{
    "name": "upload_map",
    "loop_flag": false,
    "id": "8bc1e6dd-df1f-4a78-a17b-c60911ce5a06",
    "state": "WORKING",
    "sub_tasks": [
        {
            "server_id": "9188d590-8508-47bd-a704-d34ddb803265",
            "state": "WORKING",
            "result": "",
            "progress": 0,
            "type": "upload_map_action",
            "id": "e21c743d-c692-4ed2-b2be-1470da60ba1e"
        }
    ],
    "progress": 0.0,
    "current_task": {
        "server_id": "9188d590-8508-47bd-a704-d34ddb803265",
        "state": "WORKING",
        "result": "",
        "progress": 0,
        "type": "upload_map_action",
        "id": "e21c743d-c692-4ed2-b2be-1470da60ba1e"
    }
}
```

### 从调度服务器下载地图

URL: /navigation/download_map

请求方式: GET

请求参数:

|参数|类型|说明|
|--|--|--|
|server_id|string|调度服务器id|
|map_id|string|需要下载的地图id|

返回参数:

地图下载任务数据

返回数据示例:

```json
{
    "name": "download_map",
    "loop_flag": false,
    "id": "fb7669d2-5f03-4461-a992-95422f66d49d",
    "state": "WORKING",
    "sub_tasks": [
        {
            "server_id": "9188d590-8508-47bd-a704-d34ddb803265",
            "map_id": "04f777b8-ff9d-4303-87ac-334dab2e0ffe",
            "state": "WORKING",
            "result": "",
            "progress": 0,
            "type": "download_map_action",
            "id": "2fb7eebc-8048-47d8-b62a-60365c772f87"
        }
    ],
    "progress": 0.0,
    "current_task": {
        "server_id": "9188d590-8508-47bd-a704-d34ddb803265",
        "map_id": "04f777b8-ff9d-4303-87ac-334dab2e0ffe",
        "state": "WORKING",
        "result": "",
        "progress": 0,
        "type": "download_map_action",
        "id": "2fb7eebc-8048-47d8-b62a-60365c772f87"
    }
}
```

### 开启导航任务

URL: /navigation/start_nav_task

请求方式: POST

请求参数:

|参数|类型|说明|
|--|--|--|
|x|float|导航目标点坐标x值，单位为米|
|y|float|导航目标点坐标y值，单位为米|
|theta|float|导航目标点角度值,单位为弧度|

返回参数:

导航任务数据

示例返回数据:

```json
{
  "name": "navigation task",
  "loop_flag": false,
  "id": "a6858132-5e84-4d37-b968-19808dd8fd96",
  "state": "WORKING",
  "sub_tasks": [
    {
      "state": "WAITTING",
      "result": "",
      "progress": 0,
      "theta": 1.0936689376831055,
      "y": -0.1287004053592682,
      "x": -0.28466343879699707,
      "current_location": {
        "y": -1,
        "x": -1,
        "theta": -1
      },
      "type": "nav_action",
      "id": "99bfb264-8f8f-4950-940b-6f6b883e7358"
    }
  ],
  "progress": 0,
  "current_task": {
    "state": "WAITTING",
    "result": "",
    "progress": 0,
    "theta": 1.0936689376831055,
    "y": -0.1287004053592682,
    "x": -0.28466343879699707,
    "current_location": {
      "y": -1,
      "x": -1,
      "theta": -1
    },
    "type": "nav_action",
    "id": "99bfb264-8f8f-4950-940b-6f6b883e7358"
  }
}
```

### 开始自动充电

URL: /navigation/go_charge

请求方式: GET

请求参数: 无

返回参数:

返回充电任务对象

### 停止自动充电

URL: /navigation/stop_charge

请求方式: GET

请求参数: 无

返回参数:

|参数|类型|说明|
|--|--|--|
|status|string|停止充电操作状态|
|task|object|充电任务对象|

### 移动到对应目标点

URL: /navigation/move_to_index

请求方式: GET

请求参数:

|参数|类型|说明|
|--|--|--|
|index|int|目标点index，即绘制路径时插入导航点的index|

返回参数:

返回导航任务对象

### 取消当前导航任务

URL: /navigation/stop_nav_task

请求方式: GET

请求参数: 无

返回参数:

返回当前导航任务对象

### 获取导航循环任务信息

导航循环任务为沿着插入的导航点依次移动的导航任务，其效果和点击客户端上的循环选项一样。

URL: /navigation/loop_task

请求方式: GET

请求参数: 无

返回参数:

返回循环导航任务状态

### 开始导航循环任务

URL: /navigation/loop_task

请求方式: POST

请求参数

|参数|类型|说明|
|--|--|--|
|wait_time|int|循环到达目标点后的等待时间|

返回参数:

返回新创建的导航循环任务，如果已有导航循环任务则会返回错误

### 停止导航循环任务

URL: /navigation/loop_task

请求方式: DELETE

请求参数: 无

返回参数:

返回当前的导航循环任务

### 获取充电桩位置

URL: /navigation/charge_pose

请求方式: GET

请求参数: 无

返回参数:

|参数|类型|说明|
|--|--|--|
|x|float|充电桩位置坐标x|
|y|float|充电桩位置坐标y|
|theta|float|充电桩角度theta|

## 任务相关API

机器人的一个动作被定义为Action。比如移动到[1,1]点。又如播放一段声音。一系列的Action组合在一起构成一个任务即Task。通过对Action和Task进行操作，我们可以轻松的控制机器人实现一系列动作。

### 获取任务信息

URL: /task

请求方式: GET

请求参数:

|参数|类型|说明|
|--|--|--|
|id|string|目标任务id，可选参数。若无此参数则返回所有数据库中保存的task|

返回参数:

task 数据信息

### 创建任务

URL: /task

请求方式: POST

请求参数:

|参数|类型|说明|
|--|--|--|
|name|string|创建任务的名称，可以是任意字符串|
|sub_tasks|list|包含子任务信息的列表，子任务可以是task也可以是action|
|loop_flag|bool|可选参数，是否自动循环任务|

例子:

```json
{
    "name": "test task",
    "sub_tasks": [
        {"id": "xxxxxx"},
        {"id": "xxxxxx"},
    ]
}
```

返回参数:

成功创建的task数据

示例返回数据

```json
{
  "name": "test task",
  "loop_flag": false,
  "id": "47a97e4e-9327-4214-a780-fd5a2ae39ee3",
  "state": "WAITTING",
  "sub_tasks": [],
  "progress": 0,
  "current_task": null
}
```

### 修改任务

URL: /task

请求方式: PUT

请求参数:

|参数|类型|说明|
|--|--|--|
|id|string|目标任务id|
|name|string|可选参数，新的任务名称|
|loop_flag|bool|可选参数，是否循环任务|
|sub_tasks|list|包含子任务信息的列表|

返回参数:

修改后的任务数据

示例返回数据

```json
{
  "name": "test task",
  "loop_flag": false,
  "id": "47a97e4e-9327-4214-a780-fd5a2ae39ee3",
  "state": "WAITTING",
  "sub_tasks": [],
  "progress": 0,
  "current_task": null
}
```

### 删除任务

URL: /task

请求方式: DELETE

请求参数:

|参数|类型|说明|
|--|--|--|
|id|string|目标删除任务的id|

返回参数:

|参数|类型|说明|
|--|--|--|
|status|string|目标任务是否删除成功|

### 启动任务

URL: /task/start

请求方式: GET

请求参数:

|参数|类型|说明|
|--|--|--|
|id|string|目标任务的id|

返回参数:

启动后的任务信息

### 暂停任务

URL: /task/pause

请求方式: GET

请求参数:

|参数|类型|说明|
|--|--|--|
|id|string|目标任务的id|

返回参数:

暂停后的任务信息

### 继续任务

URL: /task/resume

请求方式: GET

请求参数:

|参数|类型|说明|
|--|--|--|
|id|string|目标任务的id|

返回参数:

继续后的任务信息

### 取消任务

URL: /task/stop

请求方式: GET

请求参数:

|参数|类型|说明|
|--|--|--|
|id|string|目标任务的id|

返回参数:

取消后的任务信息

### 循环执行任务

URL: /task/loop

请求方式: GET

请求参数:

|参数|类型|说明|
|--|--|--|
|id|string|目标任务的id|
|loop_flag|bool|是否循环任务|

### 获取动作Action信息

URL: /action

请求方式: GET

请求参数:

|参数|类型|说明|
|--|--|--|
|id|string|目标action的id。可选参数，当没有此参数时返回数据库中所有保存的action|

返回参数:

目标action数据信息

### 创建动作Action信息

URL: /action

请求方式: POST

说明:

目前Action有以下几个类别。

#### callback_action 回调动作

当执行此动作时机器人将会向指定的地址(url),以指定的方式(method)，发送指定的数据(data)

|属性|类型|说明|
|--|--|--|
|url|string|回调动作地址|
|method|string|回调请求方法|
|data|object|需要回调发送到的数据|

#### sleep_action 等待动作

当执行此动作时机器人将等待对应的时间

|属性|类型|说明|
|--|--|--|
|wait_time|float|等待的时间，单位秒|

#### upload_map_action 上传地图至指定调度服务器动作

执行此任务时机器人将根据调用参数将地图数据上传至指定的调度服务器

|属性|类型|说明|
|--|--|--|
|server_id|string|调度服务器id|

#### download_map_action 从调度服务器下载地图动作

执行此动作时机器人将根据参数从指定的调度服务器下载地图数据

|属性|类型|说明|
|--|--|--|
|server_id|string|调度服务器id|
|map_id|string|下载的地图id|

#### nav_action 导航动作

执行此动作时机器人将移动到参数指定位置

|属性|类型|说明|
|--|--|--|
|x|float|目标位置x坐标，单位为米|
|y|float|目标位置y坐标，单位为米|
|theta|float|目标位置机器人朝向角度，单位为弧度|

#### charge_action 充电动作

在充电桩附近执行此动作机器人将自动对接充电桩并进行充电

|属性|类型|说明|
|--|--|--|
|x|float|充电桩位置x坐标，单位为米|
|y|float|充电桩位置y坐标，单位为米|
|theta|float|充电正面方向，单位为弧度|

#### local_move_action 局部运动动作

局部运动动作用于机器人精准对接过程。比如控制机器人倒车进入车库等等。

|属性|类型|说明|
|--|--|--|
|distance|float|机器人局部运动距离，当distance为正时向前运动，当distance为负时向后运动，单位为米|
|angle|float|机器人局部运动转向角度，机器人先转动对应角度再直行，单位为弧度|
|method|int|精准对接辅助手段，0为无，1为使用雷达|

请求参数:

|参数|类型|说明|
|--|--|--|
|type|string|action类型，必须是以上几种aciton的一个|
|task_id|string|创建的action所属的task,可选参数。当有此参数时将新建action加入对应task，没有此参数时自动创建一个新的临时task(task不会被保存至数据库)并将action添加进task|

其他参数和对应的Action类型相关

示例请求参数:

```json
{
    "type": "sleep_action",
    "wait_time": 2
}
```

返回参数:

包含新创建的action的task

示例返回参数:

```json
{
  "name": "auto task",
  "loop_flag": false,
  "id": "c50d355f-9155-4a17-ade6-a61b40820b43",
  "state": "WAITTING",
  "sub_tasks": [
    {
      "wait_time": 2,
      "state": "WAITTING",
      "type": "sleep_action",
      "id": "3bbbed8f-c0dc-4a94-8d6f-05006c943818"
    }
  ],
  "progress": 0,
  "current_task": null
}
```

### 修改动作Action信息

URL: /action

请求方式: PUT

请求参数:

|参数|类型|说明|
|--|--|--|
|id|string|目标action的id|

其他参数和Action类型相关

返回参数:

经过修改的Action数据

### 删除动作Action信息

URL: /action

请求方式: DELETE

请求参数:

|参数|类型|说明|
|--|--|--|
|id|string|目标action的id|

### 触发等待动作

说明: 对于`wait_req_action`，机器人会一直等待http请求触发，直到触发后才继续执行下面的任务。

URL: /action/update_wait_req

请求方式: GET

请求参数: 无

返回参数:

当前的`wait_req_action`对象

## Websocket相关API

对于需要高频率获取的数据我们提供了websocket api。比如我们可能需要很高频率的获取机器人的位置，速度等信息。

websocket默认端口3547

URL格式: ws://192.168.0.132:3547/topicName?token=28b1c500400611ebb805493c9303c705

其中192.168.0.132为机器人IP,28b1c500400611ebb805493c9303c705为机器人token。在开启token验证时此参数是必须的，反之则不需要。

topicName为数据对应的ros话题名称。理论上可以订阅机器人内部所有的ros话题。返回的数据为ros话题数据转换成的json数据

`
注意在开启websocket连接后要定期发送ping消息，否则连接可能会自动断开。比如可以每秒发送一个ping消息。
`

下面是常用的接口

### 获取GalileoStatus

GalileoStatus数据定义可以参照串口api中GalileoStatus的说明。

topicName: /galileo/status

返回数据为json

### 获取温湿度及可燃气体数据

TopicName: /bw_env_sensors/EnvSensorData

返回数据

```c
float temperature; // 温度，单位摄氏度
float rh; // 相对湿度 %RH
float smoke; // 烟雾 ppm
float pm1_0; // pm1.0 ug/m^3
float pm2_5; // pm2.5 ug/m^3
float pm10; // pm10 ug/m^3
float lel; // 可燃气体 ppm
float noise; // 噪声 db
```
