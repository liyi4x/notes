# Launch

## 1. 通过Launch文件启动ROS

当一个机器人系统拥有多个Node时，可以使用launch文件来组织这些节点。通过对launch文件的配置，可以达到“一键启动的效果”`roslaunch`就是一个启动管理器。_一般情况下整个系统的启动文件是在`{name}_bringup`包下的`{name}.launch`_

```bash
roslaunch pkg_name file_name.launch
```

`roslaunch`命令会首先检查`roscore`是否成功启动，即判断节点管理器`master`是否正常运行如若没有启动会在启动节点之前先启动`roscere`

## 2. launch文件语法格式

`launch`文件本质也是`xml`文件，因此语法格式满足`xml`文件的标准，它包括的标签

| 标签 | 作用 |
| :------: | :------: |
| `<launch>` | 根标签 |
| `<node>` | 需要启动的node及其参数 |
| `<include>` | 包含其他launch |
| `<machine>` | 指定运行的机器 |
| `<env-loader>` | 设置环境变量 |
| `<param>` | 定义参数到参数服务器 |
| `<rosparam>` | 启动yaml文件参数到参数服务器 |
| `<arg>` | 定义变量 |
| `<remap>` | 设定参数映射 |
| `<group>` | 设定命名空间 |

每个标签有多个属性，具体含义[参考wiki](http://wiki.ros.org/roslaunch/XML)

- [`<node>`](http://wiki.ros.org/roslaunch/XML/node)
    - `name` 节点重命名，相当于节点的第二个名字，这个可以重复
    - `pkg` 节点所在的package名
    - `type` 在package里的节点名，使用`rosrun`命令是的node名
- [`<arg>`](http://wiki.ros.org/roslaunch/XML/arg)
    - `name` 变量名
    - `default` 默认值（可选）
    - `value` 变量值（可选）
- [`<include>`](http://wiki.ros.org/roslaunch/XML/include)
    - `file` 格式为`$(find pkg_name)/path/filename.xml`
- [`<rosparam>`](http://wiki.ros.org/roslaunch/XML/rosparam)

## 3. 示例文件

最简单的`launch`文件只需要有node即可，比如启动`rospy_tutorials`包下的`talker`节点

```xml
<launch>
    <node name="talker" pkg="rospy_tutorials" type="talker" />
</launch>
```

而在实际应用时的`launch`文件就复杂一些，`robot_sim_demo`下的`robot_spawn.launch`文件如下

```xml
<launch>
    <!--arg是launch标签中的变量声明，arg的name为变量名，default或者value为值，在下面的 include 标签中引用了-->
    <arg name="robot" default="xbot-u"/>
    <arg name="debug" default="false"/>
    <arg name="gui" default="true"/>
    <arg name="headless" default="false"/>

    <!--include用来嵌套仿真场景的launch文件-->
    <include file="$(find gazebo_ros)/launch/empty_world.launch"> 
        <arg name="world_name" value="$(find robot_sim_demo)/worlds/ROS-Academy.world"/>
        <arg name="debug" value="$(arg debug)" />
        <arg name="gui" value="$(arg gui)" />
        <arg name="paused" value="false"/>
        <arg name="use_sim_time" value="true"/>
        <arg name="headless" value="$(arg headless)"/>
    </include>

    <!--嵌套了机器人的launch文件-->
    <include file="$(find robot_sim_demo)/launch/include/$(arg robot).launch.xml" />

    <!--如果你想连同RViz一起启动，可以按照以下方式加入RViz这个node-->
    <!-- node name="rviz" pkg="rviz" type="rviz" args="-d $(find robot_sim_demo)/urdf_gazebo.rviz" / -->
</launch>
```

`robot_spawn.launch`中引用的`xbot-u.launch.xml`文件的内容如下

```xml
<launch>
    <arg name="x" default="5.0" />
    <arg name="y" default=".0" />
    <arg name="z" default="0.0" />
    <arg name="yaw" default="-2.0" />

    <!-- Setup controllers -->
    <!-- rosparam file="$(find fetch_gazebo)/param/freight_controllers.yaml" command="load" / -->

    <!-- URDF and TF support -->
    <param name="robot_description" command="$(find xacro)/xacro.py $(find robot_sim_demo)/urdf/robot.xacro" />
    <!-- Put a robot in gazebo, make it look pretty -->
    <node name="urdf_spawner" pkg="gazebo_ros" type="spawn_model" respawn="false" output="screen" 
        args="-urdf -x $(arg x) -y $(arg y) -z $(arg z) -Y $(arg yaw) -model xbot-u -param robot_description"/>

    <!--node name="joint_state_publisher" pkg="joint_state_publisher" type="joint_state_publisher">
    <param name="use_gui" value="false"/>
    </node-->

    <!--Load the joint controllers to param server-->
    <rosparam file="$(find robot_sim_demo)/param/xbot-u_control.yaml" command="load"/>

    <!--Load controllers-->
    <node name="spawner" pkg="controller_manager" type="spawner" respawn="false"
        output="screen" ns="/xbot" args="joint_state_controller
        yaw_platform_position_controller
        pitch_platform_position_controller"
    />  <!--mobile_base_controller-->


    <!-- convert joint states to TF transforms for rviz, etc -->
    <node name="robot_state_publisher" pkg="robot_state_publisher" type="robot_state_publisher" ns="/xbot" respawn="false" output="screen">
        <param name="publish_frequency" value="20.0"/>
    </node>

    <!-- Publish base_scan_raw if anything subscribes to it -->
    <!-- <node name="publish_base_scan_raw" pkg="topic_tools" type="relay" args="base_scan base_scan_raw" >
        <param name="lazy" type="bool" value="True"/>
    </node> -->

    <!-- Start a mux between application and teleop, but the switch must be called by service rather than automatically -->
    <!-- <node pkg="topic_tools" type="mux" name="cmd_vel_mux" respawn="true" args="/cmd_vel /cmd_vel_mux/input/teleop /cmd_vel_mux/input/navi">
        <remap from="mux" to="cmd_vel_mux"/>
    </node> -->

    <!-- To make the interface of simulation identical to real XBot -->
    <node pkg="nodelet" type="nodelet" name="mobile_base_nodelet_manager" args="manager"/>
    <node pkg="nodelet" type="nodelet" name="cmd_vel_mux"  args="load yocs_cmd_vel_mux/CmdVelMuxNodelet mobile_base_nodelet_manager">
        <param name="yaml_cfg_file" value="$(find robot_sim_demo)/param/mux.yaml"/>
        <remap from="cmd_vel_mux/output/cmd_vel" to="cmd_vel"/>
    </node>
</launch>

```
