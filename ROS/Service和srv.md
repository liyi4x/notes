# Service和srv

## 1. Service

servivce是一种 **请求-查询** 的通信模型，适用于那些临时、非周期性的数据需求的节点使用。双向通信，不仅可以发送消息，而且还可以接受反馈。使用service通信，消息的提供者不需要一直向外发送数据，仅在请求方有消息请求的时候才发送数据。

service的通信包括两部分

- 请求方（Client）
- 服务提供方（Server）

service通信是一种同步通信，请求方向服务方发送一个request，请求方会等待服务方的reply，收到reply之后再进行接下来的工作；服务方在接收到request之后，进行相应的消息处理，并返回一个reply

![service_structure.png](./imgs/service_structure.png)

### 1.1. 和topic通信方式对比

| | topic | service |
| :---: | :---: | :---: |
| 通信方式 | 异步通信 | 同步通信 |
| 通信方向 | 单向通信 | 双向通信 |
| 实现原理 | TCP/IP | TCP/IP |
| 通信模型 | Publish-Subscribe | Request-Reply |
| 映射关系 | Publish-Subscribe（多对多） | Request-Reply（多对一） |
| 特点 | 接受者收到数据会回调（Callback） | 远程过程调用（RPC）服务器端的服务 |
| 应用场景 | 连续、高频的数据发布 | 偶尔使用的功能/具体的任务 |
| 举例 | 激光雷达、里程计发布数据 | 开关传感器、拍照、逆解计算 |

> 远程过程调用(Remote Procedure Call，RPC),可以简单通俗的理解为在一个进程里调用另一个进程的函数。

### 1.2. rosservice命令

| rosservice 命令 | 作用 |
| :---: | :---: |
| `rosservice list` | 显示服务列表 |
| `rosservice info` | 打印服务信息 |
| `rosservice call` | 使用所提供的args调用服务 |
| `rosservice type` | 打印服务类型 |
| `rosservice uri` | 打印服务ROSRPC uri |
| `rosservice find` | 按服务类型查找服务 |
| `rosservice args` | 打印服务参数 |

### 1.3. 通信示例

1. 打开模拟场景`roslaunch robot_sim_demo robot_spawn.launch`。
2. 输入`rosservice list`，可以查看当前运行的服务。
3. 例如`/gazebo/delete_light`服务，是删除光源的操作。
4. 输入`rosservice info /gazebo/delete_light`查看属性信息。可以看到信息，`Node：/gazebo，Type：gazebo_msgs/DeleteLight, Args：Light_name`。这里的类型type也就是,传递参数`Light_name`
5. 输入`rosservice call /gazebo/delete_light sun`，这里的`sun`是参数名，使我们模拟场景中的唯一光源太阳。操作完成后可以看到场景中的光线消失。
6. 可以看到终端的回传信息：`success: True和sun successfully deleted` 这就是双向通信的信息反馈，通知操作已经成功完成。

## 2. srv

类似msg文件，srv文件是用来描述服务（service）数据类型的，service通信的数据格式定义在`*.srv`中。它声明了一个服务的通信格式，包括请求(request)和响应（reply）两部分，中间用`---`隔开，用到的数据格式由`*.msg`文件定义。最终编译时需要修改`package.xml`和`CMakeLists.txt`文件

### 2.1. srv示例

以`msgs_demo/srv/DetectHuman.srv`服务为例，`DetectHuman.srv`服务取自OpenNI的人体检测ROS软件包，是用来查询当前深度摄像头中的人体姿态和关节数的。

```text
bool start_detect
---
my_pkg/HumanPose[] pose_data
```

- `---`上面的内容是请求方需要发送的数据，即请求的格式
    - `bool`格式的数据，是否开始检测
- `---`下面的内容是服务提供方返回的数据，应答数据
    - `my_pkg/HumanPose[]`可变长度数组，`pose_data`的每个元素都是该人的姿态，数据格式定义在`*.msg`文件中

### 2.2. rossrv命令

| rossrv 命令 | 作用 |
| :------: | :------: |
| `rossrv show` | 显示服务描述 |
| `rossrv list` | 列出所有服务 |
| `rossrv md5` | 显示服务md5sum |
| `rossrv package` | 列出包中的服务 |
| `rossrv packages` | 列出包含服务的包 |
