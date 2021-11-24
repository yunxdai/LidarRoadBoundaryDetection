cmake_minimum_required(VERSION 2.8)

project(boundary_detection)

set(CMAKE_BUILD_TYPE Release)
set(CMAKE_CXX_FLAGS "-std=c++11")
# set(CMAKE_BUILD_TYPE Release CACHE STRING "set build type to release")
# set (CMAKE_CXX_FLAGS " -march=native ${CMAKE_CXX_FLAGS} -g")
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_LIST_DIR}/cmake")
set(LIMBO_INCLUDE_DIRS ./limbo/src)

find_package(OpenMP REQUIRED)
find_package(Protobuf REQUIRED)
find_package(Boost)
find_package(PCL REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(OpenCV REQUIRED)
find_package(Glog REQUIRED) 

# message(${OpenCV_INCLUDE_DIRS})

# PROTO
# function(get_pb_file H CPP)
# 	list(APPEND PH ${ARGN})
# 	list(APPEND PC ${ARGN})
# 	list(TRANSFORM PH PREPEND ${CMAKE_BINARY_DIR}/proto/)
# 	list(TRANSFORM PH APPEND .pb.h)
# 	list(TRANSFORM PC PREPEND ${CMAKE_BINARY_DIR}/proto/)
# 	list(TRANSFORM PC APPEND .pb.cc)
# 	set(${H} ${PH} PARENT_SCOPE)
# 	set(${CPP} ${PC} PARENT_SCOPE)
# endfunction()
# SET(PROTO_META_BASE_DIR ${CMAKE_CURRENT_BINARY_DIR}/proto)
# file(MAKE_DIRECTORY ${PROTO_META_BASE_DIR})
# LIST(APPEND PROTO_FLAGS -I${CMAKE_CURRENT_SOURCE_DIR}/proto)
# file(GLOB PROTO_FILES "${CMAKE_SOURCE_DIR}/proto/*.proto")
# # protobuf_generate_cpp(PROTO_CPP PROTO_H ${PROTO_FILES})
# FOREACH(FIL ${PROTO_FILES})
# 	GET_FILENAME_COMPONENT(ABS_FIL ${FIL} ABSOLUTE)
# 	GET_FILENAME_COMPONENT(FIL_WE ${ABS_FIL} NAME_WE)
# 	# LIST(APPEND PROTO_CPP "${CMAKE_CURRENT_BINARY_DIR}/${FIL_WE}.pb.cc")
# 	# LIST(APPEND PROTO_H "${CMAKE_CURRENT_BINARY_DIR}/${FIL_WE}.pb.h")
# 	EXECUTE_PROCESS(
# 		COMMAND ${PROTOBUF_PROTOC_EXECUTABLE} ${PROTO_FLAGS} --cpp_out=${PROTO_META_BASE_DIR} ${FIL}
# 		WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
# 	)
# ENDFOREACH()

# INCLUDE
INCLUDE_DIRECTORIES(
    ${PCL_INCLUDE_DIRS}
    ${OpenCV_INCLUDE_DIRS}  
    ${Qt5Charts_INCLUDE_DIRS}
    ${EIGEN3_INCLUDE_DIRS} 
    ${LIMBO_INCLUDE_DIRS}
    ${Protobuf_INCLUDE_DIRS}
    ${GLOG_INCLUDE_DIRS}
    ${CMAKE_SOURCE_DIR}
    ${CMAKE_BINARY_DIR}/proto
)

LINK_LIBRARIES(
    ${PCL_LIBRARY_DIRS}
    ${PROTOBUF_LIBRARIES} 
)

# OPENMP
if(NOT TARGET OpenMP::OpenMP_CXX)
    find_package(Threads REQUIRED)
    add_library(OpenMP::OpenMP_CXX IMPORTED INTERFACE)
    set_property(TARGET OpenMP::OpenMP_CXX
                 PROPERTY INTERFACE_COMPILE_OPTIONS ${OpenMP_CXX_FLAGS})
    # Only works if the same flag is passed to the linker; use CMake 3.9+ otherwise (Intel, AppleClang)
    set_property(TARGET OpenMP::OpenMP_CXX
                 PROPERTY INTERFACE_LINK_LIBRARIES ${OpenMP_CXX_FLAGS} Threads::Threads)

endif()

# add_definitions(${PCL_DEFINITIONS})


set (PROTO_CPP ${CMAKE_SOURCE_DIR}/proto/road_boundary.pb.cc)
set (PROTO_H ${CMAKE_SOURCE_DIR}/proto/road_boundary.pb.h)


# LIBRARY
file(GLOB LIBFILES 
    "${CMAKE_SOURCE_DIR}/lib/*.cpp"
)
add_library( extract
    ${LIBFILES}
    ${PROTO_CPP}
    ${PROTO_H}
)

target_link_libraries( extract 
    OpenMP::OpenMP_CXX
    ${PCL_LIBRARIES}
    ${OpenCV_LIBS} 
    ${Protobuf_LIBRARIES}
    ${GLOG_LIBRARIES}
)

# MAIN SOURCE
add_executable(demo_frame src/main.cpp)
target_link_libraries(demo_frame ${PCL_LIBRARIES} ${OpenCV_LIBS} extract
${Protobuf_LIBRARIES} ${GLOG_LIBRARIES})


add_executable(demo_video src/main_thread.cpp )

target_link_libraries(demo_video ${PCL_LIBRARIES} ${OpenCV_LIBS} extract
${Protobuf_LIBRARIES} ${GLOG_LIBRARIES})
