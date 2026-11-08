cmake_minimum_required(VERSION 3.8)
project(stonefish_ros2 VERSION 1.6.0)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

set(CMAKE_CXX_STANDARD 20)

# find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(ament_index_cpp REQUIRED)
find_package(std_srvs REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(nav_msgs REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(visualization_msgs REQUIRED)
find_package(rosidl_default_generators REQUIRED)
find_package(pcl_conversions REQUIRED)
find_package(image_transport REQUIRED)
find_package(tf2 REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(Stonefish REQUIRED 1.6.0)

# messages
rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/Int32Stamped.msg"
  "msg/BeaconInfo.msg"
  "msg/NEDPose.msg"
  "msg/INS.msg"
  "msg/DVLBeam.msg"
  "msg/DVL.msg"
  "msg/ThrusterState.msg"
  "msg/DebugPhysics.msg"
  "msg/Event.msg"
  "msg/EventArray.msg"
  "srv/SonarSettings.srv"
  "srv/SonarSettings2.srv"
  "srv/Respawn.srv"
  DEPENDENCIES std_msgs std_srvs geometry_msgs
)
rosidl_get_typesupport_target(cpp_typesupport_target ${PROJECT_NAME} rosidl_typesupport_cpp)

find_package(PCL REQUIRED) # Interferes with rosidl_generate_interfaces and thus has to be called after

# define libraries
add_library(stonefish_ros2_interface SHARED
  src/${PROJECT_NAME}/ROS2ConsoleSimulationApp.cpp
  src/${PROJECT_NAME}/ROS2GraphicalSimulationApp.cpp
  src/${PROJECT_NAME}/ROS2SimulationManager.cpp
  src/${PROJECT_NAME}/ROS2ScenarioParser.cpp
  src/${PROJECT_NAME}/ROS2Interface.cpp
)
target_link_libraries(stonefish_ros2_interface Stonefish::Stonefish "${cpp_typesupport_target}")
target_include_directories(stonefish_ros2_interface
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>)
target_link_libraries(stonefish_ros2_interface
  rclcpp::rclcpp
  ament_index_cpp::ament_index_cpp
  ${std_srvs_TARGETS}
  ${geometry_msgs_TARGETS}
  ${nav_msgs_TARGETS}
  ${visualization_msgs_TARGETS}
  ${sensor_msgs_TARGETS}
  ${PCL_LIBRARIES}
  pcl_conversions::pcl_conversions
  image_transport::image_transport
  tf2::tf2
  tf2_ros::tf2_ros
)
target_include_directories(stonefish_ros2_interface SYSTEM PUBLIC ${PCL_INCLUDE_DIRS})
target_compile_definitions(stonefish_ros2_interface PUBLIC ${PCL_DEFINITIONS})
ament_export_libraries(stonefish_ros2_interface)
ament_export_include_directories(include)

# define exacutables
add_executable(stonefish_simulator src/stonefish_simulator.cpp)
target_link_libraries(stonefish_simulator stonefish_ros2_interface)
target_link_libraries(stonefish_simulator rclcpp::rclcpp)

add_executable(stonefish_simulator_nogpu src/stonefish_simulator_nogpu.cpp)
target_link_libraries(stonefish_simulator_nogpu stonefish_ros2_interface)
target_link_libraries(stonefish_simulator_nogpu rclcpp::rclcpp)

# installation
install(
  TARGETS stonefish_ros2_interface
  LIBRARY DESTINATION lib
  INCLUDES DESTINATION include
)
install(
  DIRECTORY include
  DESTINATION include
)

install(
  TARGETS stonefish_simulator stonefish_simulator_nogpu
  DESTINATION lib/${PROJECT_NAME}
)
install(
  DIRECTORY launch
  DESTINATION share/${PROJECT_NAME}
)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  # the following line skips the linter which checks for copyrights
  # comment the line when a copyright and license is added to all source files
  set(ament_cmake_copyright_FOUND TRUE)
  # the following line skips cpplint (only works in a git repo)
  # comment the line when this package is in a git repo and when
  # a copyright and license is added to all source files
  set(ament_cmake_cpplint_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
