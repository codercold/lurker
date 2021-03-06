CMAKE_MINIMUM_REQUIRED(VERSION 2.8)
PROJECT(Lurker)

if(NOT CMAKE_BUILD_TYPE)
    SET(CMAKE_BUILD_TYPE "Release")
endif()
SET(CMAKE_CXX_FLAGS_RELEASE "-Wall -O2 -std=c++0x")
SET(CMAKE_CXX_FLAGS_DEBUG   "-Wall -O0 -std=c++0x -g")
SET(LIBRARY_OUTPUT_PATH    ${PROJECT_BINARY_DIR}/lib)
SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)

option(WITH_FLUENT "libfluent prefix" ".")
SET(FLUENT_INCLUDES  ${WITH_FLUENT}/src)
SET(FLUENT_LIBRARIES ${WITH_FLUENT}/lib)

cmake_policy(SET CMP0015 NEW)

INCLUDE_DIRECTORIES(${INC_DIR} ${FLUENT_INCLUDES} ./src)
LINK_DIRECTORIES(${LIB_DIR} ${FLUENT_LIBRARIES})
FILE(GLOB BASESRCS "src/*.cc"
	  "src/swarm/*.cc" "src/swarm/*.h" "src/swarm/swarm/*.h" 
	  "src/swarm/utils/*.cc" "src/swarm/utils/*.h"
	  "src/swarm/proto/*.cc" "src/swarm/proto/*.h"
)
FILE(GLOB TESTSRCS "test/*.cc")

# Module code
ADD_LIBRARY(lurker SHARED ${BASESRCS})
TARGET_LINK_LIBRARIES(lurker fluent pcap ev)

# Test code
ADD_EXECUTABLE(lurker-test ${TESTSRCS})
TARGET_LINK_LIBRARIES(lurker-test lurker pthread)

# Application (CLI) code
ADD_EXECUTABLE(lurker-bin cli/main.cc cli/optparse.cc)
SET_TARGET_PROPERTIES(lurker-bin PROPERTIES OUTPUT_NAME lurker)
TARGET_LINK_LIBRARIES(lurker-bin lurker)

INSTALL(TARGETS lurker-bin RUNTIME DESTINATION bin)
INSTALL(TARGETS lurker LIBRARY DESTINATION lib)
