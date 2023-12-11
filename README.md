# ENPM808_ind_study

## Prerequisites

* Relatively new Nvidia Drivers, 515* should work
* Decently powerful GPU. Nvidia 2080ti preferred.
* Nvidia CUDA installed on your local system: [https://developer.nvidia.com/cuda-11-8-0-download-archive](Download Nvidia CUDA)
* Nvidia container toolkit: [https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html](Install Guide for Nvidia Container Toolkit)

## Setting up carla sim with docker

* To utilize the latest ROS 2 packages, the [https://carla.readthedocs.io/projects/ros-bridge/en/latest/](ROS 2 bridge) provided by Carla sim has to be utilized. The ROS 2 bridge captures the data collected by sensors in the ego vehicle in Carla sim and publishes them over the ROS 2 network with appropriate message types and topics. This bridge has to be built on top of ROS 2 foxy and Ubuntu 20. 

* To ensure a uniform development environment and compatibility across different devices with different hardware and software configurations, a docker image was built. The image uses a base ROS2 foxy image, on top of which Carla sim is installed along with the necessary packages consisting of ROS2 bridge. Other ROS2 packages for handling messages and sensor message conversions were also added. To ensure optimal performance of Carla sim along with display rendering, necessary xdg and libgl packages were installed. Nvidia driver capabilities were also configured to allow appropriate hardware acceleration while running graphics intensive applications. 

* The carla with ROS 2 bridge and Nvidia gpu support has been made publicly available on docker hub: [https://hub.docker.com/repository/docker/muditsingal98/carla_ros2/general](Carla Sim with ROS bridge docker image).


## Installing and setting up docker

To be added

## Installing and setting up rocker

To be added

## Resolving some common issues

To be added
