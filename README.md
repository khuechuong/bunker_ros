# ROS Packages for Bunker Mobile Robot

## Packages

This repository contains minimal packages to control the bunker robot using ROS.

* bunker_bringup: launch and configuration files to start ROS nodes
* bunker_base: a ROS wrapper around [ugv_sdk](https://github.com/agilexrobotics/ugv_sdk) to monitor and control the bunker robot
* bunker_msgs: bunker related message definitions

## Supported Hardware

* bunker
* bunker_mini
* bunker pro

### Update the packages for your customized robot

**Additional sensors**

It's likely that you may want to add additional sensors to the bunker mobile platform, such as a Lidar for navigation. In such cases, a new ".xacro" file needs to be created to describe the relative pose of the new sensor with respect to the robot base, so that the sensor frame can be reflected in the robot tf tree.

A [sample](bunker_description/sample/bunker_v2_nav.xacro) ".xacro" file is present in this repository, in which the base ".xacro" file of an empty bunker platform is first included, and then additional links are defined.

The nodes in this ROS package are made to handle only the control of the bunker base and publishing of the status. Additional nodes may need to be created by the user to handle the sensors.

**Alternative odometry calculation**

By default the bunker_base package will publish odometry message to topic "/odom". In case you want to use a different approach to calculate the odometry, for example estimating the position together with an IMU, you could rename the default odometry topic to be something else.

```
$ bunker_bringup bunker_minimal.launch odom_topic_name:="<custom_name>"
```

## Communication interface setup

Please refer to the [README](https://github.com/agilexrobotics/ugv_sdk_sdk#hardware-interface) of "ugv_sdk" package for setup of communication interfaces.

#### Note on CAN interface on Nvidia Jetson Platforms

Nvidia Jeston TX2/Xavier/XavierNX have CAN controller(s) integrated in the main SOC. If you're using a dev kit, you need to add a CAN transceiver for proper CAN communication. 

## Basic usage of the ROS packages

1. Install dependent libraries

    ```
    $ sudo apt install -y ros-$ROS_DISTRO-teleop-twist-keyboard
    ```

2. Clone the packages into your catkin workspace and compile

    (the following instructions assume your catkin workspace is at: ~/catkin_ws/src)

    ```
    $ cd ~/catkin_ws/src
    $ git clone  https://github.com/agilexrobotics/ugv_sdk.git
    $ git clone https://github.com/agilexrobotics/bunker_ros.git
    $ cd ..
    $ catkin_make
    ```

3. Setup CAN-To-USB adapter

* Enable gs_usb kernel module(If you have already added this module, you do not need to add it)
    ```
    $ sudo modprobe gs_usb
    ```
    
* first time use bunker-ros package
   ```
   $ rosrun bunker_bringup setup_can2usb.bash
   ```
   
* if not the first time use bunker-ros package(Run this command every time you turn off the power) 
   ```
   $ rosrun bunker_bringup bringup_can2usb.bash
   ```
   
* Testing command
    ```
    # receiving data from can0
    $ candump can0
    ```
---------------------------------------------------------------------------------

**Alternatives to 3:**
1) Add into bunker_robot_base.launch:
```
<node name="setup_can_interface" pkg="bunker_bringup" type="bringup_can2usb.bash" output="screen" />
```
* The ROS environment is set up to handle password prompts, or configure sudo to allow these commands to execute without a password. This can be done by editing the sudoers file (using sudo visudo) like so:
  ```
    $ sudo visudo
  ```
* And adding a line for specific commands to be run without a password:
  ```
    <username> ALL=(ALL) NOPASSWD: /sbin/modprobe, /sbin/ip link set can0 up type can bitrate 500000
  ```
OR

2) Run the shellscript:
```
$ sudo ./bunker_pro.sh
```
*Note*: modify bunker_pro.sh:
```
# Replace '/opt/ros/noetic' with your actual ROS distribution path.
source /opt/ros/noetic/setup.bash
# Replace '/home/ara/bunker_ws' with your actual ROS ws path.
source /home/ara/bunker_ws/devel/setup.bash
```
---------------------------------------------------------------------------------

4. Launch ROS nodes

* Start the base node for the real robot

    ```
    $ roslaunch bunker_bringup bunker_robot_base.launch
    ```

    The [bunker_bringup/bunker_robot_base.launch](bunker_bringup/launch/bunker_robot_base.launch) has 4 parameters:

    - port_name: specifies the port used to communicate with the robot, default = "can0"

    - odom_topic_name: sets the name of the topic which calculated odometry is published to, defaults = "odom"



* Start the keyboard tele-op node

    ```
    $ roslaunch bunker_bringup bunker_teleop_keyboard.launch
    ```

    **SAFETY PRECAUSION**: 

    The default command values of the keyboard teleop node are high, make sure you decrease the speed commands before starting to control the robot with your keyboard! Have your remote controller ready to take over the control whenever necessary. 
