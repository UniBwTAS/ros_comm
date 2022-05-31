cmake_minimum_required(VERSION 3.0.2)
project(rosbag)

if(NOT WIN32)
  set_directory_properties(PROPERTIES COMPILE_OPTIONS "-Wall;-Wextra")
endif()

find_package(catkin REQUIRED COMPONENTS rosbag_storage rosconsole roscpp std_srvs std_msgs topic_tools xmlrpcpp rospy std_msgs message_generation)
find_package(Boost REQUIRED COMPONENTS date_time filesystem program_options regex thread)
find_package(BZip2 REQUIRED)

catkin_python_setup()

# Support large bags (>2GB) on 32-bit systems
add_definitions(-D_FILE_OFFSET_BITS=64)

include_directories(include ${catkin_INCLUDE_DIRS} ${Boost_INCLUDE_DIRS}
  ${BZIP2_INCLUDE_DIR}
)

# Generate messages in the 'msg' folder
add_message_files(
  FILES
  BagInfo.msg
  MessageQuery.msg
)

# Generate services in the 'srv' folder
add_service_files(
    FILES
    OpenBags.srv
    PlayOptions.srv
    QueryOptions.srv
)

# Generate added messages and services with any dependencies listed here
generate_messages(
    DEPENDENCIES
    std_msgs std_srvs
)

catkin_package(
  LIBRARIES rosbag
  INCLUDE_DIRS include
  CATKIN_DEPENDS rosbag_storage rosconsole roscpp std_srvs std_msgs topic_tools xmlrpcpp)

add_library(rosbag
  src/player.cpp
  src/recorder.cpp
  src/time_translator.cpp)

target_link_libraries(rosbag ${catkin_LIBRARIES} ${Boost_LIBRARIES}
  ${BZIP2_LIBRARIES}
)
add_dependencies(rosbag ${rosbag_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

add_executable(record src/record.cpp)
target_link_libraries(record rosbag)

add_executable(play src/play.cpp)
target_link_libraries(play rosbag)


if(NOT WIN32)
  add_executable(encrypt src/encrypt.cpp)
  target_link_libraries(encrypt ${catkin_LIBRARIES} ${Boost_LIBRARIES})
endif()

install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.h")
install(TARGETS rosbag
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION})
install(TARGETS record play
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})
if(NOT WIN32)
  install(TARGETS encrypt
    ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})
endif()
catkin_install_python(PROGRAMS
  scripts/bag2png.py
  scripts/bagsort.py
  scripts/fastrebag.py
  scripts/fixbag.py
  scripts/fixbag_batch.py
  scripts/fix_md5sums.py
  scripts/fix_moved_messages.py
  scripts/fix_msg_defs.py
  scripts/makerule.py
  scripts/savemsg.py
  scripts/topic_renamer.py
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})

if(CATKIN_ENABLE_TESTING)
  catkin_add_nosetests(test)
endif()
