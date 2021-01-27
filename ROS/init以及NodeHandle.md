# init以及NodeHandle

## 1. 节点初始化

ROS程序和其他Cpp程序的主要区别在于

- 它调用了`ros::init()`函数，完成了ROS节点的初始化，节点名称等信息
- 创建了`ros::NodeHandle`对象，通过这个节点句柄完成ROS通信的一些功能，比如创建Publisher和Subscriber等

句柄（Handle）的概念类似于`this`指针，NodeHandle相当于是对节点资源的描述，有了这个句柄就可以操作节点的具体功能实现

### 1.1. ros::init()

函数有三种重载

```cpp
ROSCPP_DECL void init(int &argc, char **argv, const std::string& name, uint32_t options = 0);

ROSCPP_DECL void init(const M_string& remappings, const std::string& name, uint32_t options = 0);

ROSCPP_DECL void init(const VP_string& remapping_args, const std::string& name, uint32_t options = 0);
```

- `argc` `argv`是系统传入参数，在终端运行程序的时候传入
- `name`是节点名字，字符串类型，节点名字需要在同一个ROS系统中保持唯一；如果出现重名，之前的节点会被关闭
- `options`可以设置对节点的具体操作，默认值0，因此在调用的时候可以不指定该参数，具体取值为`InitOption`枚举体。

当ROS系统中有多个功能相同的节点的时候，可以使用匿名节点，`options`参数设置为`ros::init_options::AnonymousName`，在创建节点的时候ROS系统会自动在节点名后加随机数来保证ROS系统中的节点名字唯一

```cpp
enum InitOption
{
    /**
    * Don't install a SIGINT handler.  You should install your own SIGINT handler in this
    * case, to ensure that the node gets shutdown correctly when it exits.
    */
    NoSigintHandler = 1 << 0,
    /** \brief Anonymize the node name.  Adds a random number to the end of your node's name, to make it unique.
    */
    AnonymousName = 1 << 1,
    /**
    * \brief Don't broadcast rosconsole output to the /rosout topic
    */
    NoRosout = 1 << 2,
};
```

`ros::init()`函数主要功能是

```cpp
network::init(remappings);
master::init(remappings);
this_node::init(name, remappings, options);
file_log::init(remappings);
param::init(remappings);
```

参考[ROS程序的初始化](https://blog.csdn.net/wanghuiquan0712/article/details/78052093)

### 1.2. NodeHandle

NodeHandle是Node的句柄，用来对当前节点进行各种操作。在ROS中，NodeHandle是一个定义好的类，通过`include<ros/ros.h>`，我们可以创建这个类，以及使用它的成员函数。各种类型的通信都需要用NodeHandle来创建完成

```cpp
//创建话题的publisher 
ros::Publisher advertise(const string &topic, uint32_t queue_size, bool latch=false); 
//第一个参数为发布话题的名称
//第二个是消息队列的最大长度，如果发布的消息超过这个长度而没有被接收，那么就的消息就会出队。通常设为一个较小的数即可。
//第三个参数是是否锁存。某些话题并不是会以某个频率发布，比如 /map 这个topic，只有在初次订阅或者地图更新这两种情况下，/map才会发布消息。这里就用到了锁存。

//创建话题的subscriber
ros::Subscriber subscribe(const string &topic, uint32_t queue_size, void(*)(M));
//第一个参数是订阅话题的名称
//第二个参数是订阅队列的长度，如果受到的消息都没来得及处理，那么新消息入队，旧消息就会出队
//第三个参数是回调函数指针，指向回调函数来处理接收到的消息

//创建服务的server，提供服务
ros::ServiceServer advertiseService(const string &service, bool(*srv_func)(Mreq &, Mres &)); 
//第一个参数是service名称
//第二个参数是服务函数的指针，指向服务函数。指向的函数应该有两个参数，分别接受请求和响应。

//创建服务的client
ros::ServiceClient serviceClient(const string &service_name, bool persistent=false); 
//第一个函数式service名称
//第二个参数用于设置服务的连接是否持续，如果为true，client将会保持与远程主机的连接，这样后续的请求会快一些。通常我们设为flase

//查询某个参数的值
bool getParam(const string &key, std::string &s); 
bool getParam (const std::string &key, double &d) const；
bool getParam (const std::string &key, int &i) const；
//从参数服务器上获取key对应的值，已重载了多个类型

//给参数赋值
void setParam (const std::string &key, const std::string &s) const；
void setParam (const std::string &key, const char *s) const;
void setParam (const std::string &key, int i) const;
//给key对应的val赋值，重载了多个类型的val
```

## 2. 节点关闭

通常我们要关闭一个节点可以直接在终端上按`Ctrl+C`，系统会自动触发`SIGINT`句柄来关闭这个进程。 你也可以通过调用`ros::shutdown()`来手动关闭节点，但通常我们很少这样做。默认情况下终端中使用`Ctrl+C`，最终会调用`ros::shutdown()`，对于一些在退出时需要其他操作的节点，也可以自定义退出的操作，具体参考[ROS节点的初始化及退出详解](https://blog.csdn.net/u013834525/article/details/80119047)

```cpp
#include<ros/ros.h>
int main(int argc, char** argv)
{
    ros::init(argc, argv, "your_node_name"); 
    ros::NodeHandle nh;
    //....节点功能
    //....
    ros::spin();//用于触发topic、service的响应队列
    return 0;
}
```
