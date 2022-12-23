#cmake_minimum_required(VERSION 3.5)

#include_directories(${CMAKE_CURRENT_SOURCE_DIR})

#FILE(GLOB SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
FILE(GLOB INCLUDES ${CMAKE_CURRENT_SOURCE_DIR}/*.h)
FILE(GLOB INCLUDESPP ${CMAKE_CURRENT_SOURCE_DIR}/*.hpp)

add_library(CosmosSupport STATIC cosmos-defs.h configCosmosKernel.h)
set_target_properties(CosmosSupport PROPERTIES LINKER_LANGUAGE CXX)
install(TARGETS CosmosSupport DESTINATION lib)
#target_link_libraries(CosmosSupport CosmosMath)
target_link_libraries(CosmosSupport)

add_library(CosmosSlip STATIC sliplib.cpp sliplib.h)
install(TARGETS CosmosSlip DESTINATION lib)

add_library(CosmosError STATIC cosmos-errno.h)
set_target_properties(CosmosError PROPERTIES LINKER_LANGUAGE CXX)
install(TARGETS CosmosError DESTINATION lib)
target_link_libraries(CosmosError CosmosSupport)

add_library(CosmosPacket STATIC packetcomm.cpp packetcomm.h)
install(TARGETS CosmosPacket DESTINATION lib)
target_link_libraries(CosmosPacket CosmosDeviceGeneral CosmosMath CosmosSlip CosmosError CosmosSupport)

install(FILES ${INCLUDES} DESTINATION include/support)
install(FILES ${INCLUDESPP} DESTINATION include/support)

if(${DEPLOY_SRC_FILES_TO_REMOTE} MATCHES "YES")
    add_deployment_source_directory(${CMAKE_CURRENT_SOURCE_DIR} "src")
endif()
