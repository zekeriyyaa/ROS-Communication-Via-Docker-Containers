# ROS-Communication-Via-Docker-Containers

The purpose of this project is establish a ROS(robot operating system) communication architecture via independent docker containers. Basically, ROS communication consists of three components namely master, publisher and subscriber nodes. All communication goes through the master. 
 
<p align="center" width="100%">
    <img width="50%" src="https://github.com/zekeriyyaa/ROS-Communication-Via-Docker-Containers/blob/master/rosArchitecture.png"> 
</p>

So, there are three standalone docker containers that represent these ROS nodes: 
- **Master**  : All communication takes place through him.
- **Talker**  : Broadcasts a message over the ROS network.
- **Listener**: Listens to messages broadcast from the network.

ROS is intended to be accessed and used through a graphical interface. Therefore, [VNC](https://www.realvnc.com/en/connect/download/viewer/) was preferred for remote control over Firefox's interface. The ROS wiki already specified needed configuration for use VNC with ROS in [here](http://wiki.ros.org/docker/Tutorials/GUI).

Although there are official ROS image on docker hub, [vnc-ros-kinetic-full](https://hub.docker.com/r/ct2034/vnc-ros-kinetic-full/) is used on this project. The reasoning behind this choice is that the official ROS image has only the core of the real ROS, not the additional features. **vnc-ros-kinetic-full** already allows us to access and control ROS via the Firefox interface via VNC.

In order to communicate ROS nodes, a master must be identified and advertised to other nodes. The given docker-compose.yaml already has all the required configurations. VNC communication is implemented on different ports for each ROS node as mentioned above. And the ROS communication port is also set by default.

The docker-compose file includes given configurations: 
```
version: '2'
 
services:
  master:
    image: ct2034/vnc-ros-kinetic-full
    ports:
      - "6080:80"
      - '11311:11311'
    container_name: master
    hostname: master
    environment:
      - "ROS_HOSTNAME=master"
      - "ROS_MASTER_URI=http://localhost:11311"
    command: roscore

  talker:
    image: ct2034/vnc-ros-kinetic-full
    ports:
      - '6081:80'
    container_name: talker
    hostname: talker
    environment:
      - ROS_HOSTNAME=talker
      - ROS_MASTER_URI=http://master:11311
          
  listener:
    image: ct2034/vnc-ros-kinetic-full
    ports:
      - '6082:80'
    container_name: listener
    hostname: listener
    environment:
      - ROS_HOSTNAME=listener
      - ROS_MASTER_URI=http://master:11311
```
### Running on docker-compose.yaml 
To create docker-containers: 
```
sudo docker-compose up
or
sudo docker-compose -f filename up
```
After the containers are ready, you can reach ROS desktop over Firefox interface using given url:
```
master   : 127.0.0.1:6080
talker   : 127.0.0.1:6081
listener : 127.0.0.1:6082
```
![](https://github.com/zekeriyyaa/ROS-Communication-Via-Docker-Containers/blob/master/ros_kinetic_vnc.jpg)
In order to test whether your connection is establish, use given commands on your ROS containers terminals:
```
master   : roscore
talker   : rostopic pub /test std_msgs/String "hello world" -r 1
listener : rostopic echo /test
```
In this case, talker node broadcasts a "hello world" message per second and listener subscribes it. The listener terminal is like given below:
![](https://github.com/zekeriyyaa/ROS-Communication-Via-Docker-Containers/blob/master/listener.jpg)

There is another case study to make sure your connection is well done. The talker is used to create the turtlesim node while the listener is used to control it by using arrow keys on your keyboard:
```
master   : roscore
talker   : rosrun turtlesim turtlesim_node
listener : rosrun turtlesim turtle_teleop_key
```
![](https://github.com/zekeriyyaa/ROS-Communication-Via-Docker-Containers/blob/master/turtlesim_app.jpg)

