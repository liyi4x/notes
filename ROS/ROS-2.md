# 二进制包和源码包

## 1. 区别

二进制包是可以直接通过使用`sudo apt-get install`命令进行安装的，能够直接使用而无需重新编译。
源代码包是程序的源代码，能够根据具体情况进行修改，再经过计算机编译，生成二进制可执行文件才能运行。

|区别|二进制包|源代码包|
|-|-|-|
|下载方式|`sudo apt-get install`|`git clone`|
|ROS包存放位置|`/opt/ros/kinetic/`|随意存放，一般为工作目录|
|应用场景|基础软件|第三方程序，需要修改的源码|

## 2. ROS二进制包的安装

例如安装`GMapping`包

```bash
sudo apt-get install ros-kinetic-slam-gmapping
```

所有apt官方的ROS功能包命名均为`ros-版本代号-功能包名`, `kinect`为Ubuntu16.04下的ROS的名称

## 3. 源码安装

以[`ROS-Academy-for-Beginners`](https://github.com/DroidAITech/ROS-Academy-for-Beginners)为例进行安装

### 3.1. 创建工作空间

```bash
mkdir -p ~/tutorial_ws/src
cd tutorial_ws/src
```

### 3.2. 下载源码

```bash
git clone https://github.com/DroidAITech/ROS-Academy-for-Beginners.git
```

### 3.3. 安装依赖

在ros工作空间内安装全部包的依赖

```bash
cd ~/tutorial_ws
rosdep install --from-paths src --ignore-src --rosdistro=kinetic -y
```

另外需要`Gazebo`的版本高于 7.0，可通过`gazebo -v`查看当前版本，升级步骤如下

```bash
sudo sh -c 'echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable `lsb_release -cs` main" > /etc/apt/sources.list.d/gazebo-stable.list'
wget http://packages.osrfoundation.org/gazebo.key -O - | sudo apt-key add -
sudo apt-get update
sudo apt-get install gazebo7
```

### 3.4. 编译

```bash
cd ~/tutorial_ws
catkin_make
source ~/tutorial_ws/devel/setup.bash
```

- `source`命令在每一次打开新终端的时候都要执行一次，故可以写入`~/.bashrc`
- `catkin_make`是一个编译构建系统

### 3.5. 运行仿真程序

```bash
roslaunch robot_sim_demo robot_spawn.launch
```
