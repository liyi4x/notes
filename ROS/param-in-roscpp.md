# param in roscpp

[Parameter server 相关解释](./ROS/Parameter-server.md)

roscpp为参数服务器的维护提供了两套api

- `ros::param`命名空间下的函数
- `ros::NodeHandle`类的成员函数

## 1. param_demo

`param_demo/src/param.cpp`文件

```cpp
#include<ros/ros.h>

int main(int argc, char **argv){
    ros::init(argc, argv, "param_demo");
    ros::NodeHandle nh;
    int parameter1, parameter2, parameter3, parameter4, parameter5;
    
    //Get Param的三种方法
    //ros::param::get()获取参数“param1”的value，写入到parameter1上
    bool ifget1 = ros::param::get("param1", parameter1);
    
    //ros::NodeHandle::getParam()获取参数
    bool ifget2 = nh.getParam("param2",parameter2);
   
    //如果get不到指定的param，它可以给parameter3指定一个默认值(如33333)
        nh.param("param3", parameter3, 33333);
    
    if(ifget1)
        ROS_INFO("Get param1 = %d", parameter1);
    else
        ROS_WARN("Didn't retrieve param1");
    if(ifget2)
        ROS_INFO("Get param2 = %d", parameter2);
    else
        ROS_WARN("Didn't retrieve param2");
    if(nh.hasParam("param3"))
        ROS_INFO("Get param3 = %d", parameter3);
    else
        ROS_WARN("Didn't retrieve param3");


    //Set Param的两种方法
    //ros::param::set()设置参数
    parameter4 = 4;
    ros::param::set("param4", parameter4);

    //ros::NodeHandle::setParam()设置参数
    parameter5 = 5;
    nh.setParam("param5",parameter5);
    
    ROS_INFO("Param4 is set to be %d", parameter4);
    ROS_INFO("Param5 is set to be %d", parameter5);


    //Check Param的两种方法
    //ros::NodeHandle::hasParam()
    bool ifparam5 = nh.hasParam("param5");

    //ros::param::has()
    bool ifparam6 = ros::param::has("param6");

    if(ifparam5) 
        ROS_INFO("Param5 exists");
    else
        ROS_INFO("Param5 doesn't exist");
    if(ifparam6) 
        ROS_INFO("Param6 exists");
    else
        ROS_INFO("Param6 doesn't exist");


    //Delete Param的两种方法
    //ros::NodeHandle::deleteParam()
    bool ifdeleted5 = nh.deleteParam("param5");

    //ros::param::del()
    bool ifdeleted6 = ros::param::del("param6");
    

    if(ifdeleted5)
        ROS_INFO("Param5 deleted");
    else
        ROS_INFO("Param5 not deleted");
    if(ifdeleted6)
        ROS_INFO("Param6 deleted");
    else
        ROS_INFO("Param6 not deleted");


    ros::Rate rate(0.3);
    while(ros::ok()){
        int parameter = 0;
        
        ROS_INFO("=============Loop==============");
        //roscpp中尚未有ros::param::getallparams()之类的方法
        if(ros::param::get("param1", parameter))
            ROS_INFO("parameter param1 = %d", parameter);
        if(ros::param::get("param2", parameter))
            ROS_INFO("parameter param2 = %d", parameter);
        if(ros::param::get("param3", parameter))
            ROS_INFO("parameter param3 = %d", parameter);
        if(ros::param::get("param4", parameter))
            ROS_INFO("parameter param4 = %d", parameter);
        if(ros::param::get("param5", parameter))
            ROS_INFO("parameter param5 = %d", parameter);
        if(ros::param::get("param6", parameter))
            ROS_INFO("parameter param6 = %d", parameter);
        rate.sleep();
    }
}
```

`param_demo/launch/param_demo_cpp.launch`文件

```xml
<launch>
    <!--param参数配置-->
    <param name="param1" value="1" />
    <param name="param2" value="2" />
    <!--param name="table_description" command="$(find xacro)/xacro.py $(find gazebo_worlds)/objects/table.urdf.xacro" /-->

    <!--rosparam参数配置-->
    <rosparam>   
        param3: 3
        param4: 4
        param5: 5
    </rosparam>
    <!--以上写法将参数转成YAML文件加载，注意param前面必须为空格，不能用Tab，否则YAML解析错误-->
    <!--rosparam file="$(find robot_sim_demo)/config/xbot-u_control.yaml" command="load" /-->
    <node pkg="param_demo" type="param_demo" name="param_demo" output="screen" />
    
</launch>
```

## 2. 运行结果

执行`rosrun param_demo param_demo`的结果如下

```text
[ WARN] [1611930965.096364533]: Didn't retrieve param1
[ WARN] [1611930965.096428485]: Didn't retrieve param2
[ WARN] [1611930965.097569665]: Didn't retrieve param3
[ INFO] [1611930965.099216747]: Param4 is set to be 4
[ INFO] [1611930965.099250417]: Param5 is set to be 5
[ INFO] [1611930965.100457062]: Param5 exists
[ INFO] [1611930965.100498410]: Param6 doesn't exist
[ INFO] [1611930965.102516533]: Param5 deleted
[ INFO] [1611930965.102558813]: Param6 not deleted
[ INFO] [1611930965.102579994]: =============Loop==============
[ INFO] [1611930965.104550994]: parameter param4 = 4
[ INFO] [1611930968.438369688]: =============Loop==============
[ INFO] [1611930968.441288282]: parameter param4 = 4
```

- param1 2定义在launch文件中，直接通过`rosrun`打开节点的话，没有加载launch文件中对这几个参数的定义，所以节点无法读取参数
- param3 读取的时候参数服务器中并没有这个参数，但是针对`param3`的读取方法是`nh.param("param3", parameter3, 33333);`，这个参数的第三个变量是默认值，因此如若参数服务器中没有`param3`这个参数，会给`parameter3`变量赋默认值`33333`

```cpp
template<typename T>
bool param(const std::string& param_name, T& param_val, const T& default_val) const
{
    if (hasParam(param_name))
    {
        if (getParam(param_name, param_val))
        {
            return true;
        }
    }

param_val = default_val;
return false;
}
```

执行`roslaunch param_demo param_demo_cpp.launch`的结果如下

```text
process[param_demo-1]: started with pid [29494]
[ INFO] [1611929812.346775227]: Get param1 = 1
[ INFO] [1611929812.346835230]: Get param2 = 2
[ INFO] [1611929812.347514679]: Get param3 = 3
[ INFO] [1611929812.349519691]: Param4 is set to be 4
[ INFO] [1611929812.349576397]: Param5 is set to be 5
[ INFO] [1611929812.351273386]: Param5 exists
[ INFO] [1611929812.351307720]: Param6 doesn't exist
[ INFO] [1611929812.353115211]: Param5 deleted
[ INFO] [1611929812.353152792]: Param6 not deleted
[ INFO] [1611929812.353196434]: =============Loop==============
[ INFO] [1611929812.354020600]: parameter param1 = 1
[ INFO] [1611929812.354780836]: parameter param2 = 2
[ INFO] [1611929812.355424684]: parameter param3 = 3
[ INFO] [1611929812.356337908]: parameter param4 = 4
[ INFO] [1611929815.688232581]: =============Loop==============
[ INFO] [1611929815.689976152]: parameter param1 = 1
[ INFO] [1611929815.691289476]: parameter param2 = 2
[ INFO] [1611929815.692041662]: parameter param3 = 3
[ INFO] [1611929815.692681417]: parameter param4 = 4
```
