---
title: ROS2 Installation
date: 2026-09-22
---

# ROS2 Installation

## Update Sistem

```bash
sudo apt update
sudo apt upgrade -y
sudo reboot
```

## Set Locale

```bash
locale
sudo apt install locales -y
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8
```

## Add Repository Universe

```bash
sudo apt install software-properties-common -y
sudo add-apt-repository universe -y
```

## Add Repository ROS2

```bash
sudo apt update
sudo apt install curl -y
export ROS_APT_SOURCE_VERSION=$(curl -s https://api.github.com/repos/ros-infrastructure/ros-apt-source/releases/latest | grep -F "tag_name" | awk -F '"' '{print $4}')
curl -L -o /tmp/ros2-apt-source.deb "https://github.com/ros-infrastructure/ros-apt-source/releases/download/${ROS_APT_SOURCE_VERSION}/ros2-apt-source_${ROS_APT_SOURCE_VERSION}.$(. /etc/os-release && echo ${UBUNTU_CODENAME:-${VERSION_CODENAME}})_all.deb"
sudo dpkg -i /tmp/ros2-apt-source.deb
sudo apt update
```

## Install ROS2

```bash
sudo apt install ros-jazzy-ros-base -y   # untuk versi tanpa RViz dan Gazebo
sudo apt install ros-jazzy-desktop -y
```

## Setup Environment Otomatis

```bash
echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

## Cek Apakah Berhasil

```bash
echo $ROS_DISTRO   # -> jazzy
ros2 --help
```

## Install Tools

```bash
sudo apt install ros-dev-tools -y
sudo apt install python3-rosdep -y
sudo rosdep init
rosdep update
```

## Terkait

- [[Project_Overview]]
- [[Operate_ROS2]]
- [[22_September_2026]]
