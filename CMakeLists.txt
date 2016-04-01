project(KissFFT)

cmake_minimum_required(VERSION 2.6)
enable_language(CXX)

set(KissFFT_VERSION_MAJOR 1)
set(KissFFT_VERSION_MINOR 30)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

include(UseMultiArch.cmake)
#include(CheckAtomic.cmake)

if(NOT WIN32)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
endif()

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/tools
                    ${CMAKE_CURRENT_SOURCE_DIR})
                    
set(KissFFT_SOURCES "kiss_fft.c"
                    "tools/kiss_fftr.c")

set(KissFFT_HEADERS "kissfft.hh"
                    "kiss_fft.h"
                    "_kiss_fft_guts.h"
                    "tools/kiss_fftr.h")

add_library(KissFFT STATIC ${KissFFT_SOURCES} ${KissFFT_HEADERS})

set_property(TARGET KissFFT PROPERTY FOLDER "contrib")

message(STATUS "CMAKE_BINARY_DIR=${CMAKE_BINARY_DIR}") #sources path
message(STATUS "CMAKE_SOURCE_DIR=${CMAKE_SOURCE_DIR}") #project file path

#include_directories(${CMAKE_SOURCE_DIR})

#target_link_libraries(kodiplatform ${kodiplatform_LIBRARIES} ${p8-platform_LIBRARIES})
set_target_properties(KissFFT PROPERTIES VERSION    ${KissFFT_VERSION_MAJOR}.${KissFFT_VERSION_MINOR}
                                         SOVERSION  ${KissFFT_VERSION_MAJOR}.0)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/KissFFT-config-version.cmake"
  VERSION "${KissFFT_VERSION_MAJOR}.${KissFFT_VERSION_MINOR}"
  COMPATIBILITY ExactVersion)

#set(CMAKE_INSTALL_LIBDIR lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR})  
message(STATUS "CMAKE_LIBRARY_OUTPUT_DIRECTORY=${CMAKE_LIBRARY_OUTPUT_DIRECTORY}")
install(FILES ${CMAKE_BINARY_DIR}/KissFFT-config-version.cmake
        DESTINATION ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/KissFFT)
                                              
install(TARGETS KissFFT DESTINATION ${CMAKE_INSTALL_LIBDIR})

set(KissFFT_LIBRARIES KissFFT)

foreach(_header ${KissFFT_HEADERS})
  install(FILES ${_header} DESTINATION ${CMAKE_INSTALL_PREFIX}/include/KissFFT)
endforeach()

IF(NOT WIN32)
  # Pkgconfig
  include(PkgConfigHandler.cmake)
  configure_pc_file(KissFFT   KissFFT.pc.in
                              KissFFT.pc
                              ${CMAKE_INSTALL_PREFIX}
                              ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/KissFFT
                              ${CMAKE_INSTALL_PREFIX}/include)

  install(FILES ${CMAKE_BINARY_DIR}/KissFFT.pc
          DESTINATION ${CMAKE_INSTALL_PREFIX}/pkgconfig)
ENDIF(NOT WIN32)

# config mode
configure_file (KissFFT-config.cmake.in
                KissFFT-config.cmake @ONLY)
install(FILES ${CMAKE_BINARY_DIR}/KissFFT-config.cmake
        DESTINATION ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/KissFFT)
