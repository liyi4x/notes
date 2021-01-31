# 时钟

## 1. Time与Duration

在ROS里面对于机器人的控制会经常使用到关于时间的功能，设定程序等待时间、设定定时器等。roscpp提供了两种时间表示方法

- 时刻`ros::Time`
- 时间`ros::Duration`

`Time`和`Duration`都由秒和纳秒组成。表示方法为

```text
int32 sec
int32 nsec
```

在使用这两种功能时需要引用相应的头文件`#include <ros/time.h>`和`#include <ros/duration.h>`

```cpp
ros::Time begin = ros::Time::now(); //获取当前时间
ros::Time at_some_time1(5, 20000000);  //5.2s
ros::Time at_some_time2(5.2) //同上，重载了float类型和两个uint类型的构造函数
ros::Duration one_hour(60 * 60, 0); //1h

double secs1 = at_some_time1.toSec();//将Time转为double型时间
double secs2 = one_hour.toSec();//将Duration转为double型时间
```

时间和持续时间算术运算

- 1 hour + 1 hour = 2 hours (duration + duration = duration)
- 2 hours - 1 hour = 1 hour (duration - duration = duration)
- Today + 1 day = tomorrow (time + duration = time)
- Today - tomorrow = -1 day (time - time = duration)
- Today + tomorrow = error (time + time is undefined)

```cpp
ros::Time t1 = ros::Time::now() - ros::Duration(5.5);   //t1是5.5s前的时刻，Time加减Duration返回都是Time
ros::Time t2 = ros::Time::now() + ros::Duration(3.3);   //t2是当前时刻往后推3.3s的时刻
ros::Duration d1 = t2 - t1;     //从t1到t2的时长，两个Time相减返回Duration类型
ros::Duration d2 = d1 -ros::Duration(0,300);   //两个Duration相减，还是Duration
```

## 2. Sleeping 和 Rates

`bool ros::Duration::sleep()`可以设置延迟一段时间，比如

```cpp
ros::Duration(0.5).sleep();     //用Duration对象的sleep方法休眠

ros::Duration one_second(1, 0);
one_second.sleep();     //休眠1秒
```

`ros::Rate`类可以设置频率

```cpp
ros::Rate r(10); // 10 hz
while (ros::ok())
{
    // ... do some work ...
    r.sleep();
}
```

## 3. Timer

Rate的作用是指定一个频率，与之类似的是ROS中的定时器Timer，它是通过设定回调函数和触发时间来实现某些动作的反复执行，就和传统意义上的定时器功能一致

`ros::Timer`是一个类，使用时添加`#include "ros/ros.h"`即可，定时器也是需要通过使用节点句柄来创建

```cpp
void callback1(const ros::TimerEvent&)
{
    ROS_INFO("Callback 1 triggered");
}

void callback2(const ros::TimerEvent&)
{
    ROS_INFO("Callback 2 triggered");
}

int main(int argc, char **argv)
{
    ros::init(argc, argv, "talker");
    ros::NodeHandle n;

    ros::Timer timer1 = n.createTimer(ros::Duration(0.1), callback1);  //timer1每0.1s触发一次callback1函数
    ros::Timer timer2 = n.createTimer(ros::Duration(1.0), callback2);  //timer2每1.0s触发一次callback2函数

    ros::spin();  //千万别忘了spin，只有spin了才能真正去触发回调函数

    return 0;
}
```

回调函数中的`TimerEvent`结构体

```cpp
struct TimerEvent
{
    Time last_expected;             ///< 理想状态下上一次回调函数运行的时刻
    Time last_real;                 ///< 实际情况下上一次回调函数运行的时刻

    Time current_expected;          ///< In a perfect world, this is when the current callback should be happening
    Time current_real;              ///< This is when the current callback was actually called (Time::now() as of the beginning of the callback)

    struct
    {
        WallDuration last_duration;   ///< How long the last callback ran for, always in wall-clock time
    } profile;
};
```
