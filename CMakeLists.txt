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
option(QPOASES_BUILD_TESTS "Build tests." OFF)
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

  foreach(ELEMENT ${EXAMPLE_NAMES})
    add_executable(${ELEMENT} examples/${ELEMENT}.cpp)
    target_link_libraries(${ELEMENT} qpOASES)
  endforeach(ELEMENT ${EXAMPLE_NAMES})
endif(QPOASES_BUILD_EXAMPLES)

if (QPOASES_BUILD_TESTS)
  set(TEST_NAMES
    test_bench
    test_matrices
    test_matrices2
    test_matrices3
    test_indexlist
    test_example1
    test_example1a
    test_example1b
    test_example2
    test_example4
    test_example5
    test_example6
    test_example7
    test_exampleLP
    test_qrecipe
    test_qrecipeSchur
    test_smallSchur
    test_infeasible1
    test_hs268
    test_gradientShift
    test_runAllOqpExamples
    test_sebastien1
    test_vanBarelsUnboundedQP
    test_janick1
    test_janick2
    test_constraintProduct1
    test_constraintProduct2
    test_guessedWS1
    test_externalChol1
    test_identitySqproblem
  )

  enable_testing()
  find_program(MEMORYCHECK_COMMAND NAMES valgrind)
  set(MEMORYCHECK_COMMAND_OPTIONS "--trace-children=yes --leak-check=full")

  foreach(ELEMENT ${TEST_NAMES})
    add_executable(${ELEMENT} tests/${ELEMENT}.cpp)
    target_link_libraries(${ELEMENT} qpOASES)
    add_test(NAME ${ELEMENT} COMMAND ${ELEMENT})
  endforeach(ELEMENT ${TEST_NAMES})

endif(QPOASES_BUILD_TESTS)
