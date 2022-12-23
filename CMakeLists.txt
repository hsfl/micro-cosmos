message ("----------------------------------------------------------")
message ("Build micro-cosmos")
#--------------------------------------------------------------------
# Windows/MSVC2013 notes:
# * Near the end of this script are some replacement lines to select the
#   MSVC static runtime. Turn USE_MSVC_STATIC_RUNTIME to OFF to
#   use the default "multithreaded DLL" style of RTL
# * For x64 linkage give the "-A x64" option to the CMAKE command
#--------------------------------------------------------------------
option(USE_MSVC_STATIC_RUNTIME "USE Static Runtime with MSVC" "OFF")

#--------------------------------------------------------------------
# user config variables
#--------------------------------------------------------------------

# define the project name
set(PROJECT_NAME micro-cosmos)

# configure the COSMOS/core project here!
# include(${CMAKE_SOURCE_DIR}/cmake/config.cmake)

# Type of build
set(CMAKE_BUILD_TYPE Debug) # Release or Debug

# Type of architecture (arm, ...)
add_definitions( -DCROSS_TYPE_${CROSS_TYPE})
message(STATUS "Architecture: ${CROSS_TYPE}")

# define folder where your programs are going to be installed
#set(INSTALL_DIR run)

# by default


#--------------------------------------------------------------------
# automatic config

set(CROSS_TYPE x86_64)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED YES)
set(CMAKE_CXX_EXTENSIONS OFF)
message("Setting Standard to c++11")

# comment
#set(CMAKE_LEGACY_CYGWIN_WIN32 0) # Remove when CMake >= 2.8.4 is required
cmake_minimum_required(VERSION 3.1)
#cmake_minimum_required(VERSION 3.3) # if ( ... IN_LIST ...)

project(${PROJECT_NAME})

#Note: Actual flags are COSMOS_CXX_FLAGS + COSMOS_CXX_FLAGS_RELEASE/DEBUG?
set(COSMOS_CXX_FLAGS "-Wmaybe-uninitialized -Wtype-limits -Wall -pedantic -std=c++11 -U__STRICT_ANSI__ -Wno-invalid-offsetof -Wno-format")

# Force strict ANSI
#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -U__STRICT_ANSI__")

# Debugging?
if (CMAKE_BUILD_TYPE MATCHES "Debug")
    if (MSVC)
        set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MTd")
    else()
        if (${CROSS_TYPE} MATCHES "arm")
            set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -gdwarf-2")
        else()
            set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -g")
        endif()
    endif()
    message("Build Settings: Debug ON")
endif()

option(BUILD_STATIC "Build Static Executables" OFF)
if (BUILD_STATIC AND NOT MSVC)
    message("Build Settings: static executables ON")
    set(BUILD_SHARED_LIBS OFF)
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -static -static-libgcc -static-libstdc++ -Wl,--no-as-needed")
endif ()

# compile on Linux host
IF(${CMAKE_HOST_SYSTEM_NAME} MATCHES "Linux")

MESSAGE("Compiling on Linux for ${CROSS_TYPE}")

#Check for ARM Build
IF (${CROSS_TYPE} MATCHES "arm" OR ${CROSS_TYPE} MATCHES "iobc" OR ${CROSS_TYPE} MATCHES "rpi")

# ARM Build Detected

# Set installation directory
IF("$ENV{COSMOS}" STREQUAL "")
SET(COSMOS "~/cosmos/${CROSS_TYPE}")
message("ARM Default COSMOS: ${COSMOS}")
ELSE()
SET(COSMOS "$ENV{COSMOS}/${CROSS_TYPE}")
message("ARM Environment COSMOS: ${COSMOS}")
ENDIF("$ENV{COSMOS}" STREQUAL "")

# Set Toolchain File
SET(CMAKE_TOOLCHAIN_FILE "${PROJECT_SOURCE_DIR}/cmake/toolchain_${CROSS_TYPE}")

# Add preprocessor definition to help source files determine code to use for ARMv7 specific implementation
add_definitions(-DCROSS_TYPE_arm) #20150825JC: Added definition

# Add include directories for locally compiled arm openssl
include_directories(${PROJECT_SOURCE_DIR}/thirdparty/arm/include)

ELSE()

# Non-ARM Build Detected

IF("$ENV{COSMOS}" STREQUAL "")
SET(COSMOS "~/cosmos")
ELSE()
SET(COSMOS "$ENV{COSMOS}")
ENDIF("$ENV{COSMOS}" STREQUAL "")

ENDIF()

ENDIF(${CMAKE_HOST_SYSTEM_NAME} MATCHES "Linux")


# Establish includes, both standard and device specific
# Standard inludes
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/device)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/math)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/support)

add_subdirectory(${PROJECT_SOURCE_DIR}/device/general)
add_subdirectory(${PROJECT_SOURCE_DIR}/math)
add_subdirectory(${PROJECT_SOURCE_DIR}/support)
