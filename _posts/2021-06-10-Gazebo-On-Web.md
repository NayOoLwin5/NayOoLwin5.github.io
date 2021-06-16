---
layout: post
author: nayoolwin
title: Integration of ROS and Gazebo on Web Application
date: 2021-02-20
thumbnail: /assets/img/web-template.png
category: web development
summary: A web based template for Global Navigation simulation for robotics enthusiasts
keywords: ros and gazebo on web, gazebo web template, integration of ros and gazebo on web application, 
permalink: /blog/ros-gazebo-on-web/
---

# Contents 
- [A quick overview](#header1)
- [Project references](#header2)
- [Web structure](#header3)
- [Design Architecture](#header4)
    - [Tools used](#header5)
    - [Working principle on the server side](#header6)
       - [Launch and world files](#header7)
       - [Gazebo Models](#header8)
       - [HAL.py](#header9)
       - [map.py](#header10)
       - [gui.py](#header11)
       - [exercise.py](#header12)



# A quick overview {#header1} 

This project is my internship work in [JdeRobot open source organization](https://jderobot.github.io/RoboticsAcademy/) which develops a collection of exercises and challenges to learn robotics in a practical way. It includes exercises about drone programming, computer vision, mobile robots, autonomous cars, etc and all mainly based on [Gazebo simulator and ROS](https://www.ros.org/). The organization develops software(ROS and Gazebo based exercises) with a main focus on educational impact on users who are keen on learning and testing different robotics algorithms for various scenarios. 

By the time I joined the organization, they were primarily working on new release 2.3 which is migrating all the available exercises to new web-templates paradigm. I decided to take on one of those exercises which is **Global Navigation** and it was quiet challenging for the migration. And I studied their open sourced [repository](https://github.com/JdeRobot/RoboticsAcademy) structure to better understand the source code. 

# Project references {#header2}

1. [Global Navigation exercise webpage](http://jderobot.github.io/RoboticsAcademy/exercises/AutonomousCars/global_navigation/)
2. [Project source code](https://github.com/JdeRobot/RoboticsAcademy/tree/master/exercises/static/exercises/global_navigation/web-template)

# Web Structure {#header3}

As one can see the thumbnail image, it is a single page localhost web-template contains :-
- **[ACE editor](https://github.com/ajaxorg/ace)** for the user to code particular algorithm in python 
- **[Gzweb client ](http://gazebosim.org/gzweb.html) simulation window + [noVNC viewer](https://github.com/novnc/noVNC)** in which the robot acts upon user's algorithm 
- **[VNC Console](https://www.realvnc.com/en/connect/download/viewer/linux/)** for debugged informations
- **Control widgets** such as play/stop/reset the simulation 
- **Map image** with canvas robot position to locate it on the map

# Design Architecture {#header4}

## Tools used {#header5}
- **[ROS-melodic(LTS)](http://wiki.ros.org/melodic/Installation/Ubuntu)** - It is main framework used for simulation environment.
- **Two [websocket connections](https://www.geeksforgeeks.org/what-is-web-socket-and-how-it-is-different-from-the-http/)** : **GUI websocket** for sending sensor data published from gzserver to gzweb (server to client), etc and **Code websocket** for user queries such as change in the sensor data, interfaces, etc (client to server). 
- **Two python theads** : GUI thread for starting GUI websocket server and brain thread for Code websocket server along with their respective components. 
- **[Docker](https://www.docker.com/)** : We built a docker image contains all the dependencies for running the exercise and use it as server side to maintain minimal installation and set up a ready working environment for the user with a single command line. It is :- 
```bash
docker run -it -p 8080:8080 -p 2303:2303 -p 1905:1905 -p 8765:8765 -p 6080:6080 -p 1108:1108 jderobot/robotics-academy python3.8 manager.py
```
    - Port 8080 for Gzweb
    - Port 2303 for GUI websocket 
    - Port 1905 for Code websocket
    - Port 8765 for manager.py for setting up ROS environment for specific chosen exercise
    - Port 6080 for vnc viewer for Gazebo
    - Port 1108 for vnc viewer for console

## Working principle on server side {#header6}
### Launch and world files {#header7}
- It is suggested to run the launch file headless to be able to run Gzweb. Since the docker container handles a bundle of dependencies, running Gazebo GUI window inside the docker will consume too much CPU power.
- By launching it, only gzserver will be running behind the scene along with master node and other nodes.

### Gazebo Models {#header8}
- The gazebo models are taxi_holo_ROS and cityLarge that are come preinstalled with [JdeRobot academy software](http://jderobot.github.io/RoboticsAcademy/installation/). Since they come preinstalled, I simply got nothing to do with for using them. 
- And they are loaded from [CustomRobots repo](https://github.com/NayOoLwin5/CustomRobots/tree/melodic-devel/Taxi_navigator) by the [docker image](https://github.com/JdeRobot/RoboticsAcademy/blob/master/scripts/Dockerfile).

### HAL.py (hardware abstratcion layer) {#header9}
- From the master node, I take only two neccessary topics which are `/taxi_holo/odom` for the user to know robot position and `/taxi_holo/cmd_vel`
for the user to override the linear and angular velocities. 
- I subscribed to `/taxi_holo/odom` so that user gets updated robot position messages continously and published `/taxi_holo/cmd_vel` and setting up linear and angular velocities to zero so that user can override any values they want to the message.

### map.py {#header10}
- Map with robot location provided on the template to monitor the robot. 
- I extracted x,y coordinates and yaw (angle of rotation plane) from `/taxi_holo/odom` topic from `HAL class` and converted them from world coordinates to map coordinates and used them on canvas element. 
- Some components are needed to be translated from world coordinates to map coordinates and vice-versa. For those,  created built-in modules which do the function for the user. For all the built-in modules details, please refer [Ref 1](#header2)
 

### gui.py (GUI thread) {#header11}
 
- In this, the GUI thread will be started that runs GUI websocket server.
- The components in this thread are measuring the frequency of it, sending built-in modules functions called by the user on the socket such as robot position parsed by `MAP class`, user's debugged image which is GPP(Gradient Path Planning) and in CvMat form, 2D array points to show shortest path decided from path planning on the map, etc.

### exercise.py (main launching file)(brain thread) {#header12}
- Whenever user clicks widgets, the server gets the response from client and controls the simulations accordingly. 
- The user code is sent to this websocket and executed using `exec()` function. 
- All the accessible built-in APIs are created for the user to use dynamically on the editor in `exec()` func. With the built modules as globals and empty dictionary as locals.
- And the brain thread will be started when the user clicks `play` button and the code in the editor will be executed.




   
