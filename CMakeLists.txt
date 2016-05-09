cmake_minimum_required(VERSION 3.0)
project(KissFFT VERSION 1.30)

#enable_language(CXX)

# set(KissFFT_VERSION_MAJOR 1)
# set(KissFFT_VERSION_MINOR 30)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

include(cmake/UseMultiArch.cmake)
#include(cmake/CheckAtomic.cmake)

if(NOT WIN32)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
endif()

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/tools
                    ${CMAKE_CURRENT_SOURCE_DIR})
                    
set(SOURCES "kiss_fft.c"
            "tools/kiss_fftr.c")

set(HEADERS "kissfft.hh"
            "kiss_fft.h"
            "_kiss_fft_guts.h"
            "tools/kiss_fftr.h")

add_library(${PROJECT_NAME} STATIC ${SOURCES} ${HEADERS})
add_library(${PROJECT_NAME}::${PROJECT_NAME} ALIAS ${PROJECT_NAME})
target_include_directories(${PROJECT_NAME}
                           PRIVATE ${PROJECT_NAME}
                           INTERFACE $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
                                     $<INSTALL_INTERFACE:include>
                                     $<INSTALL_INTERFACE:include/KissFFT>)

set_property(TARGET KissFFT PROPERTY FOLDER "contrib")

set_target_properties(KissFFT PROPERTIES VERSION    ${KissFFT_VERSION_MAJOR}.${KissFFT_VERSION_MINOR}
                                         SOVERSION  ${KissFFT_VERSION_MAJOR}.0)

include(CMakePackageConfigHelpers)
write_basic_package_version_file("${CMAKE_CURRENT_BINARY_DIR}/KissFFTConfigVersion.cmake"
                                 VERSION "${KissFFT_VERSION_MAJOR}.${KissFFT_VERSION_MINOR}"
                                 COMPATIBILITY ExactVersion)

foreach(_header ${HEADERS})
  install(FILES ${_header} DESTINATION ${CMAKE_INSTALL_PREFIX}/include/KissFFT)
endforeach()

install(TARGETS ${PROJECT_NAME} EXPORT ${PROJECT_NAME} DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)


include(CMakePackageConfigHelpers)

set(CMAKE_INSTALL_CMAKECONFIGDIR
    ${CMAKE_INSTALL_LIBDIR_NOARCH}/cmake/${PROJECT_NAME}-${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR})

write_basic_package_version_file(${PROJECT_NAME}ConfigVersion.cmake COMPATIBILITY SameMajorVersion)
install(FILES ${CMAKE_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
        DESTINATION ${CMAKE_INSTALL_CMAKECONFIGDIR})

configure_package_config_file(cmake/${PROJECT_NAME}Config.cmake.in ${PROJECT_NAME}Config.cmake
                              INSTALL_DESTINATION ${CMAKE_INSTALL_CMAKECONFIGDIR})
                              #PATH_VARS INCLUDE_INSTALL_DIR)
install(FILES ${CMAKE_BINARY_DIR}/${PROJECT_NAME}Config.cmake
        DESTINATION ${CMAKE_INSTALL_CMAKECONFIGDIR})

install(EXPORT ${PROJECT_NAME} FILE ${PROJECT_NAME}-targets.cmake NAMESPACE ${CMAKE_PROJECT_NAME}::
        DESTINATION ${CMAKE_INSTALL_CMAKECONFIGDIR})
