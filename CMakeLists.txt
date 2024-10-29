cmake_minimum_required(VERSION 3.8)
project(kinect_ros2)

# Ustawienia kompilacji
if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# Ustawienie katalogu z nagłówkami
include_directories(include)

# Znajdź zależności
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rclcpp_components REQUIRED)
find_package(ament_index_cpp REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(cv_bridge REQUIRED)
find_package(image_transport REQUIRED)
find_package(depth_image_proc REQUIRED)
find_package(camera_info_manager REQUIRED)
find_package(OpenCV REQUIRED)


# Sprawdzenie dostępności libfreenect
find_package(libfreenect REQUIRED)

# Tworzenie biblioteki komponentu
add_library(kinect_ros2_component SHARED src/kinect_ros2_component.cpp)
target_compile_definitions(kinect_ros2_component
  PRIVATE "COMPOSITION_BUILDING_DLL")

# Powiązanie zależności dla komponentu
ament_target_dependencies(kinect_ros2_component 
  rclcpp
  sensor_msgs
  std_msgs
  cv_bridge
  camera_info_manager
  rclcpp_components
  ament_index_cpp
  image_transport
  OpenCV
  libfreenect)  # Dodano libfreenect jako zależność

# Rejestracja węzła komponentu
rclcpp_components_register_nodes(kinect_ros2_component "kinect_ros2::KinectRosComponent")

# Tworzenie węzła wykonawczego
add_executable(kinect_ros2_node
  src/kinect_ros2_node.cpp)

# Powiązanie biblioteki komponentu z węzłem
target_link_libraries(kinect_ros2_node
  kinect_ros2_component
  freenect)

# Powiązanie zależności dla węzła
ament_target_dependencies(kinect_ros2_node
  rclcpp
  sensor_msgs
  std_msgs
  cv_bridge
  camera_info_manager
  rclcpp_components
  ament_index_cpp
  image_transport
  OpenCV
  libfreenect)  # Upewnij się, że libfreenect jest tutaj także

# Instalacja komponentu
install(TARGETS
  kinect_ros2_component
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin)

# Instalacja węzła
install(TARGETS
  kinect_ros2_node
  DESTINATION lib/${PROJECT_NAME})

# Instalacja dodatkowych zasobów
install(
  DIRECTORY cfg launch rviz
  DESTINATION share/${PROJECT_NAME}
)

# Zakończenie pakietu
ament_package()
