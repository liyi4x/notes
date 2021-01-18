# ROS通信架构

## 1. Node & Master

![masterandnode.png](./imgs/masterandnode.png)

ROS通信系统是一个由ROS进程组成的点对点网络，如图所示ROS系统分为一个`Master`和多个`Node`节点，`Node`向`Master`注册，`Node`之间的通信则不需要通过`Master`，可以直接进行点对点的通信。ROS的通信方式主要有：

- `topic`话题
- `service`服务
- `action`动作
- `Parameter Service`参数服务器

### 1.1. Node

`Node`是ROS的最小运行单元，在ROS里每个进程都是一个`Node`，每个`package`的可执行文件都是一个`Node`，代表的是一种功能。对于机器人编程而言，需要对机器人各个功能进行分离，因此最终一个机器人系统会由多个`Node`组成

机器人多个`Node`组成的优点

- 提高系统的鲁棒性，能在某些节点出问题的情况下不影响其他节点
- 降低系统的耦合程度，提高系统的可移植性和可调试性

### 1.2. Master

`Master`是ROS的节点管理器，用于管理ROS中的所有`Node`，`Node`在启动之后的第一件事就是需要向`Master`注册，以便能够完成各个节点之间的通信等操作。所以在启动`Node`之前需要保证`Master`的工作正常

当一个节点开始发布一个`topic`，节点将会将`topic`的名字和数据类型信息传递给`Master`，`Master`将会检查是否有其他的节点订阅了这个`topic`，如果有任何的节点订阅了这个`topic`，`Master`将会共享发布者的消息给订阅者节点。

### 1.3. 分布式ROS

对于单机ROS系统而言，ROS的各个`Node`和`Master`可以不用考虑`URI`问题。但是对于分布式系统而言，ROS系统需要知道每个分布式主机对应的`Node`的地址（IP和端口）。分布式ROS系统中的`Master`同样只能有一个，且需要在`Node`之前启动。

- 通过设置`ROS_MASTER_URI`环境变量来完成局域网内主机对`Master`的定位，进而进行接下来的`Node`启动等工作
- ROS的各个主机之间的通信是通过`{主机名}:{端口号}`的形式进行通信的，因此需要修改局域网内的ROS主机的`host`文件来定位

参考文件

- [ros分布式多机通信 - CSDN](https://blog.csdn.net/hehedadaq/article/details/82898307)
- [ROS分布式多机通信 - 知乎](https://zhuanlan.zhihu.com/p/101331694)

## 2. ROS启动流程

### 2.1. 启动roscore

```bash
roscore
```

与此同时启动的还有

- `rosout`
    - `rosout`是一个节点，主要作用就是日志输出，告之用户当前系统的运行状态，系统运行过程中的error和warning，并将log记录在文件中方便后期复盘
- `parameter server`
    - `parameter server`是参数服务器，主要作用是维护系统运行中的全局参数，各个节点都可以读取参数服务器中的数据

### 2.2. 启动Node

`rosrun`启动一个节点

```bash
rosrun pkg_name node_name
```

- `pkg_name`package名字
- `node_name`package下的node名

Node管理相关的命令`rosnode`

| rosnode命令 | 作用 |
| :------: | :------: |
| `rosnode list` | 列出当前运行的node信息 |
| `rosnode info node_name` | 显示出node的详细信息 |
| `rosnode kill node_name` | 结束某个node |
| `rosnode ping` | 测试连接节点 |
| `rosnode machine` | 列出在特定机器或列表机器上运行的节点 |
| `rosnode cleanup` | 清除不可到达节点的注册信息 |
