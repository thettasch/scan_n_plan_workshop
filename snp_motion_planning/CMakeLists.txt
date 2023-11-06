cmake_minimum_required(VERSION 3.5)
project(snp_motion_planning)

if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 17)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

set(ROS2_DEPS
    rclcpp
    tesseract_monitoring
    tf2_eigen)

find_package(ament_cmake REQUIRED)
foreach(dep ${ROS2_DEPS})
  find_package(${dep} REQUIRED)
endforeach()

find_package(descartes_light REQUIRED)
find_package(tesseract_command_language REQUIRED)
find_package(tesseract_task_composer REQUIRED)
find_package(tesseract_kinematics REQUIRED)
find_package(LAPACK REQUIRED) # Requried for ikfast
find_package(yaml-cpp REQUIRED)

# Planning server

# Plugin Library
include_directories(include/${PROJECT_NAME})
add_library(${PROJECT_NAME}_tasks SHARED src/constant_tcp_speed_time_parameterization_task.cpp
                                         src/kinematic_limits_check_task.cpp)
ament_target_dependencies(${PROJECT_NAME}_tasks ${ROS2_DEPS})
target_link_libraries(
  ${PROJECT_NAME}_tasks
  tesseract::tesseract_common
  tesseract::tesseract_command_language
  tesseract::tesseract_task_composer
  tesseract::tesseract_kinematics_kdl
  tesseract::tesseract_time_parameterization_core
  yaml-cpp)

install(DIRECTORY include/${PROJECT_NAME} DESTINATION include)

# Install the executable(s)
# install(TARGETS ${PROJECT_NAME}_node DESTINATION lib/${PROJECT_NAME})

# Install the library(ies)
# install(TARGETS ${PROJECT_NAME}_plugins ${PROJECT_NAME}_tasks EXPORT ${PROJECT_NAME}-targets DESTINATION lib)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

ament_export_dependencies(${ROS2_DEPS})
# ament_export_targets(${PROJECT_NAME}-targets HAS_LIBRARY_TARGET)
ament_package()

# install(DIRECTORY config launch DESTINATION share/${PROJECT_NAME}/)
