# TF in roscpp

## 数据类型

| 名称 | 数据类型 |
| :---: | :---  |
| 向量 | `tf::Vector3` |
| 点 | `tf::Point` |
| 四元数 | `tf::Quaternion` |
| 3*3矩阵（旋转矩阵） | `tf::Matrix3x3`|
| 位姿 | `tf::pose` |
| 变换 | `tf::Transform` |
| 带时间戳的以上类型 | `tf::Stamped<T>` |
| 带时间戳的变换 | `tf::StampedTransform` |

- 这里说的`tf::StampedTransform`（带时间戳的变换）和`TransformStampde.msg`数据类型完全不同，`tf::StampedTransform`是c++的一个类，并不像`TransformStampde.msg`是ROS中的消息的概念，因此前者具有局限性，只能在c++中使用，而后者是依赖于ROS系统的一个数据结构，与语言无关，可以在C++、Python、Java等其他各种语言中使用该消息

## 数据转换

tf中与数据转化的数据都类型都包含在`#include<tf/tf.h>`头文件中

| 函数名称 | 函数功能 |
| :---  | :---  |
| `tfScalar tf::Vector3::length()` | 计算向量的模 |
| `tfScalar tf::tfDot(const Vector3 &v1, const Vector3 &v2)` | 计算两个向量的点积 |
| `tfScalar tf::tfAngle(const tf::Vector3 &v1, const tf::Vector3 &v2)` | 计算两个向量的夹角 |
| `tfScalar tf::tfDistance2(const tf::Vector3 &v1, const tf::Vector3 &v2)` | 计算两个向量的距离 |
| `tf::Vector3 tf::tfCross(const tf::Vector3 &v1, const tf::Vector3 &v2)` | 计算两个向量的乘积 |
| `tf::Vector3 &tf::Vector3::normalize()` | 求与已知向量同方向的单位向量 |
| `void tf::Quaternion::setRPY(const tfScalar& yaw, const stScalar &pitch, const tfScalar &roll)` | 由欧拉角计算四元数 |
| `tf::Vector3 tf::Quaternion::getAxis()` | 由四元数得到旋转轴 |
| `void tf::Quaternion::setRotation(const tf::Vector3 &axis, const tfScalar &angle)` | 已知旋转轴和旋转角估计四元数 |
| `void tf::Matrix3x3::setRotation(const tf::Quaternion &q)` | 通过四元数得到旋转矩阵 |
| `void tf::Matrix3x3::getEulerYPR(tfScalar &yaw, tfScalar &pitch, tfScalar &roll)` | 由旋转矩阵求欧拉角 |

- `tfScalar`为标量，通过`typedef`定义为`double`
- `tf`为命名空间，`Vector3`，`Quaternion`和`Matrix3x3`是三个类，上述函数有成员函数、有内普通内联函数

Demo

```cpp
#include <ros/ros.h>
#include <tf/tf.h>
int main(int argc, char **argv)
{
    ros::init(argc, argv, "coordinate_transformation");
    ros::NodeHandle node;


    std::cout << "------定义空间点和空间向量------" << std::endl;
    tf::Vector3 v1(1, 1, 1);
    tf::Vector3 v2(1, 0, 1);

    std::cout << "向量v1:"
              << "(" << v1[0] << "," << v1[1] << "," << v1[2] << "),";
    std::cout << "向量v2:"
              << "(" << v2[0] << "," << v2[1] << "," << v2[2] << ")" << std::endl;
    std::cout << "两个向量的点积：" << tfDot(v1, v2) << std::endl;

    std::cout << "向量v2的模值:" << v2.length() << std::endl;

    tf::Vector3 v3;
    v3 = v2.normalize();
    std::cout << "与向量v2的同方向的单位向量v3:"
              << "(" << v3[0] << "," << v3[1] << "," << v3[2] << ")" << std::endl;

    std::cout << "两个向量的夹角(弧度):" << tfAngle(v1, v2) << std::endl;

    std::cout << "两个向量的距离:" << tfDistance2(v1, v2) << std::endl;

    tf::Vector3 v4;
    v4 = tfCross(v1, v2);
    std::cout << "两个向量的乘积v4:"
              << "(" << v4[0] << "," << v4[1] << "," << v4[2] << ")" << std::endl;


    std::cout << "------定义四元数------" << std::endl;
    tfScalar yaw, pitch, roll;
    yaw = 0;
    pitch = 0;
    roll = 0;
    std::cout << "欧拉角rpy(" << roll << "," << pitch << "," << yaw << ")";
    tf::Quaternion q;
    q.setRPY(yaw, pitch, roll);
    std::cout << "，转化到四元数q:"
              << "(" << q[3] << "," << q[0] << "," << q[1] << "," << q[2] << ")" << std::endl;

    tf::Vector3 v5;
    v5 = q.getAxis();
    std::cout << "四元数q的旋转轴v5"
              << "(" << v5[0] << "," << v5[1] << "," << v5[2] << ")" << std::endl;

    tf::Quaternion q2;
    q2.setRotation(v5, 1.570796);
    std::cout << "旋转轴v5和旋转角度90度，转化到四元数q2:"
              << "(" << q2[3] << "," << q2[0] << "," << q2[1] << "," << q2[2] << ")" << std::endl;


    std::cout << "------定义旋转矩阵------" << std::endl;
    tf::Matrix3x3 Matrix;
    tf::Vector3 v6, v7, v8;

    Matrix.setRotation(q2);
    v6 = Matrix[0];
    v7 = Matrix[1];
    v8 = Matrix[2];
    std::cout << "四元数q2对应的旋转矩阵M:" << v6[0] << "," << v6[1] << "," << v6[2] << std::endl;
    std::cout << "                       " << v7[0] << "," << v7[1] << "," << v7[2] << std::endl;
    std::cout << "                       " << v8[0] << "," << v8[1] << "," << v8[2] << std::endl;

    tfScalar m_yaw, m_pitch, m_roll;
    Matrix.getEulerYPR(m_yaw, m_pitch, m_roll);
    std::cout << "由旋转矩阵M,得到欧拉角rpy(" << m_roll << "," << m_pitch << "," << m_yaw << ")";
    return 0;
};
```

欧拉角转换为四元数demo

```cpp
#include <ros/ros.h>
#include <tf/tf.h>
int main(int argc, char **argv)
{
    ros::init(argc, argv, "Euler2Quaternion");
    ros::NodeHandle node;

    geometry_msgs::Quaternion q;
    double roll, pitch, yaw;

    while (ros::ok())
    {
        //输入一个相对原点的位置
        std::cout << "输入的欧拉角：roll,pitch,yaw:";
        std::cin >> roll >> pitch >> yaw;
        //输入欧拉角，转化成四元数在终端输出
        q = tf::createQuaternionMsgFromRollPitchYaw(roll, pitch, yaw);
        //ROS_INFO("输出的四元数为：w=%d,x=%d,y=%d,z=%d"，q.w,q.x,q.y,q.z);
        std::cout << "输出的四元数为：w=" << q.w << ",x=" << q.x << ",y=" << q.y << ",z=" << q.z << std::endl;
        ros::spinOnce();
    }
    return 0;
};
```

四元数转换为欧拉角demo

```cpp
#include <ros/ros.h>
#include "nav_msgs/Odometry.h"
#include <tf/tf.h>
int main(int argc, char **argv)
{
    ros::init(argc, argv, "Quaternion2Euler");
    ros::NodeHandle node;

    nav_msgs::Odometry position;
    tf::Quaternion RQ2;
    double roll, pitch, yaw;

    while (ros::ok())
    {
        //输入一个相对原点的位置
        std::cout << "输入的四元数：w,x,y,z:";
        std::cin >> position.pose.pose.orientation.w >> position.pose.pose.orientation.x >> position.pose.pose.orientation.y >> position.pose.pose.orientation.z;
        //输入四元数，转化成欧拉角
        tf::quaternionMsgToTF(position.pose.pose.orientation, RQ2);
        // tf::Vector3 m_vector3; 方法2
        // m_vector3=RQ2.getAxis();
        tf::Matrix3x3(RQ2).getRPY(roll, pitch, yaw);
        std::cout << "输出的欧拉角为：roll=" << roll << ",pitch=" << pitch << ",yaw=" << yaw << std::endl;
        //std::cout<<"输出欧拉角为：roll="<<m_vector3[0]<<",pitch="<<m_vector3[1]<<",yaw="<<m_vector3[2]<<std::endl;
        ros::spinOnce();
    }
    return 0;
};
```
