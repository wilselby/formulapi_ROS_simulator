# FormulaPi ROS Simulator

This project focused on developing a simulation environment in ROS and Gazebo to test and develop software for the FormulaPi competition. FormulaPi is an autonomous vehicle racing competition hosted by PiBorg using Yetiborg vehicles. This develop environment is meant to assist ForumulaPi competitors in developing more advanced image processing and vehicle control algorithms for incorporation into their race code. 

This project begins with installing the Clearpath Robotics Husky ground vehicles packages. Once simulation and control of the Husky vehicle is successful, the Husky model is modified to better represent the FormulaPi Yetiborg robot. Next, the FormulaPi track is imported from Sketchup into Gazebo. Finally, the FormulaPi image processing functions are leveraged to provide basic control of the simulated Yetiborg. A simple lane following algorithm is implemented and simulated.

Detailed instructions and images describing the installation and utilization of this software is available at https://www.wilselby.com/research/autonomous-vehicles/formulapi-autonomous-vehicle-racing/formulapi-ros-simulation-environment/

This software package contains several modules, described below:

formulapi_gazebo - Launch files and world files required for Gazebo simulation
formulapi_sitl - Package used to integrate FormulaPi functions into the ROS environment 
yetiborg_control - Package to initialize the Yetiborg control systems
yetiborg_description - Files to support the Yetiborg model required for Gazebo simulation


## Getting Started

This section assumes the reader is already comfortable working with ROS, Gazebo, and the catkin build environment. ROS (Robot Operating System) provides libraries and tools to help software developers create robot applications. It provides hardware abstraction, device drivers, libraries, visualizers, message-passing, package management, and more. Along with ROS, Gazebo is used for simulation. A well-designed simulator makes it possible to rapidly test algorithms, design robots, and perform regression testing using realistic scenarios. Gazebo offers the ability to accurately and efficiently simulate populations of robots in complex indoor and outdoor environments.


This first section installs the Clearpath Husky packages and a few additional dependencies and compiled packages required for the simulation.

### Install Clearpath Husky Packages

```
$ cd ~/catkin_ws/src
$ git clone https://github.com/husky/husky.git
$ git clone https://github.com/husky/husky_simulator.git
$ git clone https://github.com/husky/husky_desktop.git
$ cd ~/catkin_ws/
$ catkin_make
```

Get some additional dependencies

```
$ sudo apt-get install ros-indigo-lms1xx
$ sudo apt-get install ros-indigo-yocs-cmd-vel-mux
$ sudo apt-get install ros-indigo-gazebo-ros-control
$ sudo apt-get install ros-indigo-hector-gazebo
$ sudo apt-get install ros-indigo-teleop-twist-joy 
$ sudo apt-get install ros-indigo-robot_localization
$ sudo apt-get install ros-indigo-interactive-marker-twist-server
$ sudo apt-get install ros-indigo-joint-state-controller
$ sudo apt-get install ros-indigo-diff-drive-controller
$ sudo apt-get install ros-indigo-dwa-local-planner
$ sudo apt-get install ros-indigo-rviz-imu-plugin
$ sudo apt-get install ros-indigo-vision-opencv

```


## Simulate the Husky model

To ensure the Husky model and the basic dependencies are installed correctly, launch the Husky model in an empty Gazebo world.

```
$ roslaunch husky_gazebo husky_empty_world.launch
```

Next, test the joystick control by running the following commands in separate windows. The robot can be maneuvered using the joystick to change its direction. The user can alternate between two speeds by holding either the “A” or “B” buttons on an Xbox controller.

```
$ roslaunch husky_control teleop.launch
$ roslaunch husky_gazebo husky_empty_world.launch
```


Finally, test the path planning with a basic move_base setup but no mapping or localization. Run these commands in separate windows.
```
$ roslaunch husky_gazebo husky_playpen.launch
$ roslaunch husky_viz view_robot.launch
$ roslaunch husky_navigation move_base_mapless_demo.launch
```

In Rviz, ensure the Fixed Frame parameter is set to “odom” and the visualizers in the Navigation group are enabled. You can use the 2D Nav Goal tool in the top toolbar to select a movement goal in the visualizer. The Husky robot should automatically navigate to the goal position and orientation. An example of the Rviz and Gazebo visualizations is shown below. The green arrow depicts the goal, the yellow lines represent the LIDAR output, and the blue line behind the Husky represents the previously navigated path.


## Simulate the modified Yetiborg model
After the Husky model is successfully running and integrated with ROS and Gazebo, we need to modify the model parameters to more closely resemble the Yetiborg robot. Modifying these parameters will improve the accuracy of the simulation and help ensure that the FormulaPi software performs as well in the real-world races as it does in the simulations. The formulapi_ROS_simulator package contains modified versions of the Husky simulation files and serves as the foundation for the development of the FormulaPi simulation environment.

To test everything, we can visualize the Yetiborg model in an empty Gazebo world. To do this, run the following command.
Basic Yetiborm Model Gazebo Simulation

```
$ roslaunch formulapi_gazebo yetiborg_empty_world.launch
```

## Simulate the FormulaPi Track
The PiBorg team designed and built the FormulaPi track at their offices. The track is divided up into 6 lanes alternating between red, blue, and green. The track consists of 5 turns with a lap length of around 23 meters. 

To recreate the track in the Gazebo simulation environment, the track was drawn in Sketchup. The resulting model can be downloaded from the Sketchup Warehouse:
https://3dwarehouse.sketchup.com/model.html?id=918c85e8-dd05-452b-90c7-86c4885369d3

In order to import the track model into Gazebo, it needs to be saved as a .dae file. Next, a world file is developed. To test the model, the track can be visualized in Gazebo by running the following commands.

```
$ roscd formulapi_gazebo
$ cd worlds
$ gazebo formulapi_track1.world
```

### Putting it all together
The file formulapi_track1.launch loads the track and spawns the Yetiborg model. It can be launched with the following command. 
The resulting Gazebo simulation depicts the FormulaPi track with the Yetiborg model simulated on it.

```
$ roslaunch formulapi_gazebo formulapi_track1.launch
```

## FormulaPi Software Integration
The PiBorg team provides all competitors with a common code base. This software contains the minimum amount of functions needed to process images and control the Yetiborg. The software is only available to competitors and is hosted privately on SourceForge. The intent is to keep the software simple enough for entry-level competitors but to allow flexibility for more advanced competitors to modify the software to improve performance.

For the purposes of developing this simulation environment, the FormulaPi code was placed into a separate ROS package outside of the formulapi_ROS_simulator package. This allows those with access to the FormulaPi software to integrate that code base into this simulator while enabling the development of supplemental image processing or vehicle control software to be developed. This software can then be integrated back into the FormulaPi software if desired.

While the FormulaPi package is intended to remain private to those participating in the competition, the formulapi_sitl package was developed in the formulapi_ROS_simulator that relies on FormulaPi functions to demonstrate basic line following and autonomous control with the FormulaPi functions.

The file sitl_file.py creates a ROS node which receives the simulated images from the camera, calls the FormulaPi ImageProcessor.py functions to estimate the Yetiborg’s position, and computes a velocity command to send to the Yetiborg to keep it at the desired track position.

The simulation is run using the following command.

```
$ roslaunch formulapi_gazebo linefollow_cam_formulapi_track1.launch
```

Once the simulation is run, the Yetiborg will move forward, maintaining the desired track position around the track while the computer vision output is displayed. An example screenshot is shown below.


## Authors

* **Wil Selby** - *Initial work* - 


## License

This project is licensed under the MIT License

## Acknowledgments and References
http://wiki.ros.org/indigo/Installation/Ubuntu
http://wiki.ros.org/Robots/Husky
http://wiki.ros.org/husky_navigation
http://wiki.ros.org/husky_gazebo/Tutorials/Simulating%20Husky
http://wiki.ros.org/husky_bringup/Tutorials/Customize%20Husky%20Configuration
http://answers.ros.org/question/203264/importing-python-module-from-another-ros-package-hydro/
http://wiki.ros.org/cv_bridge/Tutorials/
http://wiki.ros.org/ROS/Tutorials/WritingPublisherSubscriber
http://wiki.ros.org/husky_control/Tutorials/Interfacing%20with%20Husky
http://wiki.ros.org/diff_drive_controller








