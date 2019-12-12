# 伽利略导航系统HTTP协议说明

API的格式为API版本号加上对应的URL，以获取系统状态API为例，实际请求地址为/api/v1/system/status。API返回值都是json格式的数据。下面的文档将省略API前缀。API服务程序默认端口为3546

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
|speed_x|float|x方向速度|
|speed_y|float|y方向速度|
|speed_angle|float|转动角速度|

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
|x|float|机器人当前x坐标|
|y|float|机器人当前y坐标|
|angle|float|机器人当前朝向角度|

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
|x|float|机器人x坐标|
|y|float|机器人y坐标|
|angle|float|机器人朝向角度|

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
|x|float|导航目标点坐标x值|
|y|float|导航目标点坐标y值|
|theta|float|导航目标点角度值|

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
|x|float|目标位置x坐标|
|y|float|目标位置y坐标|
|theta|float|目标位置机器人朝向角度|

#### charge_action 充电动作

在充电桩附近执行此动作机器人将自动对接充电桩并进行充电

|属性|类型|说明|
|--|--|--|
|x|float|充电桩位置x坐标|
|y|float|充电桩位置y坐标|
|theta|float|充电正面方向|

#### local_move_action 局部运动动作

局部运动动作用于机器人精准对接过程。比如控制机器人倒车进入车库等等。

|属性|类型|说明|
|--|--|--|
|distance|float|机器人局部运动距离，当distance为正时向前运动，当distance为负时向后运动|
|angle|float|机器人局部运动转向角度，机器人先转动对应角度再直行|
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
