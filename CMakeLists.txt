cmake_minimum_required(VERSION 3.5)

project(qpOASES
  LANGUAGES CXX
  VERSION 3.2.0
)

set(TARGET_NAME ${PROJECT_NAME}Targets)

# Project options
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
endif(NOT CMAKE_BUILD_TYPE)
option(QPOASES_BUILD_EXAMPLES "Build examples." OFF)
option(QPOASES_AVOID_LA_NAMING_CONFLICTS "If ON, avoid to re-defined symbols that conflict with Blas/Lapack provided functions." OFF)

############################################################
#################### build and install #####################
############################################################

# compile qpOASES libraries
file(GLOB SRC src/*.cpp)

# library
add_library(qpOASES ${SRC})
target_include_directories(qpOASES PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
)
target_compile_definitions(qpOASES PRIVATE
  __NO_COPYRIGHT__
  $<$<BOOL:${QPOASES_AVOID_LA_NAMING_CONFLICTS}>:__AVOID_LA_NAMING_CONFLICTS__>
  $<$<BOOL:${UNIX}>:LINUX>
  $<$<BOOL:${WIN32}>:WIN32>
)

if(UNIX)
  target_compile_options(qpOASES PRIVATE
    -O3 -finline-functions
  )
elseif(WIN32)
  target_compile_options(qpOASES PRIVATE
    -nologo -EHsc
  )
endif(UNIX)

set(INSTALL_CMAKEDIR share/cmake/${PROJECT_NAME})

install(
  TARGETS
    ${PROJECT_NAME}
  EXPORT ${TARGET_NAME}
  LIBRARY
    COMPONENT lib
  ARCHIVE
    COMPONENT lib
)

install(
  DIRECTORY
    "${CMAKE_SOURCE_DIR}/include/"
  TYPE INCLUDE
)

install(
  EXPORT
    ${TARGET_NAME}
  NAMESPACE
    "${PROJECT_NAME}::"
  DESTINATION
    ${INSTALL_CMAKEDIR}
  COMPONENT
    dev
)

include(CMakePackageConfigHelpers)

write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
  VERSION ${PROJECT_VERSION}
      COMPATIBILITY SameMajorVersion
)

configure_package_config_file(
  ${CMAKE_SOURCE_DIR}/cmake/${PROJECT_NAME}Config.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
  INSTALL_DESTINATION ${INSTALL_CMAKEDIR}
)

install(
  FILES
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
  DESTINATION
    ${INSTALL_CMAKEDIR}
)


############################################################
######################### examples #########################
############################################################

if (QPOASES_BUILD_EXAMPLES)
    # compile qpOASES examples
    set(EXAMPLE_NAMES
        example1
        example1a
        example1b
        example2
        example3
        example3b
        example4
        example5
        exampleLP
        qrecipe
        qrecipeSchur
    )

    FOREACH(ELEMENT ${EXAMPLE_NAMES})
        ADD_EXECUTABLE(${ELEMENT} examples/${ELEMENT}.cpp)
        TARGET_LINK_LIBRARIES(${ELEMENT} qpOASES)
    ENDFOREACH(ELEMENT ${EXAMPLE_NAMES})
endif(QPOASES_BUILD_EXAMPLES)
