# ROS工作空间

## 1. Catkin编译系统

源码需要编译、链接等步骤才能生成二进制可执行文件，对于 ROS 等大型项目而言，需要管理项目结构，进行自动化编译等。

Catkin 是 ROS 对 CMake 进行拓展之后的，CMake 能够生成`makefile`文件，进而能够控制编译的过程

Catkin 编译系统的层级结构如图

![catkin.jpg](./imgs/catkin.jpg)

### 1.1. Catkin编译流程

- 在工作空间`catkinws/src/`下进行递归查询每一个ROS包
- ROS包中有`package.xml`和`CMakeLists.txt`文件，则根据`CMakeLists.txt`生成相应的`makefile`文件
- 再 `make` 进行编译

相当于`catkin_make`是将cmake与make进行了合并操作，同时提高了跨项目依赖性

### 1.2. 使用catkin\_make进行编译

必须要在工作空间的目录下执行`catkin_make`，之后需要`source`命令更新环境变量，否则`rosrun`无法执行ros包

```bash
cd ~/catkinws
catkin_make
source ./devel/setup.bash
```

## 2. ROS工作空间结构

```text
─ build
│   ├── catkin
│   │   └── catkin_generated
│   │       └── version
│   │           └── package.cmake
│   ├──
......

│   ├── catkin_make.cache
│   ├── CMakeCache.txt
│   ├── CMakeFiles
│   │   ├──
......

├── devel
│   ├── env.sh
│   ├── lib
│   ├── setup.bash
│   ├── setup.sh
│   ├── _setup_util.py
│   └── setup.zsh
└── src
└── CMakeLists.txt -> /opt/ros/kinetic/share/catkin/cmake/toplevel.cmake
```

- `src/`用于存放各种`package`，其中同一类多个功能包可以存放在一个子目录下
- `build/`用于存放编译过程中的中间文件，缓存信息等
- `devel/`用于存放生成的目标文件，包括可执行文件，静态链接库，动态链接库等

`src/`下的源代码，经过编译生成中间文件，存在`build/`下，`build/`下的文件再经过链接等处理，最后生成可执行文件，库文件等，存放在`devel/`

`src/`下允许存在多个package，在ROS工作空间下的各个package可以在子目录下

![catkin_ws.jpg](./imgs/catkin_ws.jpg)
