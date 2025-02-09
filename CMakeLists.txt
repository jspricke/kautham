#     Copyright 2021 Institute of Industrial and Control Engineering (IOC)
#                  Universitat Politecnica de Catalunya
#                  BarcelonaTech
#     All Rights Reserved.
#
#     This program is free software; you can redistribute it and/or modify
#     it under the terms of the GNU General Public License as published by
#     the Free Software Foundation; either version 2 of the License, or
#     (at your option) any later version.
#
#     This program is distributed in the hope that it will be useful,
#     but WITHOUT ANY WARRANTY; without even the implied warranty of
#     MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#     GNU General Public License for more details.
#
#     You should have received a copy of the GNU General Public License
#     along with this program; if not, write to the
#     Free Software Foundation, Inc.,
#     59 Temple Place - Suite 330, Boston, MA  02111-1307, USA.
#
#    Author: Leopold Palomo-Avellaneda 2014-2019



project(kautham)

cmake_minimum_required(VERSION 3.1.3)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_definitions(-Wall -Wextra -Wpedantic -fPIC)#-Werror)

set( KAUTHAM_VERSION 4.4.2 )
STRING( REGEX MATCHALL "[0-9]+" KAUTHAM_VERSIONS ${KAUTHAM_VERSION} )
LIST( GET KAUTHAM_VERSIONS 0 KAUTHAM_VERSION_MAJOR)
LIST( GET KAUTHAM_VERSIONS 1 KAUTHAM_VERSION_MINOR)
LIST( GET KAUTHAM_VERSIONS 2 KAUTHAM_VERSION_PATCH)

message( "Kautham version ${VERSION} (${KAUTHAM_VERSION_MAJOR}.${KAUTHAM_VERSION_MINOR}.${KAUTHAM_VERSION_PATCH})" )

if(COMMAND cmake_policy)
    cmake_policy(SET CMP0003 NEW)
endif(COMMAND cmake_policy)


if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "Release" CACHE STRING
      "Choose the type of build, options are: Debug Release RelWithDebInfo MinSizeRel."
      FORCE)
endif(NOT CMAKE_BUILD_TYPE)

########### Options ####################
option( KAUTHAM_FCL "Use FCL" ON )
option( KAUTHAM_OMPL "Use OMPL planners" ON )
option( KAUTHAM_IOC "Use IOC planners" ON )
option( KAUTHAM_OPENDE "Use Open Dynamic Engine" OFF)
option( KAUTHAM_GUI "Build the gui version" ON )
option( KAUTHAM_CONSOLE "Build the console version" ON )
option( KAUTHAM_ROS "Build the ROS version" ON )
option( KAUTHAM_DOCUMENTATION_API "Create and install the HTML based API documentation (requires Doxygen)" OFF)

########### Set flags ####################
message( STATUS "Using this flags ........")
message( STATUS "Building the GUI version: ${KAUTHAM_GUI}")
message( STATUS "Building the console version: ${KAUTHAM_CONSOLE}")
message( STATUS "Building the ROS version: ${KAUTHAM_ROS}")
message( STATUS "Using OMPL planners: ${KAUTHAM_OMPL}")
message( STATUS "Using IOC planners: ${KAUTHAM_IOC}")
message( STATUS "Using Open Dynamic Engine: ${KAUTHAM_OPENDE}")
message( STATUS "Using FCL: ${KAUTHAM_FCL}")
message( STATUS "Creating the HTML based API documentation: ${KAUTHAM_DOCUMENTATION}")

########### Modules path and system options ###############
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/CMakeModules)
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

##############################################################################

##################### ROS-catkin ####################################


if( KAUTHAM_ROS )

    find_package(catkin REQUIRED COMPONENTS
      roscpp
      std_msgs
      trajectory_msgs
      message_generation
      roslib
      rospy
      geometry_msgs
      visualization_msgs
      tf2
      tf2_ros
    )
    include_directories(${catkin_INCLUDE_DIRS} include)

    add_service_files(
       DIRECTORY apps/ros/srv
       FILES
       AddObstacle.srv
       DetachObstacle.srv
       SetDefaultRobControls.srv
       SetObsControlsStream.srv
       SetQuery.srv
       AddRobot.srv
       GetPath.srv
       SetFixedObsControls.srv
       SetObstaclesConfig.srv
       SetRobControls.srv
       SetRobControlsNoQuery.srv
       AttachObstacle2RobotLink.srv
       OpenProblem.srv
       OpenProblemStream.srv
       SetGoal.srv
       SetPlannerByName.srv
       SetRobControlsStream.srv
       CheckCollision.srv
       ProblemOpened.srv
       SetInitObs.srv
       SetPlannerParameter.srv
       SetRobotsConfig.srv
       ClearSampleSet.srv
       RemoveObstacle.srv
       SetInit.srv
       SetPlanner.srv
       Solve.srv
       CloseProblem.srv
       RemoveRobot.srv
       SetObsControls.srv
       SetPlannerStream.srv
       Connect.srv
       GetLastPlanComputationTime.srv
       GetNumEdges.srv
       GetNumVertices.srv
       ObsPos.srv
       RobPos.srv
       FindIK.srv
       LoadRobots.srv
       VisualizeScene.srv
       LoadObstacles.srv
       GetObstaclesNames.srv
       PathDofNames.srv
    )

    add_message_files(
       DIRECTORY apps/ros/msg
       FILES
       fVector.msg
       robconf.msg
        )
    catkin_python_setup()
    generate_messages(DEPENDENCIES roscpp std_msgs trajectory_msgs  visualization_msgs )
    catkin_package(
      INCLUDE_DIRS include
      #LIBRARIES libkautham
      CATKIN_DEPENDS roscpp visualization_msgs trajectory_msgs std_msgs  geometry_msgs tf2 tf2_ros
    )

endif( KAUTHAM_ROS )
include_directories(${catkin_INCLUDE_DIRS})

##################### Qt-Creator ####################################

FILE(GLOB_RECURSE LibFiles include/*.h src/external/*.h src/external/gdiam/*.h src/external/pqp/*.h src/external/pugixml/*.h)
add_custom_target(headers SOURCES ${LibFiles})


#### Boost package ###############
set(Boost_USE_MULTITHREADED      ON)
find_package( Boost COMPONENTS  system serialization thread REQUIRED )
if(Boost_FOUND)
  message(STATUS "Boost library found at first attempt.")
  message(STATUS "Using Boost linking against ${Boost_LIBRARIES}")
  set(EXTRA_LIBS ${EXTRA_LIBS} ${Boost_LIBRARIES})
else(Boost_FOUND)
  set( BOOST_ROOT $ENV{BOOSTDIR})
        message(STATUS "Boost library NOT found at first attempt. Setting BOOST_ROOT to" $ENV{BOOSTDIR})
        find_package( Boost COMPONENTS  system serialization thread REQUIRED )
        if(Boost_FOUND)
                message(STATUS "Boost library found in " ${BOOST_ROOT})
                include_directories(${Boost_INCLUDE_DIRS})
                set(EXTRA_LIBS ${EXTRA_LIBS} ${Boost_LIBRARIES})
                link_directories( ${Boost_LIBRARY_DIRS} )
        else(Boost_FOUND)
                message(STATUS "Boost library not found. Boost-dependent headers will NOT be available")
        endif(Boost_FOUND)
endif(Boost_FOUND)

#### MT ###############
set(mt_DIR include/kautham/util/libmt)
message (STATUS "Adding libmt directory ${mt_DIR}")
include_directories( ${mt_DIR} )

### Eigen3 ############
find_package(Eigen3 REQUIRED NO_MODULE)
include_directories(${EIGEN3_INCLUDE_DIR})

#### ompl ######
# since noetic, ros provides ompl library version.
# but we prefer the Debian version
set(CMAKE_FIND_ROOT_PATH /usr/share;${CMAKE_FIND_ROOT_PATH})
find_package(ompl 1.5)

if( OMPL_FOUND )
   if(OMPL_VERSION  VERSION_LESS 1.3.0)
        message(ERROR "OMPL Package FOUND version ${OMPL_VERSION} and Kautham need at least 1.3")
   else(OMPL_VERSION  VERSION_LESS 1.3.0)
        message( STATUS "OMPL Package FOUND version ${OMPL_VERSION} : setting -DKAUTHAM_USE_OMPL flag")
        message( STATUS "OMPL lib is: " ${OMPL_LIBRARIES})
   endif(OMPL_VERSION  VERSION_LESS 1.3.0)

   message( STATUS "OMPL include is: " ${OMPL_INCLUDE_DIRS})
   include_directories(${OMPL_INCLUDE_DIRS})
   set(EXTRA_LIBS ${EXTRA_LIBS} ${OMPL_LIBRARIES})
   add_definitions( -DKAUTHAM_USE_OMPL )
endif()


##################### ROS-catkin ####################################

include_directories( include )
include_directories( ${CMAKE_BINARY_DIR}/include/)

include_directories( src )  # for external


# Building librarires
add_subdirectory ( include )
add_subdirectory ( src )
add_subdirectory ( apps )
add_subdirectory ( doc )

#########################################################
# Copy Help file
# install(DIRECTORY include/kautham DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
install(DIRECTORY demos DESTINATION share/kautham)
