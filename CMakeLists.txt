cmake_minimum_required(VERSION 3.1)
# note: cxx-17 requires cmake 3.8, cxx-20 requires cmake 3.12
project(fleet-provisioning CXX)

if ((DEFINED ENV{CROSS_COMPILE}) OR (CMAKE_CROSSCOMPILING))
    SET(INFINITY_TARGET_ENV_BOARD 1)
    SET(INFINITY_TARGET_SYS "YOCTO")
    SET(INFINITY_TARGET_SYS_YOCTO 1)
else()
    SET(INFINITY_TARGET_ENV_DEV 1)
    if (APPLE)
        SET(INFINITY_TARGET_SYS "OSX")
        SET(INFINITY_TARGET_SYS_OSX 1)
    else()
        SET(INFINITY_TARGET_SYS "LINUX")
        SET(INFINITY_TARGET_SYS_LINUX 1)
    endif()
endif()

add_definitions(-DINFINITY_TARGET_SYS=${INFINITY_TARGET_SYS})
add_definitions(-DINFINITY_TARGET_SYS_${INFINITY_TARGET_SYS}=1)

if (INFINITY_TARGET_SYS_LINUX)
    set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/artifacts/linux/install/lib ${CMAKE_MODULE_PATH})
    set(CMAKE_PREFIX_PATH ${CMAKE_SOURCE_DIR}/artifacts/linux/install/lib ${CMAKE_PREFIX_PATH})
    set(CMAKE_SYSTEM_LIBRARY_PATH ${CMAKE_SOURCE_DIR}/artifacts/linux/install/lib ${CMAKE_SYSTEM_LIBRARY_PATH})
    set(CMAKE_SYSTEM_INCLUDE_PATH ${CMAKE_SOURCE_DIR}/artifacts/linux/install/include ${CMAKE_SYSTEM_INCLUDE_PATH})

    #include_directories(${CMAKE_SOURCE_DIR}/artifacts/linux/install/include)
    link_directories(${CMAKE_SOURCE_DIR}/artifacts/linux/install/lib)
elseif (INFINITY_TARGET_SYS_YOCTO)
    #forza l'uso delle librerie dinamiche aws
    #set(BUILD_SHARED_LIBS ON)
endif()

file(GLOB SRC_FILES
       "*.cpp"
       "CommandLineUtils.cpp"
       "utils/CommandLineUtils.h"
)

add_executable(${PROJECT_NAME} ${SRC_FILES})

set_target_properties(${PROJECT_NAME} PROPERTIES
    CXX_STANDARD 14)

#set warnings
target_compile_options(${PROJECT_NAME} PRIVATE -Wall -Wno-long-long -pedantic -Werror)

find_package(aws-crt-cpp REQUIRED)
find_package(IotIdentity-cpp REQUIRED)

install(TARGETS ${PROJECT_NAME} DESTINATION bin)

target_link_libraries(${PROJECT_NAME} PRIVATE AWS::aws-crt-cpp AWS::IotIdentity-cpp)