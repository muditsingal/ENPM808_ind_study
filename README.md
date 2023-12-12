# ENPM808 Independent Study

## Prerequisites

* Relatively new Nvidia Drivers, 515* should work
* Decently powerful GPU. Nvidia 2080ti preferred.
* Nvidia CUDA installed on your local system: )[Download Nvidia CUDA](https://developer.nvidia.com/cuda-11-8-0-download-archive)
* Nvidia container toolkit: [Install Guide for Nvidia Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)

## Setting up carla sim with docker

* To utilize the latest ROS 2 packages, the [ROS 2 bridge](https://carla.readthedocs.io/projects/ros-bridge/en/latest/) provided by Carla sim has to be utilized. The ROS 2 bridge captures the data collected by sensors in the ego vehicle in Carla sim and publishes them over the ROS 2 network with appropriate message types and topics. This bridge has to be built on top of ROS 2 foxy and Ubuntu 20. 

* To ensure a uniform development environment and compatibility across different devices with different hardware and software configurations, a docker image was built. The image uses a base ROS2 foxy image, on top of which Carla sim is installed along with the necessary packages consisting of ROS2 bridge. Other ROS2 packages for handling messages and sensor message conversions were also added. To ensure optimal performance of Carla sim along with display rendering, necessary xdg and libgl packages were installed. Nvidia driver capabilities were also configured to allow appropriate hardware acceleration while running graphics intensive applications. 

* The carla with ROS 2 bridge and Nvidia gpu support has been made publicly available on docker hub: [Carla Sim with ROS bridge docker image](https://hub.docker.com/repository/docker/muditsingal98/carla_ros2/general).


## Installing and setting up docker

*Install Docker using convenient script*

`curl -fsSL https://get.docker.com -o g1et-docker.sh` <br>
`sudo sh ./get-docker.sh --dry-run`

*Add docker to sudo user access group:*

`sudo groupadd docker` <br>
`sudo gpasswd -a $USER docker` <br>
`newgrp docker` <br>

_Reboot your system for changes to take effect_

*Verify installation*

`docker --version`

## Installing and setting up rocker

> Reference: https://github.com/osrf/rocker#installation

* Add the ROS 2 apt repository to your system. First authorize our GPG key with apt:

```
sudo apt update && sudo apt install curl gnupg lsb-release
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
```

* Then add the repository to your sources list:

```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(source /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```

```
sudo apt update
sudo apt-get install python3-rocker
```

## If you are getting errors while installing rocker natively, try using a virtual environment

*Follow the below commands:*

```
python3.7 -m virtualenv rockervenv
source rockervenv/bin/activate
pip install --upgrade python3-rocker

```

*To verify working of rocker:*

_with the virtual environment turned on:_
`rocker --help`

Now, whenever your virtual env is activated you should be able to use rocker to launch gpu accelerated and port forwarded containers.

## Installing Nvidia CUDA on your system:

For CUDA 11.7, follow the instructions given in:
[Install CUDA 11.7 for local system - linux](https://developer.nvidia.com/cuda-11-7-0-download-archive?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=20.04&target_type=deb_local)

For other versions of CUDA, you can just search on google: `cuda x.y download` and you'll get the respective download link.

Reboot your system once setup is completed.

Verify download by running: `nvidia-smi`.

## Setting up nvcc on your system:

Add the below 2 lines in .bashrc file in home directory (since I installed cuda 11.7, X=11, Y=7):
```
export PATH="/usr/local/cuda-X.Y/bin:$PATH"
export LD_LIBRARY_PATH="/usr/local/cuda-X.Y/lib64:$LD_LIBRARY_PATH"
```

> Replace cuda-X.Y with the version of cuda you installed

* Now `nvcc –version` should work and return the CUDA version specified in .bashrc file*


## Instal nvidia container toolkit in ubuntu system using the below apt commands:

Follow the [Nvidia container toolkit installation guide](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) for all instructions. Below are the instructions that I used and that worked for me.

```
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list \
  && \
    sudo apt-get update
```

```
sudo apt-get install -y nvidia-container-toolkit
```

*Configure docker for nvidia container toolkit*

```
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

*Check working of GPU with docker containers using:*

```
sudo docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi
```

## To run the Carla sim with ros2 bridge in GPU accelerated container use the command:

```
docker run --rm --gpus all --net=host -it --privileged --pid=host --runtime=nvidia --env="DISPLAY" --env="LOCAL_USER_ID=1000" --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" --device=/dev/video0 --device=/dev/video1 --name="carla_ct" muditsingal98/carla_ros2:v2.0 bash
```

*Inside the container*
```
cd /opt/carla-simulator
./CarlaUE4.sh -preferNvidia -quality-level=Medium
```

*Open another terminal to the container and start ros2 bridge*
```
docker exec -it carla_ct bash
source /opt/ros/foxy/setup.bash
source install/setup.bash
ros2 launch carla_ros_bridge carla_ros_bridge_with_example_ego_vehicle.launch.py  
```


## To setup Autoware using docker follow the instructions in the below link:

[Autoware Docker Installation](https://autowarefoundation.github.io/autoware-documentation/main/installation/autoware/docker-installation-devel/)


## To install multiple versions of CUDA:

However, `nvidia-smi` will always return the latest CUDA supported by your Nvidia card.

[Installing multiple versions of cuda in a machine - Gamma Lab](https://wiki.cs.umd.edu/gamma/view/Installing_multiple_versions_of_cuda_in_a_machine#:~:text=These%20are%20step%2Dby%2Dstep,to%20more%20than%20two%20versions.&text=Choose%20the%20appropriate%20version%20from,we%20will%20choose%20version%2010.0.)


## Pulling and running docker containers inside UMD nexus cluster:

* To request GPU resources:
`srun --pty --mem=32G --qos=default --gres=gpu:1 bash`

* To pull docker image using apptainer:
`apptainer pull docker://repo/image_name:tag`

* Run the docker image pulled and stored as a sif file:
`apptainer run image_name.sif`

## Resolving common Errors:


#### Resolving xdg error:

If while rendering something inside a docker container you get the below error:
```python
sh: 1: xdg-user-dir: not found
No protocol specified
error: XDG_RUNTIME_DIR not set in the environment.
No protocol specified
error: XDG_RUNTIME_DIR not set in the environment.
No protocol specified
error: XDG_RUNTIME_DIR not set in the environment.
```

To resolve the error exit the container and run:
```
xhost local:root
```

Then relaunch the container and try running carla sim.

#### Resolving Vulkan Driver (ICD) error:

Run the below command on both local system and inside the container: 

```
sudo apt install mesa-vulkan-drivers 
```

#### Resolving Xlib:  extension "NV-GLX" missing on display ":1" :

Install new drivers for nvidia gpu.


#### Error while launching carla sim inside the docker container - Refusing to run with the root privileges:


Do the following:

```
adduser xyz --disabled-password
sudo -u xyz ./Carla.sh -preferNvidia -quality-level=Medium
```


#### Another warning while running carla sim inside the container:
(Not an error. Just showing that issue with sound card, doesn’t matter and can be ignored)

```
ALSA lib pcm.c:2642:(snd_pcm_open_noupdate) Unknown PCM default
ALSA lib confmisc.c:767:(parse_card) cannot find card '0'
ALSA lib conf.c:4732:(_snd_config_evaluate) function snd_func_card_driver returned error: No such file or directory
ALSA lib confmisc.c:392:(snd_func_concat) error evaluating strings
ALSA lib conf.c:4732:(_snd_config_evaluate) function snd_func_concat returned error: No such file or directory
ALSA lib confmisc.c:1246:(snd_func_refer) error evaluating name
ALSA lib conf.c:4732:(_snd_config_evaluate) function snd_func_refer returned error: No such file or directory
ALSA lib conf.c:5220:(snd_config_expand) Evaluate error: No such file or directory
```

## Pushing Docker images on docker hub:

```
docker login (enter username and password of your docker hub account)
docker tag img_name username/new_img_name:v1.0
(please note that new_img_name is the one that will show up in your docker hub, it can be the same as the image name you have for your image in local system)
docker push username/new_img_name:v1.0
```
