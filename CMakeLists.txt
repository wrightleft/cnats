cmake_minimum_required(VERSION 2.8)

project(cnats)
include(CTest)

# Uncomment to have the build process verbose
#set(CMAKE_VERBOSE_MAKEFILE TRUE)

# Uncomment to have the executable moved to 'build' instead of their respective 'build/xxx' directories
#set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR})

# Platform specific settings
if(UNIX)
  #---------------------------------------------------------------------------
  # Define NATS cache variables that override the CMAKE and MEMCHECK variables
  #---------------------------------------------------------------------------
  set(NATS_BUILD_TYPE Release CACHE STRING "Build type: Release, Debug, RelWithDebInfo, MinRelSize")
  set(CMAKE_BUILD_TYPE ${NATS_BUILD_TYPE} CACHE INTERNAL "")

  set(NATS_BUILD_ARCH "64" CACHE STRING "32 for 32bits builds")

  set(NATS_INSTALL_PREFIX ../install CACHE PATH "Install prefix")
  set(CMAKE_INSTALL_PREFIX ${NATS_INSTALL_PREFIX} CACHE INTERNAL "")

  set(NATS_MEMCHECK_CMD "/usr/bin/valgrind" CACHE FILE "Memcheck tool")
  set(MEMORYCHECK_COMMAND ${NATS_MEMCHECK_CMD} CACHE INTERNAL "")

  set(NATS_MEMCHECK_CMD_OPTS "--leak-check=full --track-fds=yes --show-reachable=yes --num-callers=50" CACHE STRING "Memcheck options")
  set(MEMORYCHECK_COMMAND_OPTIONS ${NATS_MEMCHECK_CMD_OPTS} CACHE INTERNAL "")

  set(NATS_SUPPRESSIONS_FILE ${PROJECT_SOURCE_DIR}/test/valgrind.sup CACHE FILE "Suppressions file")
  set(MEMORYCHECK_SUPPRESSIONS_FILE ${NATS_SUPPRESSIONS_FILE} CACHE INTERNAL "")

  set(NATS_COMMON_C_FLAGS "-std=c99 -pedantic")
  
  if(NATS_BUILD_TYPE MATCHES "Debug")
    set(NATS_COMMON_C_FLAGS "${NATS_COMMON_C_FLAGS} -ggdb")    
  endif(NATS_BUILD_TYPE MATCHES "Debug")
  
  set(NATS_WARNINGS "-Wall -W -Wno-unused-variable -Wno-unused-parameter -Wno-unused-function -Wstrict-prototypes -Wwrite-strings")
  set(NATS_PLATFORM_INCLUDE "unix")
  
  if(APPLE)
    set(CMAKE_MACOSX_RPATH OFF)
    set(NATS_OS "DARWIN")
  else(APPLE)
    set(NATS_OS "LINUX")
    set(NATS_USE_PTHREAD "-pthread")
  endif(APPLE)
  
  if (${NATS_BUILD_ARCH} MATCHES "32")
    if(NOT APPLE)
      message("-----------------------------------------------------------")
      message("If build fails, you probably need to install libc6-dev-i386")
      message("apt-get install libc6-dev-i386")
      message("-----------------------------------------------------------")
    endif(NOT APPLE)
      
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -m32")
    set(CMAKE_C_LINKER_FLAGS "${CMAKE_C_LINKER_FLAGS} -m32")
  endif(${NATS_BUILD_ARCH} MATCHES "32")
  
elseif(WIN32)
  set(NATS_OS "_WIN32")
  set(NATS_PLATFORM_INCLUDE "win") 
endif(UNIX)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${NATS_COMMON_C_FLAGS} ${NATS_USE_PTHREAD} ${NATS_WARNINGS}") 

add_definitions(-D${NATS_OS})
add_definitions(-D_REENTRANT)

#---------------------------------------------------------------------
# Add to the 'clean' target the list (and location) of files to remove

list(APPEND NATS_INSTALLED_FILES "${CMAKE_INSTALL_PREFIX}/include/nats.h")
list(APPEND NATS_INSTALLED_FILES "${CMAKE_INSTALL_PREFIX}/include/status.h")
list(APPEND NATS_INSTALLED_FILES "${CMAKE_INSTALL_PREFIX}/lib/${CMAKE_STATIC_LIBRARY_PREFIX}nats_static${CMAKE_STATIC_LIBRARY_SUFFIX}")
list(APPEND NATS_INSTALLED_FILES "${CMAKE_INSTALL_PREFIX}/lib/${CMAKE_SHARED_LIBRARY_PREFIX}nats${CMAKE_SHARED_LIBRARY_SUFFIX}")

set_directory_properties(PROPERTIES ADDITIONAL_MAKE_CLEAN_FILES "${NATS_INSTALLED_FILES}")
#---------------------------------------------------------------------

#----------------------------
# Add the project directories

add_subdirectory(src)
add_subdirectory(examples)
add_subdirectory(test)
#----------------------------
