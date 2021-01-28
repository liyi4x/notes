# service in roscpp

[Service和srv相关解释](./Service和srv.md)

以`service_demo`为例，定义`Greeting.srv`服务，接收姓名和年龄，返回一个字符串。一个节点发出服务请求，另一个节点提供服务，返回问候语

## 1. 创建服务

### 1.1. `Greeting.srv`文件

```text
string name        #短横线上边部分是服务请求的数据
int32 age          
---                #短横线下面是服务回传的内容。
string feedback
```

- 对于`Greeting.srv`而言，最终也被编译成`Greeting`结构体，在`service_demo`命名空间下
- 服务的两部分`Request`和`Response`是两个结构体，并且被嵌套在`Greeting`结构体下

## 2. 创建节点

### 2.1. server节点

`service_demo/src/server.cpp`文件

```cpp
# include "ros/ros.h"
# include "service_demo/Greeting.h"
# include "string"

// 定义请求处理函数
bool handle_function(service_demo::Greeting::Request &req, service_demo::Greeting::Response &res)
{
    // 此处我们对请求直接输出
    ROS_INFO("Request from %s with age %d ", req.name.c_str(), req.age);

    // 返回一个反馈，将response设置为"..."
    res.feedback = "Hi " + req.name + ". I'm server!";
    return true;
}

int main(int argc, char **argv)
{
    // 初始化节点，命名为"greetings_server"
    ros::init(argc, argv, "greetings_server");

    // 定义service的server端，service名称为“greetings”，收到request请求之后传递给handle_function进行处理
    ros::NodeHandle nh;
    ros::ServiceServer service = nh.advertiseService("greetings", handle_function);

    // 调用可
    ros::spin();

    return 0;
}
```

- 服务端对外提供服务，通过`nh`句柄创建`service`，`advertiseService()`有两个参数，服务名和处理函数
- 当`server`节点收到服务请求时会调用`handle_function()`
- `handle_function()`的两个参数是`Greeting`的`Request`和`Response`两部分的引用，返回值是布尔型，是否成功执行
- 需要调用`ros::spin();`进行消息队列处理

### 2.2. client节点

`service_demo/src/client.cpp`文件

```cpp
# include "ros/ros.h"
# include "service_demo/Greeting.h"

int main(int argc, char **argv)
{
    // 初始化，节点命名为"greetings_client"
    ros::init(argc, argv, "greetings_client");

    // 定义service客户端，service名字为“greetings”，service类型为service_demo
    ros::NodeHandle nh;
    ros::ServiceClient client = nh.serviceClient<service_demo::Greeting>("greetings");

    // 实例化srv，设置其request消息的内容，这里request包含两个变量，name和age，见Greeting.srv
    service_demo::Greeting srv;
    srv.request.name = "HAN";
    srv.request.age = 20;

    if (client.call(srv))
    {
        // 注意我们的response部分中的内容只包含一个变量response，另，注意将其转变成字符串
        ROS_INFO("Response from server: %s", srv.response.feedback.c_str());
    }
    else
    {
        ROS_ERROR("Failed to call service Service_demo");
        return 1;
    }
    return 0;
}

```

- 通过nh句柄创建`ros::ServiceClient`对象的时候需要指定服务结构体和服务名，这里说的服务名是`server`节点在创建服务的时候起的名字
- 通过`client.call()`请求服务，参数为`service_demo::Greeting`的实例
- `srv.response.feedback`是string类对象，在输出时通过`c_str()`成员函数转换为字符数组。参考[C++中c_str()函数的用法](https://blog.csdn.net/JIEJINQUANIL/article/details/51547027)

## 3. 项目结构文件

### 3.1. `CMakeLists.txt`

```cmake
cmake_minimum_required(VERSION 2.8.3)
project(service_demo)

find_package(catkin REQUIRED COMPONENTS
    roscpp
    std_msgs
    message_generation   # 需要添加的地方
)

add_service_files(
   FILES
   Greeting.srv
)

generate_messages(DEPENDENCIES 
    std_msgs
)

include_directories(
  include
  ${catkin_INCLUDE_DIRS}
)

add_executable(server src/server.cpp)
add_dependencies(server ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})
target_link_libraries(server ${catkin_LIBRARIES})

add_executable(client src/client.cpp)
add_dependencies(client ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})
target_link_libraries(client ${catkin_LIBRARIES})
```

- 和消息一样，新建服务也需要`generate_messages()`否则在代码中`greeting.h`头文件会报错

### 3.2. `packages.xml`

```xml
<package format = "2">
  <name> service_demo </name>
  <version> 0.1.0 </version>
  <description>
      This package service_demo
  </description>

  <maintainer email="user@example.com">someone</maintainer>
  <license> mit </license>

  <buildtool_depend> catkin </buildtool_depend>

  <depend> roscpp </depend>
  <depend> std_msgs </depend>

  <build_depend> message_generation </build_depend>

  <exec_depend> message_runtime </exec_depend>

</package>
```
