cmake_minimum_required(VERSION 2.8.3)
project(cob_twist_controller)

# Load catkin and all dependencies required for this package
find_package(catkin REQUIRED COMPONENTS cmake_modules roscpp sensor_msgs geometry_msgs nav_msgs kdl_parser kdl_conversions tf tf_conversions dynamic_reconfigure)

find_package(Eigen REQUIRED)
add_definitions(${EIGEN_DEFINITIONS})

find_package(orocos_kdl REQUIRED)

generate_dynamic_reconfigure_options(
  cfg/TwistController.cfg
)

# Declare catkin package
catkin_package(
   INCLUDE_DIRS include
   LIBRARIES cob_twist_controller cob_twist_controller_debug augmented_solver
   CATKIN_DEPENDS roscpp sensor_msgs geometry_msgs nav_msgs
)

include_directories(include ${catkin_INCLUDE_DIRS} ${EIGEN_INCLUDE_DIRS} ${orocos_kdl_INCLUDE_DIRS} )


## Build
# Declare a cpp library
add_library(augmented_solver src/augmented_solver.cpp)
target_link_libraries(augmented_solver ${orocos_kdl_LIBRARIES})

add_library(${PROJECT_NAME} src/cob_twist_controller.cpp )
add_dependencies(augmented_solver ${PROJECT_NAME}_gencfg)
target_link_libraries(${PROJECT_NAME} augmented_solver ${catkin_LIBRARIES} ${orocos_kdl_LIBRARIES})

add_library(${PROJECT_NAME}_debug src/cob_twist_controller_debug.cpp )
add_dependencies(augmented_solver ${PROJECT_NAME}_gencfg)
target_link_libraries(${PROJECT_NAME}_debug augmented_solver ${catkin_LIBRARIES} ${orocos_kdl_LIBRARIES})

## Declare a cpp executable
add_executable(${PROJECT_NAME}_node src/cob_twist_controller_node.cpp)
target_link_libraries(${PROJECT_NAME}_node ${PROJECT_NAME} ${catkin_LIBRARIES} ${orocos_kdl_LIBRARIES})

add_executable(${PROJECT_NAME}_debug_node src/cob_twist_controller_debug_node.cpp)
target_link_libraries(${PROJECT_NAME}_debug_node ${PROJECT_NAME}_debug ${catkin_LIBRARIES} ${orocos_kdl_LIBRARIES})


## Install
# Mark executables and/or libraries for installation
install(TARGETS ${PROJECT_NAME} ${PROJECT_NAME}_node ${PROJECT_NAME}_debug ${PROJECT_NAME}_debug_node augmented_solver
 ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
 LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
 RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

## Mark cpp header files for installation
install(DIRECTORY include/${PROJECT_NAME}/
 DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
 FILES_MATCHING PATTERN "*.h"
 PATTERN ".svn" EXCLUDE
)

# Mark other files for installation (e.g. launch and bag files, etc.)
install(PROGRAMS scripts/test_publisher_twist.py scripts/test_publisher_twist_stamped.py scripts/test_publisher_twist_series.py
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}/scripts
)

