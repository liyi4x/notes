# packages

## 1. package结构

package是catkin编译的基本单元，一个package可以编译出来多个目标文件（ROS可执行程序、动态静态库、头文件等等）

```TEXT
├── CMakeLists.txt    #package的编译规则(必须)
├── package.xml       #package的描述信息(必须)
├── src/              #源代码文件
├── include/          #C++头文件
├── scripts/          #可执行脚本
├── msg/              #自定义消息
├── srv/              #自定义服务
├── action/           #自定义服务
├── models/           #3D模型文件
├── urdf/             #urdf文件
├── launch/           #launch文件
├── config/           #参数配置文件
```

`CMakeLists.txt`文件和`package.xml`文件，这两个文件定义了package

- `CMakeLists.txt`定义的是package的编译规则，用到的依赖关系等
- `package.xml`定义的是功能包相关的信息，描述package的包名、版本号、作者、依赖等信息
- `src/`存放的是源代码文件，主要是cpp源码以及python的module文件
- `include/`存放的是hpp头文件
- `scripts/`存放的是脚本文件，shell脚本、python脚本等
- `msg/`存放的是自定义格式的消息文件（*.msg）
- `srv/`存放的是自定义格式的服务文件（*.srv）
- `action/`存放的是自定义格式的动作文件（*.action）
- `models/`存放机器人或仿真场景的3D模型（.sda, .stl, .dae等）
- `urdf/`存放机器人的模型描述（.urdf, .xacro）
- `launch/`存放的是启动文件（*.launch）
- `config/`存放的是全局配置文件（*.yaml）

## 2. 创建package

创建一个package需要在`catkin_ws/src/`下，用到`catkin_create_pkg`命令，相当于脚手架，搭建package框架结构

```bash
catkin_create_pkg package depends
```

其中`package`是包名，`depends`是依赖的包名，可以依赖多个软件包。

例如，新建一个package叫做`test_pkg`，依赖`roscpp`、`rospy`、`std_msgs`(常用依赖)。

```bash
catkin_create_pkg test_pkg roscpp rospy std_msgs
```

此时目录结构为

```text
├── CMakeLists.txt
├── include
│   └── test_pkg
├── package.xml
└── src
```

`catkin_create_pkg`默认创建好了`CMakeLists.txt`和`package.xml`，并将项目依赖项添加到了`package.xml`

## 3. package相关的命令

### 3.1. rospack

`rospack`是ros的package管理工具

| rostopic命令 | 作用 |
| :---: | :---: |
| `rospack help` | 显示rospack的用法 |
| `rospack list` | 列出本机所有package |
| `rospack depends [package]` | 显示package的依赖包 |
| `rospack find [package]` | 定位某个package |
| `rospack profile` | 刷新所有package的位置记录 |

package缺省则默认为当前目录下的package

### 3.2. roscd

`roscd`命令类似与Linux系统的`cd`，改进之处在于`roscd`可以**直接**改变目录到ROS的软件包目录下

### 3.3. rosls

`rosls`也可以视为Linux指令`ls`的改进版，可以直接在**任意目录**下列出相应ROS软件包的内容

`rosls [pacakge]`列出pacakge下的文件

### 3.4. rosdep

`rosdep`是用于管理ROS package依赖项的命令行工具

| rosdep命令 | 作用 |
| :---: | :---: |
| `rosdep check [pacakge]` | 检查package的依赖是否满足 |
| `rosdep install [pacakge]` | 安装pacakge的依赖 |
| `rosdep db` | 生成和显示依赖数据库 |
| `rosdep init` | 初始化/etc/ros/rosdep中的源 |
| `rosdep keys` | 检查package的依赖是否满足 |
| `rosdep update` | 更新本地的rosdep数据库 |

一个较常使用的命令是`rosdep install --from-paths src --ignore-src --rosdistro=kinetic -y`，用于安装工作空间中`src`路径下所有package的依赖项（由pacakge.xml文件指定）

## 4. Metapackage元功能包

### 4.1. Metapackage介绍

Metapackage是一个功能包集合，把几个相近的功能模块、软件包放到一起，之前叫Stack。元功能包本身没有实质性的功能，但是依赖于多个软件包，相当于一个集合

ROS里常见的Metapacakge有：

| Metapacakge名称 | 描述 | 链接 |
| :------: | :------: | :------: |
| `navigation` | 导航相关的功能包集 | <https://github.com/ros-planning/navigation> |
| `moveit` | 运动规划相关的（主要是机械臂）功能包集  | <https://github.com/ros-planning/moveit> |
| `image_pipeline` | 图像获取、处理相关的功能包集 | <https://github.com/ros-perception/image_common> |
| `vision_opencv` | ROS与OpenCV交互的功能包集| <https://github.com/ros-perception/vision_opencv> |
| `turtlebot` | Turtlebot机器人相关的功能包集 | <https://github.com/turtlebot/turtlebot> |
| `pr2_robot` | pr2机器人驱动功能包集 | <https://github.com/PR2/pr2_robot> |
| ... | ...|...|

以上列举了一些常见的功能包集，例如navigation、turtlebot，他们都是用于某一方面的功能，以navigation metapackage（官方介绍里仍然沿用stack的叫法）为例，它包括了以下软件包：

| 包名 | 功能 |
| :------: | :------: |
| `navigation` | Metapacakge，依赖以下所有pacakge |
| `amcl` | 定位 |
| `fake_localization` | 定位 |
| `map_server` | 提供地图 |
| `move_base` | 路径规划节点 |
| `nav_core` | 路径规划的接口类|
| `base_local_planner` | 局部规划 |
| `dwa_local_planner`| 局部规划|
| ... | ... |

### 4.2. Metapackage结构

我们以ROS-Academy-for-beginners为例介绍meteapckage的写法，在教学包内，有一个`ros-academy-for-beginners`软件包，该包即为一个metapacakge，其中有且仅有两个文件：`CMakeLists.txt`和`pacakge.xml`。

`CMakeLists.txt`写法如下：

```cmake
cmake_minimum_required(VERSION 2.8.3)
project(ros_academy_for_beginners)
find_package(catkin REQUIRED)
catkin_metapackage()   #声明本软件包是一个metapacakge
```

`pacakge.xml`写法如下：

```xml
<package>
    <name>ros_academy_for_beginners</name>
    <version>17.12.4</version>
    <description>
        --------------------------------------------------------------------------
        A ROS tutorial for beginner level learners. This metapacakge includes some
        demos of topic, service, parameter server, tf, urdf, navigation, SLAM...
        It tries to explain the basic concepts and usages of ROS.
        --------------------------------------------------------------------------
    </description>
    <maintainer email="chaichangkun@163.com">Chai Changkun</maintainer>
    <author>Chai Changkun</author>
    <license>BSD</license>  
    <url>http://http://www.droid.ac.cn</url>

    <buildtool_depend>catkin</buildtool_depend>

    <run_depend>navigation_sim_demo</run_depend>  <!--注意这里的run_depend标签，将其他软件包都设为依赖项-->
    <run_depend>param_demo</run_depend>
    <run_depend>robot_sim_demo</run_depend>
    <run_depend>service_demo</run_depend>
    <run_depend>slam_sim_demo</run_depend>
    <run_depend>tf_demo</run_depend>
    <run_depend>topic_demo</run_depend>

    <export>    <!--这里需要有export和metapacakge标签，注意这种固定写法-->
        <metapackage/>
    </export>
</package>
```

metapacakge中的以上两个文件和普通pacakge不同点是：

- `CMakeLists.txt`:加入了`catkin_metapackage()`宏，指定本软件包为一个metapacakge。
- `package.xml`:`<run_depend>`标签将所有软件包列为依赖项，`<export>`标签中添加`<metapackage>`标签声明。

metapacakge在我们实际开发一个大工程时可能有用
