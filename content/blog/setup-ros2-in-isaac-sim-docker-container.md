---
title: Setup ROS2 in Isaac Sim Docker Container
date: 2025-02-07T14:39:00
tags: []
series: []
featured: true
---

This article is about how to setup ROS2 in Isaac Sim Docker Container.

<!--more-->
## Prerequisites

- Docker
- Isaac Sim Docker Image


## Steps

1. Start the Isaac Sim Docker Container

```bash
docker run --name isaac-sim --entrypoint bash -it --runtime=nvidia --gpus all -e "ACCEPT_EULA=Y" --rm --network=host \
    -e "PRIVACY_CONSENT=Y" \
    -v ~/docker/isaac-sim/cache/kit:/isaac-sim/kit/cache:rw \
    -v ~/docker/isaac-sim/cache/ov:/root/.cache/ov:rw \
    -v ~/docker/isaac-sim/cache/pip:/root/.cache/pip:rw \
    -v ~/docker/isaac-sim/cache/glcache:/root/.cache/nvidia/GLCache:rw \
    -v ~/docker/isaac-sim/cache/computecache:/root/.nv/ComputeCache:rw \
    -v ~/docker/isaac-sim/logs:/root/.nvidia-omniverse/logs:rw \
    -v ~/docker/isaac-sim/data:/root/.local/share/ov/data:rw \
    -v ~/docker/isaac-sim/documents:/root/Documents:rw \
    nvcr.io/nvidia/isaac-sim:4.5.0
```

2. Step up ROS2 environment

```bash
echo "export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/isaac-sim/exts/omni.isaac.ros2_bridge/humble/lib" >> ~/.bashrc
echo "export PYTHONPATH=/isaac-sim/exts/omni.isaac.ros2_bridge/humble/rclpy/:$PYTHONPATH" >> ~/.bashrc
echo "export RMW_IMPLEMENTATION=rmw_fastrtps_cpp" >> ~/.bashrc
echo "export FASTRTPS_DEFAULT_PROFILES_FILE=~/.ros/fastdds.xml" >> ~/.bashrc
```


3. Create DDS profile file

copy the following content to `~/.ros/fastdds.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<license>Copyright (c) 2022-2024, NVIDIA CORPORATION.  All rights reserved.
NVIDIA CORPORATION and its licensors retain all intellectual property
and proprietary rights in and to this software, related documentation
and any modifications thereto.  Any use, reproduction, disclosure or
distribution of this software and related documentation without an express
license agreement from NVIDIA CORPORATION is strictly prohibited.</license>


<profiles xmlns="http://www.eprosima.com/XMLSchemas/fastRTPS_Profiles" >
    <transport_descriptors>
        <transport_descriptor>
            <transport_id>UdpTransport</transport_id>
            <type>UDPv4</type>
        </transport_descriptor>
    </transport_descriptors>

    <participant profile_name="udp_transport_profile" is_default_profile="true">
        <rtps>
            <userTransports>
                <transport_id>UdpTransport</transport_id>
            </userTransports>
            <useBuiltinTransports>false</useBuiltinTransports>
        </rtps>
    </participant>
</profiles>
```

3. Test ROS2 in Isaac Sim

```bash
export ROS_DOMAIN_ID=<your_domain_id>
./python.sh ros_node.py # the script is located in the /isaac-sim
[INFO] [1738997353.194031795] [test_node]: Published: Hello from Isaac Sim internal ROS2!
[INFO] [1738997354.188423330] [test_node]: Published: Hello from Isaac Sim internal ROS2!
[INFO] [1738997355.188394277] [test_node]: Published: Hello from Isaac Sim internal ROS2!
[INFO] [1738997356.188390781] [test_node]: Published: Hello from Isaac Sim internal ROS2!
[INFO] [1738997357.188404718] [test_node]: Published: Hello from Isaac Sim internal ROS2!
[INFO] [1738997358.188400347] [test_node]: Published: Hello from Isaac Sim internal ROS2!
```

4. Test ROS2 LAN communication

The `isaac-sim` container does not provide `ros2` command, so you need to use the host or other containers which have ROS2 installed.

```bash
ros2 topic echo /test_topic

data: Hello from Isaac Sim internal ROS2!
---
data: Hello from Isaac Sim internal ROS2!
---
data: Hello from Isaac Sim internal ROS2!
---
data: Hello from Isaac Sim internal ROS2!
---
data: Hello from Isaac Sim internal ROS2!
```

**NOTE:**

- Make sure the ROS_DOMAIN_ID is the same as the host.
- Make sure the network of the host and the container are the same.
- Make sure the DDS is the same configuration.
