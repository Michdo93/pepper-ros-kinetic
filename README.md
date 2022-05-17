# pepper-ros-kinetic
Setup guide for ROS Kinetic Kame and the pepper robot.

## Install ROS Kinetic

### On Ubuntu 16.04

```
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
sudo apt install curl # if you haven't already installed curl
curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -

sudo apt-get update
sudo apt-get install ros-kinetic-desktop-full

echo "source /opt/ros/kinetic/setup.bash" >> ~/.bashrc
source ~/.bashrc

sudo apt install python-rosdep python-rosinstall python-rosinstall-generator python-wstool build-essential
sudo apt install python-rosdep

sudo rosdep init
rosdep update --include-eol-distros
```

### From Source

`sudo nano /etc/apt/sources.list.d/ros-latest.list`:

```
deb http://packages.ros.org/ros/ubuntu xenial main
```

`sudo nano /etc/apt/sources.list.d/ros-latest.list.distUpgrade`:

```
deb http://packages.ros.org/ros/ubuntu xenial main
```


`sudo nano /etc/apt/sources.list.d/ros-latest.list.save`:

```
# deb http://packages.ros.org/ros/ubuntu xenial main # disabled on upgrade to focal
```

`sudo nano /etc/apt/sources.list.d/gazebo-stable.list`:

```
# deb http://packages.osrfoundation.org/gazebo/ubuntu-stable xenial main # disabled on upgrade to focal
```


`sudo nano /etc/apt/sources.list.d/gazebo-stable.list.distUpgrade`:

```
deb http://packages.osrfoundation.org/gazebo/ubuntu-stable xenial main
```


`sudo nano /etc/apt/sources.list.d/gazebo-stable.list.save`:

```
# deb http://packages.osrfoundation.org/gazebo/ubuntu-stable xenial main # disabled on upgrade to focal
```

Then run sudo `apt-get update`:

```
sudo apt-get update
```

You have to install `Python 2.7`:

```
sudo apt-add-repository universe
sudo apt update
sudo apt install python2.7 python2.7-dev python-minimal
sudo update-alternatives --install /usr/bin/python python /usr/bin/python2 1
sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 2

curl https://raw.githubusercontent.com/Michdo93/get-pip/main/get-pip.py -o get-pip.py
sudo chmod +x get-pip.py
python get-pip.py
```

And you have to change `libboost`:

```
sudo apt remove libboost1.*

sudo apt-get install -y libboost1.58-all-dev

sudo -H pip install empy
sudo -H pip install defusedxml
sudo -H pip install paramiko
```

```
sudo apt-get install python-rosdep python-rosinstall-generator python-wstool python-rosinstall build-essential
sudo rosdep init --rosdistro kinetic
rosdep update --include-eol-distros

mkdir ~/ros_catkin_ws
cd ~/ros_catkin_ws
rosinstall_generator desktop_full --rosdistro kinetic --deps --wet-only --tar > kinetic-desktop-full-wet.rosinstall
wstool init -j8 src kinetic-desktop-full-wet.rosinstall
rosdep install --from-paths src --ignore-src --rosdistro kinetic -y
./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release
```

## Configure your ROS environment

```
source ~/.bashrc

sudo nano /etc/environment
ROS_HOSTNAME="$(hostname -f)"
ROS_MASTER_URI=http://$ROS_HOSTNAME:11311
ROS_IP="$(hostname -I | awk '{print $1;}')"
```

## Install the NAOqi Python SDK

```
mkdir ~/naoqi
cd ~/naoqi
wget https://community-static.aldebaran.com/resources/2.5.5/sdk-python/pynaoqi-python2.7-2.5.5.5-linux64.tar.gz
wget https://community-static.aldebaran.com/resources/2.5.5/naoqi-sdk/naoqi-sdk-2.5.5.5-linux64.tar.gz
tar xzf pynaoqi-python2.7-2.5.5.5-linux64.tar.gz
tar xzf naoqi-sdk-2.5.5.5-linux64.tar.gz
```

You can check it with `~/naoqi/naoqi-sdk-2.5.5.5-linux64/naoqi`.

Then you have to add it to your `PYTHONPATH`:

```
echo 'export PYTHONPATH=~/naoqi/pynaoqi-python2.5.5.5.5-linux64/lib/python2.7/site-packages:$PYTHONPATH' >> ~/.bashrc
source ~/.bashrc
```

Check with:

```
python # should open the Python 2.7 shell
>>> from naoqi import ALProxy
```

## Install Pepper Packages

```
sudo apt-get install ros-kinetic-driver-base ros-kinetic-move-base-msgs ros-kinetic-octomap ros-kinetic-octomap-msgs ros-kinetic-humanoid-msgs ros-kinetic-humanoid-nav-msgs ros-kinetic-camera-info-manager ros-kinetic-camera-info-manager-py
sudo apt-get install ros-kinetic-pepper-.*
sudo apt-get install ros-kinetic-pepper-robot ros-kinetic-pepper-meshes

mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/src
git clone https://github.com/ros-naoqi/naoqi_driver.git

rosdep install -i -y --from-paths ./naoqi_driver

git clone https://github.com/ros-naoqi/pepper_robot.git
git clone https://github.com/ros-naoqi/pepper_virtual.git
git clone https://github.com/ros-naoqi/naoqi_dcm_driver.git
git clone https://github.com/ros-naoqi/pepper_dcm_robot.git
git clone https://github.com/ros-naoqi/pepper_moveit_config.git

git clone https://github.com/ros-naoqi/naoqi_bridge.git

cd ~/catkin_ws && catkin_make

cp ~/catkin_ws/src/pepper_robot/pepper_bringup/config/pepper.rviz ~/catkin_ws/src/pepper_robot/pepper_description/config/
```

For testing it you can run at first the roscore

```
roscore
```

then

```
roslaunch pepper_bringup pepper_full.launch nao_ip:=<yourRobotIP> roscore_ip:=<roscore_ip> [network_interface:=<eth0|wlan0|vpn0>]
```

If you run ROS inside a virtual machine instead of `èth0` it could be something like `enps03` or `ens33` etc.

## Gazebo

Please have a look at [here](https://github.com/ros-naoqi/pepper_robot/issues/47) and [here](https://answers.ros.org/question/292444/gazebo_ros_control-plugin-gazeboroscontrolplugin-missing-legacymodens-defaultrobothwsim/).

```
roslaunch pepper_gazebo_plugin pepper_gazebo_plugin_Y20.launch
roslaunch pepper_dcm_bringup pepper_bringup.launch robot_ip:=127.0.0.1 network_interface:=eno2
roslaunch pepper_bringup pepper_full.launch nao_ip:=127.0.0.1 roscore_ip:=192.168.0.1 network_interface:=eno2


rosrun rviz rviz -d ~/catkin_ws/src/pepper_robot/pepper_description/config/pepper.rviz
rosrun rviz rviz -d ~/catkin_ws/src/pepper_description/config/pepper.rviz

rosrun image_view image_view image:=/pepper_robot/camera/bottom/image_raw
```

## Other

```
git clone https://github.com/ros-naoqi/nao_extras.git
git clone https://github.com/ahornung/nao_common.git

git clone https://github.com/Michdo93/pepper_nav_bringup

cd .. && catkin_make

roslaunch nao_teleop teleop_joy.launch

roslaunch pepper_plymouth_nao mapping.launch

rosrun map_server map_server <path to map>/map.yaml
rosrun amcl amcl scan:=/pepper_robot/laser
rosrun map_server map_saver

roslaunch pepper_nav_bringup nav.launch
roslaunch pepper_nav_bringup nav.launch map:=<full path to your map.yaml>
```

Please notice:
https://github.com/ros-naoqi/pepper_robot/pull/27

```
sudo apt install ros-kinetic-octo*
roslaunch pepper_nav_bringup octomap_mapping.launch
```
