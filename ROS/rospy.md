# rospy

rospy是Python版本的ROS客户端库，提供了Python编程需要的接口，可以认为rospy就是一个Python的模块(Module)。这个模块位于`/opt/ros/kineetic/lib/python2.7/dist-packages/rospy`之中。

rospy包含的功能与roscpp相似，都有关于node、topic、service、param、time相关的操作。rospy没有一个NodeHandle，像创建publisher、subscriber等操作都被直接封装成了rospy中的函数或类，调用起来简单直观。

ROS中绝大多数基本指令，例如`rostopic`, `roslaunch`都是用python开发的，简单轻巧。

## 1. rospy代码组织形式

对于一些小体量的代码，直接把`*.py`文件放到`src/script/`目录下即可

```text
robot_sim_demo
├── CMakeLists.txt
├── package.xml
...
└── scripts
    └── robot_keyboard_teleop.py
```

对于一些体量较大的项目，在src下建立一个与你的package同名的路径，其中存放`__init__.py`以及你的模块文件

```text
robot_sim_demo
├── CMakeLists.txt
├── package.xml
...
├── scripts
│   └── robot_keyboard_teleop.py
└── src
    └── robot_sim_demo
        ├── a.py
        ├── b.py
        └── __init__.py
```

`__init__.py`可以将其所在目录变为一个python包，可以在其他的`*.py`文件中`import`这个包，即可使用这个包的功能代码。我们在导入一个包时，实际上是导入了它的`__init__.py`文件。这样我们可以在`__init__.py`文件中批量导入我们所需要的模块，而不再需要一个一个的导入。

参考[Python __init__.py作用详解](https://www.cnblogs.com/Lands-ljk/p/5880483.html)

## 2. init和node相关

参考[init以及NodeHandle](./init以及NodeHandle.md)

| 返回值 | 方法 | 作用 |
| :------: | :------: | :------: |
|  | rospy.init_node(name, argv=None, anonymous=False)  | 注册和初始化node|
| MasterProxy| rospy.get_master() | 获取master的句柄 |
| bool| rospy.is_shutdown() | 节点是否关闭 |
|  | rospy.on_shutdown(fn)| 在节点关闭时调用fn函数 |
| str | get_node_uri() | 返回节点的URI |
| str | get_name() | 返回本节点的全名 |
| str | get_namespace() | 返回本节点的名字空间 |

## 3. topic in rospy

参考[topic in roscpp](./topic-in-roscpp.md)

`learn_topic/src/scripts/py-listener.py`

```python
#!/usr/bin/env python
#coding=utf-8
import rospy
import math

from learn_topic.msg import gps

# 回调函数输入的应该是msg
def callback(gps):
    distance = math.sqrt(math.pow(gps.x, 2)+math.pow(gps.y, 2)) 
    rospy.loginfo('Listener: GPS: distance=%f, state=%s', distance, gps.state)

def listener():
    rospy.init_node('pylistener', anonymous=True)
    # Subscriber函数第一个参数是topic的名称，第二个参数是接受的数据类型，第三个参数是回调函数的名称
    rospy.Subscriber('gps_info', gps, callback)
    rospy.spin()

if __name__ == '__main__':
    listener()
```

`learn_topic/src/scripts/py-talker.py`

```python
#!/usr/bin/env python
#coding=utf-8
import rospy
#倒入自定义的数据类型
from learn_topic.msg import gps

def talker():
    #Publisher 函数第一个参数是话题名称，第二个参数是数据类型，现在就是我们定义的msg，最后一个是缓冲区的大小
    pub = rospy.Publisher('gps_info', gps , queue_size=10)
    rospy.init_node('pytalker', anonymous=True)

    #更新频率是1hz
    rate = rospy.Rate(1) 
    x=1.0
    y=2.0
    state='working'
    while not rospy.is_shutdown():
        #计算距离
        rospy.loginfo('Talker: GPS: x=%f ,y= %f',x,y)
        pub.publish(gps(state,x,y))
        x=1.03*x
        y=1.01*y
        rate.sleep()

if __name__ == '__main__':
    talker()
```

`rospy`和`roscpp`，对于消息（`*.msg`）的定义都是相同的，都需要在`CMakeLists.txt`和`package.xml`添加相应的依赖，但是又有所不同。

- 如果package中只包含python代码，则`CMakeLists.txt`中不需要使用`add_executable()`，仅添加相应的msg、rospy等依赖包即可
- 对于`scripts/`目录下的脚本文件需要有可执行的权限`chmod +x *.py`添加执行权限。如果没有执行权限，`rosrun`无法运行

## 4. service in rospy

参考[service in roscpp](./service-in-roscpp.md)

`learn_service/src/sctipts/py-server.py`

```python
#!/usr/bin/env python
#coding=utf-8
import rospy
from learn_service.srv import *


def handle_function(req):
    rospy.loginfo( 'Request from %s with age %d', req.name, req.age)
    return GreetingResponse("Hi %s. I' server!"%req.name)

def server_srv():
    rospy.init_node("greetings_server")
    
    s = rospy.Service("greetings", Greeting, handle_function)
    rospy.loginfo("Ready to handle the request:")
    # 阻塞程序结束
    rospy.spin()

if __name__=="__main__":
    server_srv()
```

- 在调用`rospy.Service()`函数的时候，回调函数`handle`的参数是`*.srv`文件中的`request`部分
- `handle_function()`函数的返回值是`*.srv`文件中的`respond`部分，类型为对象，本例中的`Greeting.srv`文件中respond部分为一字符串

`learn_service/src/sctipts/py-client.py`

```python
#!/usr/bin/env python
# coding:utf-8
import rospy
from learn_service.srv import *

def client_srv():
    rospy.init_node('greetings_client')
    # 等待有可用的服务 "greetings"
    rospy.wait_for_service("greetings")

    try:
        greetings_client = rospy.ServiceProxy("greetings", Greeting)

        # resp = greetings_client(name="LI", age=11)
        
        req = GreetingRequest(name="LI", age=11)
        resp = greetings_client.call(req)

        # 打印处理结果，注意调用response的方法，类似于从resp对象中调取response属性
        rospy.loginfo("Message From server:%s"%resp.feedback)
    except rospy.ServiceException, e:
        rospy.logwarn("Service call failed: %s"%e)

if __name__=="__main__":
    client_srv()
```

- 在请求服务的时候`greetings_client.call(req)`请求参数req是`Greeting.srv`的`request`部分，函数的返回值是服务请求的`respond`
- 以下请求服务效果相同
    - `greetings_client(name="HAN", age=20)`
    - `greetings_client.call(name="HAN", age=20)`
    - 如代码所示构造`GreetingRequest`类，再传入`greetings_client()`方法

## 5. param

参考[param in roscpp](./param-in-roscpp.md)

```python
#!/usr/bin/env python
# coding:utf-8

import rospy

def param_demo():
    rospy.init_node("param_demo")
    rate = rospy.Rate(1)
    while(not rospy.is_shutdown()):
        
        parameter1 = rospy.get_param("/param1")
        parameter2 = rospy.get_param("/param2", default=222)
        rospy.loginfo('Get param1 = %d', parameter1)
        rospy.loginfo('Get param2 = %d', parameter2)

        rospy.delete_param('/param2')

        rospy.set_param('/param2',2)
        
        ifparam3 = rospy.has_param('/param3')
        if(ifparam3):
            rospy.loginfo('/param3 exists')
        else:
            rospy.loginfo('/param3 does not exist')

        params = rospy.get_param_names()
        rospy.loginfo('param list: %s', params)

        rate.sleep()

if __name__=="__main__":
    param_demo()
```

- rospy比roscpp多了一个`rospy.get_param_names()`，能返回当前ros中的所有参数，返回值为python中的列表类型

## 6. 时钟

参考[roscpp时钟](./时钟.md)

### 6.1. Time和Duration

rospy中的关于时钟的操作和roscpp是一致的，都有Time、Duration和Rate三个类。

```python
time_now1 = rospy.get_rostime()  #当前时刻的Time对象 返回Time对象
time_now2 = rospy.Time.now() #同上
time_now3 = rospy.get_time() #得到当前时间，返回float 单位秒
time_4 = rospy.Time(5)  #创建5s的时刻
duration = rospy.Duration(3*60)  #创建3min时长
```

关于Time、Duration之间的加减法和类型转换，和roscpp中的完全一致，参考

### 6.2. sleep

```python
duration.sleep()  #挂起
rospy.sleep(duration)  #同上，这两种方式效果完全一致

loop_rate = Rate(5)     #利用Rate来控制循环频率
while(rospy.is_shutdown()):
    loop_rate.sleep()   #挂起，会考虑上次loop_rate.sleep的时间
```

### 6.3. Timer

rospy里的定时器和roscpp中的也类似，只不过不是用句柄来创建，而是直接rospy.Timer(Duration, callback)，第一个参数是时长，第二个参数是回调函数。

```python
def my_callback(event):
    print 'Timer called at ' + str(event.current_real)

rospy.Timer(rospy.Duration(2), my_callback)   #每2s触发一次callback函数
rospy.spin()
```

回调函数的传入值是TimerEvent类型，和roscpp中的`TimerEvent`结构体基本相同

```python
class TimerEvent(object):
    """
    Constructor.
    @param last_expected: in a perfect world, this is when the previous callback should have happened
    @type  last_expected: rospy.Time
    @param last_real: when the callback actually happened
    @type  last_real: rospy.Time
    @param current_expected: in a perfect world, this is when the current callback should have been called
    @type  current_expected: rospy.Time
    @param last_duration: contains the duration of the last callback (end time minus start time) in seconds.
                          Note that this is always in wall-clock time.
    @type  last_duration: float
    """
    def __init__(self, last_expected, last_real, current_expected, current_real, last_duration):
        self.last_expected    = last_expected
        self.last_real        = last_real
        self.current_expected = current_expected
        self.current_real     = current_real
        self.last_duration    = last_duration
```
