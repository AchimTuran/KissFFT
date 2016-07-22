cmake_minimum_required(VERSION 3.0)

project(KissFFT VERSION 1.30) # Define KissFFT workspace with specific version.

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

include(cmake/UseMultiArch.cmake)

if(NOT WIN32)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
endif()

# Set KissFFT source and header files.
set(SOURCES "kiss_fft.c"
            "tools/kiss_fftr.c")

set(HEADERS "kissfft.hh"
            "kiss_fft.h"
            "_kiss_fft_guts.h"
            "tools/kiss_fftr.h")

# Create KissFFT static library.
add_library(${PROJECT_NAME} STATIC ${SOURCES} ${HEADERS})

# Define KissFFT namespace, which is very useful for exporting the KissFFT target.
# ------------------------------------------------------------------------------------------------
# The alias target can be used within the project and ensures with its :: name that 
# target_link_libraries fails already at configure time when the target is not here. 
# Without that, cmake would just assume that it's a system lib and just adds -llib to the 
# linker flag, which would only show errors at link time.
add_library(${PROJECT_NAME}::${PROJECT_NAME} ALIAS ${PROJECT_NAME})

# set KissFFT target include directories
# ------------------------------------------------------------------------------------------------
# PRIVATE, PUBLIC, INTERFACE is best explained here:
# https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html
# PRIVATE   = use it for compiling/linking this library
# INTERFACE = propagate it to users of the library, but don't use it for this library
# PUBLIC    = both
#
# Rule of thumb for target_include_directories(...) is:
# - If you use the dependecy only in your cpp files or private headers --> PRIVATE
# - If you use it in public headers -> INTERFACE
# - If both: PUBLIC
target_include_directories(${PROJECT_NAME}
                           PRIVATE   ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/tools
                           INTERFACE $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/..>
                                     $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
                                     $<INSTALL_INTERFACE:include>
                                     $<INSTALL_INTERFACE:include/KissFFT>)

# Add this project to Visual Studio solution folder "contrib".
set_property(TARGET KissFFT PROPERTY FOLDER "contrib")

# Install KissFFT header files.
foreach(_header ${HEADERS})
  install(FILES ${_header} DESTINATION ${CMAKE_INSTALL_PREFIX}/include/KissFFT)
endforeach()

# Export KissFFT target.
install(TARGETS ${PROJECT_NAME} EXPORT ${PROJECT_NAME} DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)

# Include configure helper module.
include(CMakePackageConfigHelpers)

# Set KissFFT configure files install path.
set(CMAKE_INSTALL_CMAKECONFIGDIR
    ${CMAKE_INSTALL_LIBDIR_NOARCH}/cmake/${PROJECT_NAME}-${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR})

# Generate and install basic KissFFT version file, which can be used by find_package(...).
write_basic_package_version_file(${PROJECT_NAME}ConfigVersion.cmake COMPATIBILITY SameMajorVersion)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
        DESTINATION ${CMAKE_INSTALL_CMAKECONFIGDIR})

# Generate and install KissFFT configure file that is used by find_package(...).
# This file makes an find module obsolute, if your project
# sets CMAKE_PREFIX_PATH to the install path of KissFFT.
configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/${PROJECT_NAME}Config.cmake.in ${PROJECT_NAME}Config.cmake
                              INSTALL_DESTINATION ${CMAKE_INSTALL_CMAKECONFIGDIR})
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
        DESTINATION ${CMAKE_INSTALL_CMAKECONFIGDIR})

# Export all targets that are included in the namespace KissFFT.
# Furthermore this will generate the necessary code for KissFFT 
# import as a CMake target for free.
install(EXPORT ${PROJECT_NAME} FILE ${PROJECT_NAME}-targets.cmake NAMESPACE ${CMAKE_PROJECT_NAME}::
        DESTINATION ${CMAKE_INSTALL_CMAKECONFIGDIR})
