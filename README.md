# Pointers

## ROS2 / Autoware syntax
See the markdown files in this repository for quick examples of how to use the main ROS2 and autoware functionality.

## Visualization
For ROS2 visualization, you can clone my `ros_tools` package (https://github.com/oscardegroot/ros_tools/tree/ros2):

```
git clone https://github.com/oscardegroot/ros_tools.git -b ros2
```

## Usage of Forces Pro in C++
See C++ code and `CMakelists.txt` in https://github.com/oscardegroot/forces_pro_server for how to compile a solver in C++/ROS.

To generate a solver with floating license, you need to set the following options in your solver:

```
options.license.use_floating_license = 1
```

You can generate the solver outside of the docker and it is possible to generate it with another license attached to the same account.

To run the solver with floating license, download and run the proxy of the floating license from your forces account:

```
./forcespro_floating_licenses_proxy
```

If the exit flag is not `-100` the floating solver is active. 
