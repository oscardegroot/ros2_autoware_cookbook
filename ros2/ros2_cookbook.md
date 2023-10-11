# ROS2 Cookbook
Below are some pointers that I took while migrating from ROS1 to ROS2.

---
### Nodes
Every ROS2 node should extend `rclcpp::Node`. Some notes:

- Occurences of `node->` can locally be replaced by `this->` (e.g., to create a publisher)
- Although shared pointers should be used for passing the node object, you cannot create a shared pointer inside of the constructor. I am following Autoware where the raw pointer is passed instead. That is `rclcpp::Node*` instead of `rclcpp::Node::SharedPtr`.

---
### Topic - Publisher 
Declaration (for string example)
```cpp
rclcpp::Publisher<std_msgs::msg::String>::SharedPtr publisher_
```

Initiation:

```cpp
publisher_ = node->create_publisher<std_msgs::msg::String>(<topic_name>, 1);
```

---
### Topic - Subscriber
Example subscriber declaration for the case that the callback function is a member function (`StateCallback` inside a class `JackalSimulatorInterface` in this case):

```
node->create_subscription<std_msgs::msg::String>(
      <topic_name>, 1,
      std::bind(&JackalSimulatorInterface::StateCallBack, this, std::placeholders::_1));
```

---
### Parameters
You need to declare parameters that you want to use in your nodes. Because parameters can only be declared once, I found the following declaration wrappers better to use
```cpp
template <class T>
T DeclareOrGetParam(rclcpp::Node *node, const std::string &name)
{
    if (node->has_parameter(name))
        return node->get_parameter(name).get_value<T>();
    else
        return node->declare_parameter<T>(name);
}

template <class T>
T DeclareOrGetParam(rclcpp::Node::SharedPtr node, const std::string &name)
{
    return DeclareOrGetParam<T>(node.get(), name);
}

template <class T>
T DeclareOrGetParam(rclcpp::Node *node, const std::string &name, const T &default_value)
{
    if (node->has_parameter(name))
        return node->get_parameter(name).get_value<T>();
    else
        return node->declare_parameter<T>(name, default_value);
}

template <class T>
T DeclareOrGetParam(rclcpp::Node::SharedPtr node, const std::string &name, const T &default_value)
{
    return DeclareOrGetParam<T>(node.get(), name, default_value);
}
```

To update the parameters, there is functionality in Autoware

```
  updateParam<bool>(parameters, "guidance.options.add_original_planner", add_original_planner_);

```

---
### Logging
To write to the console you need a logger. 

Loggers are available from nodes via `node->get_logger()`.

If you want a logger anywhere else (with another name), you can use

```cpp
    rclcpp::get_logger(<logger_name>)
```

Sometimes it is useful to have a logger anywhere by defining in a namespace
```
static const rclcpp::Logger LOGGER = rclcpp::get_logger("ros_tools.data_saver");
```


When you have a logger you can log using
```
RCLCPP_INFO(logger, <message>)
RCLCPP_WARN(logger, <message>)
RCLCPP_ERROR(logger, <message>)
```

You can get colorized outputs by defining
```
#define RCUTILS_COLORIZED_OUTPUT 1
```

You can modify the logging format by adding in ~/.bashrc
```
export RCUTILS_CONSOLE_OUTPUT_FORMAT="[{severity}] [{name}]: {message}"
```

---
### Time 
For general time (from ROS1):
`ros::Time::now → rclcpp::Clock()::now()`

Nodes also have a clock via `node->now()`. Mixing timers from different sources can lead to errors.

---
### Notes
- Other cookbook: https://github.com/mikeferguson/ros2_cookbook/tree/main/rclcpp
- For exporting Boost requirement, there is an issue, see https://gist.github.com/tylerjw/ce3bcb94b3df7996e450dec18ee54cde
- Example ROS2 code: https://github.com/ros-planning/moveit2/blob/main/moveit_ros/visualization/motion_planning_rviz_plugin/src/motion_planning_frame_planning.cpp
- Migration from ROS1 official documentation: https://docs.ros.org/en/humble/How-To-Guides/Migrating-from-ROS1/Migrating-CPP-Packages.html

---
## Examples
Example launch file
```xml
<?xml version="1.0"?>

<launch>

    <node pkg="lmpcc_jackal" exec="lmpcc_jackal_node" name="lmpcc_jackal_node" respawn="false" output="screen">
        <param from="$(find-pkg-share lmpcc_jackal)/config/jackalsimulator/path.yaml"/>
        <param from="$(find-pkg-share lmpcc_jackal)/config/jackalsimulator/parameters.yaml"/>
        <param from="$(find-pkg-share lmpcc_jackal)/config/jackalsimulator/topics.yaml"/>
        <param from="$(find-pkg-share lmpcc_jackal)/config/jackalsimulator/guidance_planner.yaml"/>
        <param from="$(find-pkg-share lmpcc_jackal)/config/jackalsimulator/autotune_parameters.yaml"/>
    </node>

    <include file="$(find-pkg-share pedestrian_simulator)/launch/simulation.launch.py"/>
    <include file="$(find-pkg-share jackal_gazebo)/launch/jackal_world.launch.py"/>

    <node pkg="tf2_ros" exec="static_transform_publisher" name="static_map" output="log" args="0 0 0 0 0 0 odom map"/>

  <node pkg="rviz2" exec="rviz2" name="rviz2" args="-d $(find-pkg-share lmpcc_jackal)/rviz/jackalsimulator/default.rviz" namespace="/" output="log"/>
  
  <node pkg="rqt_reconfigure" exec="rqt_reconfigure" name="rqt_reconfigure" namespace="/" output="log"/>

</launch>  

```
---

Example parameter file. Note that the first two lines are `/**:` (all nodes) and `ros__parameters:` (ros parameters)

```yml
/**:
  ros__parameters:
    prm:
      clock_frequency: 20.
      debug_output: false
      seed: 1                               # Seed of the visibility-PRM. Set to -1 for random!
      T: 6.0
      N: 30
      topology_comparison: Homology         # Homology (default) or UVD
      sampling_function: Uniform # Uniform (default)
      use_learning: false
      predictions_are_constant_velocity: false
      track_selected_homology_only: false
      sample_margin: 0. # Was 0.
      view_angle_times_pi: 0.606  # View angle when sampling in a conus (disabled by default)
      n_samples: 20 #30 # Number of samples for PRM
      timeout: 10. # Timeout for PRM sampling [ms]
      n_paths: 4       # Number of guidance trajectories
      max_velocity:     3.0       # Maximum velocity of connections between nodes
      max_acceleration: 3.0       # Maximum velocity of connections between nodes
      connection_filters:
        forward:      true
        acceleration: true
      goals:  # Only used when `LoadReferencePath` is used to set the goals
        longitudinal: 5   # Number of goals in direction of the path
        vertical: 5       # Number of goals in direction orthogonal to the path
      selection_weights:  # Selection weights to determine what spline is best
        consistency: 1.33 # 0.24904675216524505  # How much better should a new trajectory be to be selected [%]
        length: 5. #0. #1.
        velocity: 0. #2.
        acceleration: 1. #1.
      spline_optimization:  # Settings when the splines are optimized and used as reference trajectory
        enable: true #false
        num_points: 10      # -1 = N
        geometric: 25.
        smoothness: 10.
        collision: 0.5
        velocity_tracking: 0.01
      visuals:
        transparency: 0.5 #0.9               # The least transparent the obstacle visualization is (0-1)
        visualize_all_samples: false    # Visualizes all PRM samples
        visualize_homology: false
        show_indices: false
      enable:
        dynamically_propagate_nodes: true  # Propagate the nodes in time (dropping them)
        project_from_obstacles: false       # Project the guidance trajectory from obstacles if enabled (not necessary by default)
      test_node:
        continuous_replanning: false #true         # When using the test nodes: keep planning continuously?
```

---
Example `CMakelists`

```cmake
cmake_minimum_required(VERSION 3.5)
project(lmpcc_base)

if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 17)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic -Wno-unused-parameter)
endif()
add_compile_options(-DCMAKE_BUILD_TYPE=Release)

## DEPENDENCIES ##
set(DEPENDENCIES
  rclcpp
  ros_tools
  lmpcc_common
  lmpcc_solver
  lmpcc_msgs
  geometry_msgs
  Eigen3
)

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)

find_package(ros_tools REQUIRED)
find_package(lmpcc_msgs REQUIRED)
find_package(geometry_msgs REQUIRED)

find_package(lmpcc_common REQUIRED)
find_package(lmpcc_solver REQUIRED)

find_package(eigen3_cmake_module REQUIRED)
find_package(Eigen3 REQUIRED)
add_definitions(${EIGEN_DEFINITIONS})

## Build ##
include_directories(
  include
  ${EIGEN_INCLUDE_DIRS}
)

add_library(${PROJECT_NAME}
  src/dynamic_obstacle.cpp
  src/types.cpp
  src/controller_module.cpp
  src/lmpcc_configuration.cpp
  
  src/util/experiment-util.cpp

  include/third_party/tkspline/spline.cpp
  include/third_party/Clothoid.cpp
)
target_include_directories(${PROJECT_NAME}
  PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
    "$<INSTALL_INTERFACE:include/${PROJECT_NAME}>"
)

ament_target_dependencies(${PROJECT_NAME}
  ${DEPENDENCIES}
)

## INSTALL ##
install(
  TARGETS ${PROJECT_NAME}
  EXPORT export_${PROJECT_NAME}
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
)

install(DIRECTORY include/ 
  DESTINATION include/${PROJECT_NAME})

# ament_export_include_directories(include/${PROJECT_NAME})
ament_export_targets(export_${PROJECT_NAME} HAS_LIBRARY_TARGET)
ament_export_dependencies(${DEPENDENCIES})

ament_package()
```
---
Example `package.xml`

```xml
<?xml version="1.0"?>
<package format="3">
  <name>lmpcc_base</name>
  <version>0.6.14</version>
  <description>Package Description</description>

  <maintainer email="todo">Todo</maintainer>
  <author>Todo</author>
  <license>Todo</license>

  <depend>rclcpp</depend>

  <depend>eigen</depend>
  <depend>lmpcc_msgs</depend>
  <depend>lmpcc_solver</depend>

  <depend>ros_tools</depend>

  <export>
    <build_type>ament_cmake</build_type>
  </export>
</package>
```