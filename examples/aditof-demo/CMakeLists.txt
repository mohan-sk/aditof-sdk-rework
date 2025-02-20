cmake_minimum_required(VERSION 2.8)
project(aditof-demo)

find_package(OpenCV)
if(OPENCV_FOUND OR
   OPENCV_CORE_FOUND) # we check for this because the OpenCVConfig.cmake(v2.4.9.1) used on dragonboards fails to set OPENCV_FOUND
	if(OpenCV_VERSION VERSION_LESS "3.0.0")
		message(STATUS "USING OPENCV Version 2")
		add_definitions(-DOPENCV2)
	endif(OpenCV_VERSION VERSION_LESS "3.0.0")
else()
	if (NOT OpenCV_INCLUDE_DIR OR NOT OpenCV_LIBS)
		message(FATAL_ERROR "find_package(openCV) failed. OpenCV_INCLUDE_DIR and OpenCV_LIBS need to be set.")
	else()
		message(STATUS "OpenCV_INCLUDE_DIR and OpenCV_LIBS have been set manually.")
	endif()
endif()

if( APPLE )
    add_definitions(-DDEVICETYPE="ETH")
elseif( DRAGONBOARD OR RASPBERRYPI )
    add_definitions(-DDEVICETYPE="Local")
elseif( WIN32 OR UNIX )
     add_definitions(-DDEVICETYPE="USB ETH")
endif()

add_executable(${PROJECT_NAME} 
                        aditofdemorecorder.cpp
			aditofdemoview.cpp
			aditofdemocontroller.cpp
			main.cpp
)
target_include_directories(${PROJECT_NAME} PRIVATE ${OpenCV_INCLUDE_DIR})

set_target_properties(${PROJECT_NAME} PROPERTIES CXX_STANDARD 11)
target_link_libraries(${PROJECT_NAME} PRIVATE aditof ${OpenCV_LIBS})

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/config_pipe.sh
               ${CMAKE_CURRENT_BINARY_DIR}/config_pipe.sh
               COPYONLY)

if (NOT WIN32)
	find_library(PTHREAD_LIBRARIES pthread)
	if (PTHREAD_LIBRARIES)
                target_link_libraries(${PROJECT_NAME} PRIVATE ${PTHREAD_LIBRARIES})
	else()
		message(FATAL_ERROR "pthread libraries not found!")
	endif()
endif()

add_custom_command( TARGET ${PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_directory ${LIBTOFI_LIBDIR_PATH} $<TARGET_FILE_DIR:${PROJECT_NAME}>/.
    COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_CURRENT_SOURCE_DIR}/../../sdk/src/cameras/itof-camera/config $<TARGET_FILE_DIR:${PROJECT_NAME}>/${CONFIG_DIR_NAME}/.
)

if( WIN32 )
    add_custom_command( TARGET ${PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_if_different $<TARGET_FILE:aditof> $<TARGET_FILE_DIR:${PROJECT_NAME}>/.
    COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_CURRENT_SOURCE_DIR}/../../sdk/src/cameras/itof-camera/config $<TARGET_FILE_DIR:${PROJECT_NAME}>/../${CONFIG_DIR_NAME}/.
    )
endif()
