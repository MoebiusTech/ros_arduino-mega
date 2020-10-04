目录:
一、ROS基础
ROS常用关键词
文件系统的概念
二、环境搭建
快速安装
分步安装
三、ROS初体验
ROS串口通信
主机topic通信
四、ROS分布式通信规范
节点通信原理
话题发布
话题订阅
五、ROS描述文件
六、启动小车底盘节点
七、激光雷达构建地图
八、SLAM自主导航
九、机器视觉巡线避障
十、深度机器视觉构建三维点云
一、ROS基础
ROS常用的关键词
1: message: 即消息．机器人需要传感器，传感器采集到的信息，即这儿的message. 假如我们的GPS采集到机器人位置消息，温度计采集到的温度等.　任何数据都能作为message.

2: topic: 假设我们有两个传感器，GPS和温度计．在ROS中我们得给采集到的消息取个名字用来区分不同的message，这就是topic了.　至于怎么取名字，后面的程序可见.

3: node: 节点. ROS中，通常来讲你写的c++或者python程序主函数所在的程序称为一个节点(并不准确，暂时这么理解).

4: package: 一个ROS package包含了你要完成的一个项目的所有文件.

5: workSpace: ROSworkspace，即ROS的工作空间．你当然需要ROS来完成很多不同的项目．你的ROS工作空间是用来存放很多不同package的．

6: publish, subscribe: ROS很大的一个作用就是传递message．为什么要传递消息呢? 打个比方你写了一个程序，用来获得GPS的讯息，写了另外一个程序，用来处理GPS的信息．这时你就是需要把采集到的信息传输到处理用的程序中．信息的传输在ROS中称为publish. 信息的订阅在ROS中称为subscribe. 好了，有了这几个概念，我们就可以开始撸第一个ROS程序了．(可能大佬们觉得还有好多没讲呢，没关系我们慢慢来．)

快速了解文件系统概念
Packages: 软件包，是ROS应用程序代码的组织单元，每个软件包都可以包含程序库、可执行文件、脚本或者其它手动创建的东西。

Manifest (package.xml): 清单，是对于'软件包'相关信息的描述,用于定义软件包相关元信息之间的依赖关系，这些信息包括版本、维护者和许可协议等。

Launch (file.launch): launch文件顾名思义就是启动文件，launch文件是描述一组节点及其话题重映射和参数的XML文件。launch允许同时启动多个节点

二、环境搭建
树莓派系统安装 Ubuntu mate 下载链接

这里使用ubuntu mate 18.04 raspberry版本，考虑到官方镜像Linux是基于Debian9发行的，使用ubuntu mate可以减少很多兼容性带来的麻烦

使用镜像烧录工具将系统拷入SD卡中，启动树莓派

这里分享了两种安装方式:
1- 快速安装：快速配置所有ROS系统以及所需软件 2- 分步安装：理解安装所需要的软件包和熟悉常用的Linux操作

快速安装:
创建工作区

cd ~/ 
mkdir catkin_ws/
cd catkin_ws
ROS安装器下载

在下载链接中选择保存->下载,然后吧下载好的.sh文件保存到刚创建好的catkin_ws文件夹下，在终端输入

chmod +x ros_install.sh
./ros_install.sh
此时开始进行安装程序，等待安装完成后就可以进入下一个环节

分步安装:
安装ROS主要步骤:

1.切换软件源
sudo sh -c 'echo "deb https://mirror.tuna.tsinghua.edu.cn/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
2.配置秘钥
sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
3.更新本地软件列表
sudo apt update
4.安装ROS
sudo apt install ros-melodic-ros-base
5. 安装完成后初始化ros
sudo rosdep init
rosdep update
6. 添加环境变量
echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
source ~/.bashrc
三、ROS初体验
本章目标: 体验ROS通信原理以及测试ROS配置是否完善
ROS串口通信
为什么要用串口通信:
ROS运行在树莓派或其他嵌入式系统平台上，而机器人控制器大部分由底层的单片机或控制器组成的，多数控制算法的PID、滤波器、传感器姿态角都是重复的矩阵迭代运算，占用CPU和内存。这个时候为了降低系统资源使用，就会使用串口来与底层的驱动器通信。这样ROS就可以去处理其他的工作。

通过ros与arduino串口通信:
Arduino端配置
在linux上安装ArduinoIDE (网上很多教程，这里不多赘述).
安装rosserial库: 工具->管理库->搜索rosserial->安装.
打开Helloworld例程: 文件->示例->RosserialAdruinoLibary->HelloWorld：

烧录到Arduino中，选择波特率为115200. 完成后测试ROS通信-->

1.打开串口权限
sudo chmod 777 /dev/ttyACM0   //这里我的串口号是ttyACM0
2.开启ros主节点
roscore
3.开启ros串口节点
rosrun rosserial_python serial_node.py /dev/ttyACM0
4.订阅节点消息
rostopic echo /chatter
这里终端在不停地打印：

---
data: "hello world!"
---
data: "hello world!"
---
data: "hello world!"
---
data: "hello world!"
到这里就说明串口通信已经成功了，下面开始自己尝试构建ROS通信的node和topic:

ROS主机topic通信:
1.创建workspace
ROS通信模式主要是分布式结构，通过节点的话题通信来传递消息和指令，首先需要创建一个Workspace来放置自己的package，便于管理和维护。

mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/
catkin_make
执行完成后，在catkin_ws目录下会生成3个文件夹/build、/devel、/src
这里一个工作空间就已经创建完了，后续的package都会存放在这个目录下面

2. 创建一个ROS package
使用命令: catkin_create_pkg 创建一个包

catkin_create_pkg topic_test std_msgs rospy roscpp
topic_test 就是我们需要创建的包名
std_msgs rospy roscpp 是包所需的依赖包，类似于C语言的#include "xxxxx.h"

3. 创建话题发布者(Publisher)和订阅者(Subscriber)
进入到我们新创建的包中，创建src源文件目录

roscd topic_test
roscd: ROS工具之一，在编译通过后，就可以通过roscd快速进入包所在目录

这时你的终端的目录应该会跳转至:

~/catkin_ws/src/topic_test/:
我们创建一个发布者pub_test.cpp文件,并写入测试代码

touch pub_test.cpp
gedit pub_test.cpp
将下面代码放到pub_test.cpp文件中

#include "ros/ros.h"
#include "std_msgs/String.h"

#include <sstream>

int main(int argc, char **argv)
{
  ros::init(argc, argv, "talker");
  ros::NodeHandle n;
  ros::Publisher chatter_pub = n.advertise<std_msgs::String>("chatter", 1000);
  ros::Rate loop_rate(10);
  
  int count = 0;
  while (ros::ok())
  {
    std_msgs::String msg;
    std::stringstream ss;
    ss << "hello world " << count;
    msg.data = ss.str();
    ROS_INFO("%s", msg.data.c_str());
    chatter_pub.publish(msg);
    ros::spinOnce();
    loop_rate.sleep();
    ++count;
  }
  return 0;
}
同样, 创建一个订阅者, 将代码拷贝到.cpp文件中.

touch sub_test.cpp
gedit sub_test.cpp
将下面代码放到sub_test.cpp文件中

#include "ros/ros.h"
#include "std_msgs/String.h"

void chatterCallback(const std_msgs::String::ConstPtr& msg)
{
  ROS_INFO("I heard: [%s]", msg->data.c_str());
}

int main(int argc, char **argv)
{
  ros::init(argc, argv, "listener");
  ros::NodeHandle n;
  ros::Subscriber sub = n.subscribe("chatter", 1000, chatterCallback);
  ros::spin();
  return 0;
}
关闭已经打开的编辑框，您在上一教程中使用过catkin_create_pkg，该教程为您创建了package.xml和CMakeLists.txt文件.
现在只需将以下几行添加到CMakeLists.txt的底部

add_executable(talker src/pub_test.cpp)
target_link_libraries(talker ${catkin_LIBRARIES})
add_dependencies(talker topic_test)

add_executable(listener src/sub_test.cpp)
target_link_libraries(listener ${catkin_LIBRARIES})
add_dependencies(listener topic_test)
进入catkin_ws目录, 重新编译生成

cd ~/catkin_ws
catkin_make
四、ROS分布式通信规范
ROS节点中通信方式有四种，话题、服务、参数服务器、动作库。每个通信方式都有自己的特点

话题通信原理
ROS所有节点间通信都是以话题、订阅的方式，当发布话题者创建一个话题时，订阅者可以通过订阅这个话题来接收消息

话题
发布者
订阅者
一、话题发布：
经过上一章的试验，我们大概对ROS通信有了一定的概念，接下来我们分析一下在Arduino中，ROS是如何发布话题的:
在Arduino中选择文件-示例-Rosserial Arduino Library -> Helloworld

//Rosserial Arduino Library -> Helloworld

#include <ros.h>
#include <std_msgs/String.h>

ros::NodeHandle  nh;

std_msgs::String str_msg;
ros::Publisher chatter("chatter", &str_msg);

char hello[13] = "hello world!";

void setup()
{
  nh.initNode();
  nh.advertise(chatter);
}

void loop()
{
  str_msg.data = hello;
  chatter.publish( &str_msg );
  nh.spinOnce();
  delay(1000);
}
包含ROS头文件,主要是一些ROS用到的库和消息类型
#include <ros.h>
#include <std_msgs/String.h> 
创建一个节点句柄对象,在spinOnce()中处理消息发布、订阅等
ros::NodeHandle  nh;
示例创建了一个chatter话题对象，引用publish可以发布话题。
std_msgs::String str_msg;
ros::Publisher chatter("chatter", &str_msg);
初始化ROS节点句柄，将创建的chatter话题注册到通信句柄中
nh.initNode();
nh.advertise(chatter);
通过chatter对象的publish实例将str_msg加入消息队列，通过spinOnce()将消息队列内的消息推送给串口
chatter.publish( &str_msg );
nh.spinOnce();
二、话题订阅:
在Arduino中选择文件-示例-Rosserial Arduino Library -> Blink

// Rosserial Arduino Library -> Blink

#include <ros.h>
#include <std_msgs/Empty.h>

ros::NodeHandle  nh;

void messageCb( const std_msgs::Empty& toggle_msg){
  digitalWrite(13, HIGH-digitalRead(13));   // blink the led
}

ros::Subscriber<std_msgs::Empty> sub("toggle_led", &messageCb );

void setup()
{ 
  pinMode(13, OUTPUT);
  nh.initNode();
  nh.subscribe(sub);
}

void loop()
{  
  nh.spinOnce();
  delay(1);
}
包含ROS头文件,主要是一些ROS用到的库和消息类型
#include <ros.h>
#include <std_msgs/Empty.h>
定义回调函数，当接收到订阅时调用函数，处理话题发出的消息
void messageCb( const std_msgs::Empty& toggle_msg)
{
    digitalWrite(13, HIGH-digitalRead(13));   // blink the led
}
ros::Subscriber<std_msgs::Empty> sub("toggle_led", message);
创建了一个toggle_led订阅对象，当接收到来自toggle_led话题发布的消息时，调用message函数。
ros::Subscriber<std_msgs::Empty> sub("toggle_led", &messageCb );
初始化节点，将订阅内容注册到节点句柄中
nh.initNode();
nh.subscribe(sub);
服务通信原理
TF树
TF是机器人处理姿态坐标的一个包，机器人的不同部位和

五、ROS描述文件
1. xml文件
常见的文件: package.xml(软件包清单) 该文件定义有关软件包的属性，例如软件包名称、版本号、作者、维护者及其依赖包。
标准格式: 每个package.xml文件都以<package>标记作为根标记

<package format =“ 2”>

</ package>
必要标签:
<name> -包的名称
<version> -软件包的版本号（必须为3个点分隔的整数）
<description> -包装内容的描述
<maintainer> -正在维护包裹的人员的姓名
<license> -用于发布代码的软件许可证(例如，GPL，BSD，ASL)

例如，这是一个虚构的软件包foo_core的软件包清单:

<package format="2">
  <name>foo_core</name>
  <version>1.2.4</version>
  <description>
  This package provides foo capability.
  </description>
  <maintainer email="ivana@osrf.org">Ivana Bildbotz</maintainer>
  <license>BSD</license>
</package>
软件包清单依赖项
构建依赖项: 指定构建此软件包所需的软件包。在构建时需要这些软件包中的任何文件时就是这种情况。在交叉编译方案中，构建依赖关系是针对目标体系结构的。
构建导出依赖项: 指定针对该软件包构建库所需的软件包。
执行依赖项: 指定运行该程序包中的代码所需的程序包。
测试依赖项: 仅为单元测试指定其他依赖项。他们绝不应该复制已经提到的任何依赖，即构建或运行依赖。
编译器依赖项: 指定此软件包自行构建所需的构建系统工具。通常，唯一需要的构建工具是柔韧性。在交叉编译方案中，构建工具依赖项是针对要在其上进行编译的体系结构的。
文档工具依赖项: 指定此包生成文档所需的文档工具。
2. param文件
3. launch文件
4. msg文件
六、启动小车底盘节点
经过了前面ArduinoSerial的实验，我们对ROS的通信原理应该有一定理解了，下面我们就开始一步一步的搭建与小车通信的桥梁

远程访问小车
我们在小车上跑ROS的时候，就没有办法再接个显示器、鼠标什么的了，这时候就需要通过远程访问来实现小车控制

常用的远程桌面连接方式: Teamviewer14 (远程桌面-只适用32位系统)、xrdp(图形界面远程连接)、SSH(无线终端连接)、VNC等。
选一种喜欢的方式即可，我们这里使用ubuntu上位机通过ssh连接小车

首先将小车树莓派连接到路由器WIFI，然后通过ifconfig查询已分配的IP地址。这里我的树莓派地址是: 192.168.0.104

在ubuntu中SSH连接小车：

//Master Terminal   1
mbs@master:~$ssh mbsrobot@192.168.0.104
连接成功后可以看到用户名已经变为树莓派的设备名了，接下来启动底盘节点:

//SSH@Respberry Terminal   1
mbsrobot@mbsrobot:~$ roslaunch mbsrobot bringup.launch
再打开一个终端，运行按键控制脚本控制小车:

//SSH@Respberry Terminal   2
mbs@master:~$ssh mbsrobot@192.168.0.104
mbsrobot@mbsrobot:~$ rosrun teleop_twist_keyboard teleop_twist_keyboard.py

Reading from the keyboard  and Publishing to Twist!
---------------------------
Moving around:
   u    i    o
   j    k    l
   m    ,    .

For Holonomic mode (strafing), hold down the shift key:
---------------------------
   U    I    O
   J    K    L
   M    <    >

t : up (+z)
b : down (-z)

anything else : stop

q/z : increase/decrease max speeds by 10%
w/x : increase/decrease only linear speed by 10%
e/c : increase/decrease only angular speed by 10%

CTRL-C to quit

currently:      speed 0.5       turn 1.0
在这个终端上按u、i、o、j、k、l、m、<、>就可以控制小车运动方向
t和b控制转弯的角度
q/z 、 w/x 、e/c 都是速度大小控制

控制原理分析: 在ROS通信中，小车可以看做一个单独的节点。主机发布cmd_vel话题，小车订阅cmd_vel话题并执行运动指令。官方给出了一种用于运动控制的消息类型: geometry_msgs/Twist, 这个消息的结构是这样的：

Type: geometry_msgs/Twist
geometry_msgs/Vector3 linear
    float64 x
    float64 y
    float64 z
geometry_msgs/Vector3 angular
    float64 x
    float64 y
    float64 z
创建Twist话题时，ROS会将消息结构体注册到控制器中, 节点通过串口将消息发给小车，话题内包含一个位移向量和一个转动向量。小车作为一个节点订阅cmd_vel话题，当小车订阅器收到master发来的cmd_vel后，将位移和转动角转换为四个电机的速度值，从而实现简单的运动控制。

七、激光雷达构建地图
激光雷达，是以发射激光束探测目标的位置、速度等特征量的雷达系统。我们小车上搭配的激光雷达是由激光测距模组和云台组成的，云台在转动的过程中激光测距模组测量某个位置的距离，在扫描一圈后就会形成一个表示地图的点云集合

在我们小车中开启雷达可以通过

//SSH@Respberry Terminal   3
mbs@master:~$ssh mbsrobot@192.168.0.104
mbsrobot@mbsrobot:~$ roslaunch mbsrobot lidar_slam.launch
再打开一个终端，开启按键控制节点

//SSH@Respberry Terminal   4
mbs@master:~$ssh mbsrobot@192.168.0.104
mbsrobot@mbsrobot:~$ rosrun teleop_twist_keyboard teleop_twist_keyboard.py
通过launch文件快速开启所需的雷达节点，在上位机中，使用RVIZ观察雷达构建的点云地图

//Master Terminal   2
mbs@master:~$ rosrun rviz rviz
在上位机选择对应的.rviz配置文件，就可以看到小车扫描出的地图了 通过按键控制小车运动，就可以将整个屋内的地图扫描出来，接下来将地图文件保存到参数文件内就可以了

八、SLAM自主导航
利用激光雷达进行自动导航
1、首先我们上电启动小车，然后接上雷达（tips:rpliadr A1雷达上电就会转, A2雷达上电后需要执行建图命令后才转），然后在主控端执行启动小车的命令：

roslaunch mbsrobot bringup.launch
2、主控端启动导航的命令,等待启动完：

roslaunch mbsrobot bringup.launch
3、在远程端打开可视频工具rviz，在远程端运行下面命令，然后把出现的窗口最大化，然后左上角，有一个File->OpenConfig，然后到catkin_ws/src/mbsrobot_project/mbsrobot/rviz这个路径下面打开navigate.rviz，等待窗口把数据加载完成，导航的第一步就是要初始化位姿，用“2D Pose Estimate”，在地图的可视区域拖一下，记住拖一下不是随便拖，是你机器人的实际位置，在地图上显示的这个区域拖一下，那个箭头带表方向，这个也很重要，这个意思就是你告诉机器人，你在地图上的定位是错误的，你应该在我给定的这个位置（tip：如果不想每次给定位姿，你可以在建图时，机器人的起始位置是在那里，你导航时一样，把机器人放到这个位置，就不用估计位姿），位姿校正完成后，我们就用“2D Nav Goal”发送目标，这个作用是告诉你，你现在去我指定的地方，也是在地图的可视化区域拖一下，然后小车此时自动规化路径，自己避开地图上的障碍物，到达目的地，而且是可以动态避障的，你在一定距离内出现在地图区域，它也可以重新规化路径，绕过你这个动态的障碍物：

rosrun rviz rviz
