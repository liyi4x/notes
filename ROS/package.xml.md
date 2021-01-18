# package.xml

## 1. package.xml作用

`package.xml`也是一个catkin的package必备文件，它是这个软件包的描述文件，用于描述pacakge的基本信息。`pacakge.xml`包含了package的名称、版本号、内容描述、维护人员、软件许可、编译构建工具、编译依赖、运行依赖等信息。

实际上`rospack find`、`rosdep`等命令之所以能快速定位和分析出package的依赖项信息，就是直接读取了每一个pacakge中的`package.xml`文件。它为用户提供了快速了解一个pacakge的渠道。

## 2. package.xml写法

目前Indigo、Kinetic、Lunar等版本的ROS都同时支持两种版本的`package.xml`，所以无论选format1还是format2都可以。

参考[解析 package.xml 文件](https://dlonng.com/posts/ros-package)

### 2.1. format2

在新版本（format2）中，包含的标签为：

 | 标签名 | 作用 |
 | :---: | :---: |
 | `<pacakge>` | 根标记文件 |
 | `<name>` | 包名 |
 | `<version>` | 版本号 |
 | `<description>` | 内容描述 |
 | `<maintainer>` | 维护者 |
 | `<license>` | 软件许可证 |
 | `<buildtool_depend>` | 编译构建工具，通常为catkin |
 | `<depend>` | 指定依赖项为编译、导出、运行需要的依赖，最常用 |
 | `<build_depend>` | 编译依赖项 |
 | `<build_export_depend>` | 导出依赖项 |
 | `<exec_depend>` | 运行依赖项 |
 | `<test_depend>` | 测试用例依赖项 |
 | `<doc_depend>` | 文档依赖项 |

`<depend>`相当于`<build_export_depend>`、`<exec_depend>`、`<build_depend>` ,相当于将之前的`build`和`run`依赖项描述进行了细分。

每个ROS功能包都至少有一个依赖项，一个指定了构建、执行、测试、文档依赖项的 xml 文件如下：

```xml
<package format = "2">
  <name> sensor_funsion </name>
  <version> 1.2.3 </version>
  <description>
      This package fusion point cloud and image.
  </description>
  <url> www.xxx.com </url>
  <author> DLonng </author>
  <maintainer email="xxx@xxx.com"> xxx </maintainer>
  <license> BSD </license>

  <buildtool_depend> catkin </buildtool_depend>

  <depend> roscpp </depend>
  <depend> std_msgs </depend>

  <build_depend> message_generation </build_depend>

  <exec_depend> message_runtime </exec_depend>
  <exec_depend> rospy </exec_depend>

  <test_depend> python-mock </test_depend>

  <doc_depend> doxygen </doc_depend>
</package>
```

### 2.2. format1

`pacakge.xml`遵循xml标签文本的写法，由于版本更迭原因，现在有两种格式并存（format1与format2），不过区别不大。老版本（format1）的`pacakge.xml`通常包含以下标签:

 | 标签名 | 作用 |
 | :---: | :---: |
 | `<pacakge>` | 根标记文件 |
 | `<name>` | 包名 |
 | `<version>` | 版本号 |
 | `<description>` | 内容描述 |
 | `<maintainer>` | 维护者 |
 | `<license>` | 软件许可证 |
 | `<buildtool_depend>` | 编译构建工具，通常为catkin |
 | `<build_depend>` | 编译依赖项 |
 | `<run_depend>` | 运行依赖项 |
 | `<test_depend>` | 测试用例依赖项 |

```xml
<package>
  <name> sensor_funsion </name>
  
  <version> 1.2.3 </version>
  
  <description>
      This package fusion point cloud and image.
  </description>
  
  <url> www.xxx.com </url>
  
  <author> DLonng </author>
  
  <maintainer email="xxx@xxx.com"> xxx </maintainer>
  
  <license> BSD </license>
</package>
```

## 3. pacakge.xml例子

为了说明pacakge.xml写法，还是以turtlesim软件包为例，其`pacakge.xml`文件内容如下，我们添加了相关的注释：

```xml
<?xml version="1.0"?>       <!--本示例为老版本的pacakge.xml-->
<package>                   <!--pacakge为根标签，写在最外面-->
  <name>turtlesim</name>
  <version>0.8.1</version>
  <description>
    turtlesim is a tool made for teaching ROS and ROS packages.
  </description>
  <maintainer email="dthomas@osrfoundation.org">Dirk Thomas</maintainer>
  <license>BSD</license>

  <url type="website">http://www.ros.org/wiki/turtlesim</url>
  <url type="bugtracker">https://github.com/ros/ros_tutorials/issues</url>
  <url type="repository">https://github.com/ros/ros_tutorials</url>
  <author>Josh Faust</author>

  <!--编译工具为catkin-->
  <buildtool_depend>catkin</buildtool_depend>

  <!--编译时需要依赖以下包-->  
  <build_depend>geometry_msgs</build_depend>    
  <build_depend>qtbase5-dev</build_depend>
  <build_depend>message_generation</build_depend>
  <build_depend>qt5-qmake</build_depend>
  <build_depend>rosconsole</build_depend>
  <build_depend>roscpp</build_depend>
  <build_depend>roscpp_serialization</build_depend>
  <build_depend>roslib</build_depend>
  <build_depend>rostime</build_depend>
  <build_depend>std_msgs</build_depend>
  <build_depend>std_srvs</build_depend>
  
  <!--运行时需要依赖以下包-->
  <run_depend>geometry_msgs</run_depend>
  <run_depend>libqt5-core</run_depend>
  <run_depend>libqt5-gui</run_depend>
  <run_depend>message_runtime</run_depend>
  <run_depend>rosconsole</run_depend>
  <run_depend>roscpp</run_depend>
  <run_depend>roscpp_serialization</run_depend>
  <run_depend>roslib</run_depend>
  <run_depend>rostime</run_depend>
  <run_depend>std_msgs</run_depend>
  <run_depend>std_srvs</run_depend>
</package>
```

以上内容是老版本（format1）的写法，如果要写成新版本（format2）则可以改为：

```xml
<?xml version="1.0"?>
<package format="2">      <!--在声明pacakge时指定format2，为新版格式-->
  <name>turtlesim</name>
  <version>0.8.1</version>
  <description>
    turtlesim is a tool made for teaching ROS and ROS packages.
  </description>
  <maintainer email="dthomas@osrfoundation.org">Dirk Thomas</maintainer>
  <license>BSD</license>

  <url type="website">http://www.ros.org/wiki/turtlesim</url>
  <url type="bugtracker">https://github.com/ros/ros_tutorials/issues</url>
  <url type="repository">https://github.com/ros/ros_tutorials</url>
  <author>Josh Faust</author>

  <!--编译工具为catkin-->
  <buildtool_depend>catkin</buildtool_depend>

  <!--用depend来整合build_depend和run_depend-->  
  <depend>geometry_msgs</depend>
  <depend>rosconsole</depend>
  <depend>roscpp</depend>
  <depend>roscpp_serialization</depend>
  <depend>roslib</depend>
  <depend>rostime</depend>
  <depend>std_msgs</depend>
  <depend>std_srvs</depend>

  <!--build_depend标签未变-->
  <build_depend>qtbase5-dev</build_depend>
  <build_depend>message_generation</build_depend>
  <build_depend>qt5-qmake</build_depend>

  <!--run_depend要改为exec_depend-->
  <exec_depend>libqt5-core</exec_depend>
  <exec_depend>libqt5-gui</exec_depend>
  <exec_depend>message_runtime</exec_depend>
</package>
```
