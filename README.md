# Pointers

## ROS2 / Autoware syntax
See the markdown files in this repository for quick examples of how to use the main ROS2 and autoware functionality.

## Visualization
For ROS2 visualization, you can clone my `ros_tools` package (https://github.com/oscardegroot/ros_tools/tree/ros2):

```
git clone https://github.com/oscardegroot/ros_tools.git -b ros2
```

## Usage of Forces Pro in C++
See C++ code and `CMakelists.txt` in https://github.com/oscardegroot/forces_pro_server.

For the floating license. You need to set in your solver options:

```
options.license.use_floating_license = 1
```

Then download the proxy of the floating license from your forces account and execute it (I think from the same machine):

```
./forcespro_floating_licenses_proxy
```

