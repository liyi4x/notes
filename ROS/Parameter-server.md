# Parameter Server

## 1. 简介

参数服务器是节点存储参数的地方、用于配置参数，全局共享参数。参数服务器使用互联网传输，在节点管理器中运行，实现整个通信过程。参数服务器维护着一个字典，用来存放各个节点的参数和配置信息，它是一种相对静态的通信方式

## 2. 维护方式

参数服务器中的字典可通过三种方式维护

- 命令行
- launch文件
- Node节点源码

### 2.1. 命令行方式

主要是通过使用`rosparam`命令来管理

| rosparam 命令 | 作用 |
| :------: | :------: |
| `rosparam list`| 列出参数名称 |
| `rosparam get param_key` | 显示参数 |
| `rosparam set param_key param_value` | 设置参数 |
| `rosparam delete` |  删除参数 |
| `rosparam load file_name` | 从文件加载参数 |
| `rosparam dump file_name` | 保存参数到文件 |

在使用`load`和`dump`命令时的文件使用`yaml`格式

```yaml
name:'Zhangsan'
age:20
gender:'M'
score:{Chinese:80,Math:90}
score_history:[85,82,88,90]
```

参考[YAML 语言教程](http://www.ruanyifeng.com/blog/2016/07/yaml.html)

### 2.2. launch文件

launch文件中有很多标签，而与参数服务器相关的标签只有两个，一个是`<param>`，另一个是`<rosparam>`。其中`<param>`标签只能设置一个参数

- 通过脚本执行结果设置`<param>`的值

    ```xml
    <!-- 读取机器人模型参数 -->
    <param name="robot_description" command="$(find xacro)/xacro.py $(find robot_sim_demo)/urdf/robot.xacro" />
    
    <!--在Gazebo中启动机器人模型-->
    <node name="urdf_spawner" pkg="gazebo_ros" type="spawn_model" respawn="false" output="screen" 
        args="-urdf -x $(arg x) -y $(arg y) -z $(arg z) -Y $(arg yaw) -model xbot2 -param robot_description"/>
    ```

    这里的`robot_description`的value是用`xacro.py`执行`robot.xacro`文件之后的结果，再在启动`urdf_spawner`节点时的启动参数中加入`robot_description`这个参数

- 通过`<rosparam>`设置

    ```xml
    <!--把关节控制的配置信息读到参数服务器-->
    <rosparam file="$(find robot_sim_demo)/config/xbot2_control.yaml" command="load"/>

    <!--启动关节控制器-->
    <node name="spawner" pkg="controller_manager" type="spawner" respawn="false"
            output="screen" ns="/xbot2" args="joint_state_controller
            yaw_platform_position_controller
            pitch_platform_position_controller
            "/><!--mobile_base_controller-->

    <!-- 将关节状态转换为TF变-->
    <node name="robot_state_publisher" pkg="robot_state_publisher" type="robot_state_publisher" ns="/xbot2"
            respawn="false" output="screen">
    ```

    这里`<rosparam>`标签的作用相当于使用命令行`rosparm load xbot2_control.yaml`加载yaml文件

- 直接设置`<param>`的值

    ```xml
    <param name="publish_frequency" value="100.0"/>
    ```

### 2.3. Node节点源码

通过`roscpp`或者`rospy`中提供的相关API直接在节点源码中维护参数的值
