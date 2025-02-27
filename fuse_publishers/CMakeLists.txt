cmake_minimum_required(VERSION 3.0.2)
project(fuse_publishers)

set(build_depends
  fuse_core
  fuse_variables
  geometry_msgs
  nav_msgs
  pluginlib
  roscpp
  tf2
  tf2_geometry_msgs
  tf2_ros
)

find_package(catkin REQUIRED COMPONENTS
  ${build_depends}
)

catkin_package(
  INCLUDE_DIRS
    include
  LIBRARIES
    ${PROJECT_NAME}
  CATKIN_DEPENDS
    ${build_depends}
)

###########
## Build ##
###########
add_compile_options(-Wall -Werror)

# On ARM, the use of Eigen 3.4 with g++11 (both the versions in Jammy) causes a warning, which in turn causes
# the build to fail. Details are here: https://gitlab.com/libeigen/eigen/-/issues/2326. We can suppress the warning
# for ARM builds here. However, this package does not depend directly upon Eigen, but includes headers from other
# packages that do. So to prevent the error, we need to find Eigen and then check the version.
if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER 11.0 AND
   CMAKE_LIBRARY_ARCHITECTURE STREQUAL "aarch64-linux-gnu")

   find_package(Eigen3)

   if(Eigen3_FOUND AND Eigen3_VERSION VERSION_EQUAL 3.4)
      add_compile_options(-Wno-class-memaccess)
   endif()
endif()

# fuse_publishers library
add_library(${PROJECT_NAME}
  src/path_2d_publisher.cpp
  src/pose_2d_publisher.cpp
  src/serialized_publisher.cpp
)
add_dependencies(${PROJECT_NAME}
  ${catkin_EXPORTED_TARGETS}
)
target_include_directories(${PROJECT_NAME}
  PUBLIC
    include
    ${catkin_INCLUDE_DIRS}
)
target_link_libraries(${PROJECT_NAME}
  ${catkin_LIBRARIES}
)
set_target_properties(${PROJECT_NAME}
  PROPERTIES
    CXX_STANDARD 17
    CXX_STANDARD_REQUIRED YES
)

#############
## Install ##
#############

install(
  TARGETS ${PROJECT_NAME}
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION}
)

install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
)

install(FILES fuse_plugins.xml
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)

#############
## Testing ##
#############

if(CATKIN_ENABLE_TESTING)
  set(test_depends
    fuse_constraints
    fuse_graphs
  )

  find_package(catkin REQUIRED COMPONENTS
    ${build_depends}
    ${test_depends}
  )
  find_package(roslint REQUIRED)
  find_package(rostest REQUIRED)

  # Lint tests
  set(ROSLINT_CPP_OPTS "--filter=-build/c++11,-runtime/references")
  roslint_cpp()
  roslint_add_test()

  # Path2DPublisher Tests
  add_rostest_gtest(test_path_2d_publisher
    test/path_2d_publisher.test
    test/test_path_2d_publisher.cpp
  )
  add_dependencies(test_path_2d_publisher
    ${catkin_EXPORTED_TARGETS}
  )
  target_include_directories(test_path_2d_publisher
    PRIVATE
      include
      ${catkin_INCLUDE_DIRS}
  )
  target_link_libraries(test_path_2d_publisher
    ${PROJECT_NAME}
    ${catkin_LIBRARIES}
  )
  set_target_properties(test_path_2d_publisher
    PROPERTIES
      CXX_STANDARD 17
      CXX_STANDARD_REQUIRED YES
  )

  # Pose2DPublisher Tests
  add_rostest_gtest(test_pose_2d_publisher
    test/pose_2d_publisher.test
    test/test_pose_2d_publisher.cpp
  )
  add_dependencies(test_pose_2d_publisher
    ${catkin_EXPORTED_TARGETS}
  )
  target_include_directories(test_pose_2d_publisher
    PRIVATE
      include
      ${catkin_INCLUDE_DIRS}
  )
  target_link_libraries(test_pose_2d_publisher
    ${PROJECT_NAME}
    ${catkin_LIBRARIES}
  )
  set_target_properties(test_pose_2d_publisher
    PROPERTIES
      CXX_STANDARD 17
      CXX_STANDARD_REQUIRED YES
  )

  # StampedVariableSynchronizer Tests
  catkin_add_gtest(test_stamped_variable_synchronizer
    test/test_stamped_variable_synchronizer.cpp
  )
  add_dependencies(test_stamped_variable_synchronizer
    ${catkin_EXPORTED_TARGETS}
  )
  target_include_directories(test_stamped_variable_synchronizer
    PRIVATE
      include
      ${catkin_INCLUDE_DIRS}
  )
  target_link_libraries(test_stamped_variable_synchronizer
    ${catkin_LIBRARIES}
  )
  set_target_properties(test_stamped_variable_synchronizer
    PROPERTIES
      CXX_STANDARD 17
      CXX_STANDARD_REQUIRED YES
  )
endif()
