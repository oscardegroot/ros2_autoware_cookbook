# Compiling
Autoware has a CMake system that simplifies the building process.

All the dependencies of the package should be in the `package.xml`, then the `CMakelists.txt` can be simple. An example:

```
cmake_minimum_required(VERSION 3.5)
project(roadmap)

find_package(autoware_cmake REQUIRED)
autoware_package()

ament_auto_add_library(${PROJECT_NAME} SHARED
  src/roadmap.cpp
  src/configuration.cpp
  src/reader.cpp
  src/spline_converter.cpp
  src/spline/spline.cpp
  src/spline/Clothoid.cpp
)

ament_auto_add_executable(${PROJECT_NAME}_node src/roadmap_node.cpp)
add_dependencies(${PROJECT_NAME}_node ${PROJECT_NAME})
target_link_libraries(${PROJECT_NAME}_node ${PROJECT_NAME})

install(DIRECTORY config
  DESTINATION share/${PROJECT_NAME})

install(DIRECTORY launch
  DESTINATION share/${PROJECT_NAME})

install(DIRECTORY maps
  DESTINATION share/${PROJECT_NAME})

install(DIRECTORY rviz
  DESTINATION share/${PROJECT_NAME})

ament_auto_package()

```