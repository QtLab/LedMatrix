MESSAGE(STATUS "Checking ${CMAKE_CURRENT_SOURCE_DIR} ")

SET(BINARY_NAME "${PROJECT_NAME}_gtest")


# Threading library for gtest
# Use ${CMAKE_THREAD_LIBS_INIT} for the library
find_package(Threads REQUIRED)

# --------------------------------------------------------------------------------
# Unit test sources, TestDoubles, Mocks etc.
# --------------------------------------------------------------------------------
SET(TEST_HARNESS_SRCS
  ${CMAKE_CURRENT_SOURCE_DIR}/test.cpp
  )

# --------------------------------------------------------------------------------
# Add libraries to test here
# --------------------------------------------------------------------------------

SET(DUT_LIBS "")
if(TARGET ledmatrix)
  add_library(${PROJECT_NAME}::ledmatrix ALIAS ledmatrix)
else()
    find_package(${PROJECT_NAME} COMPONENTS ledmatrix CONFIG REQUIRED)
endif()
LIST(APPEND DUT_LIBS ledmatrix)


#------------------------------
# add compile definitions
#------------------------------
SET(CPP_DEFS "AWESOMEVAR=1")
LIST(APPEND CPP_DEFS Foo)

# --------------------------------------------------------------------------------
if(NOT BUILD_GTEST_FROM_SRC)
  #Use prebuilt google_test
  find_package(GTest REQUIRED)

else(NOT BUILD_GTEST_FROM_SRC)
  # Great idea how to download and configure gtest at configuration time
  # taken form http://crascit.com/2015/07/25/cmake-gtest/
#-- Snip
  # Download and unpack googletest at configure time
  configure_file(CMakeLists.txt.in
    ${CMAKE_BINARY_DIR}/googletest-download/CMakeLists.txt)

  execute_process(COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}" .
	WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/googletest-download )

  execute_process(COMMAND ${CMAKE_COMMAND} --build .
	WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/googletest-download )

  # Prevent GoogleTest from overriding our compiler/linker options
  # when building with Visual Studio
  set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)

  # Add googletest directly to our build. This adds
  # the following targets: gtest, gtest_main, gmock
  # and gmock_main
  add_subdirectory(${CMAKE_BINARY_DIR}/googletest-src
      ${CMAKE_BINARY_DIR}/googletest-build)
#-- snap


endif(NOT BUILD_GTEST_FROM_SRC)

#------------------------------
# Binary
#------------------------------

# UnitTests Executable (for target)
add_executable(${BINARY_NAME}
  ${TEST_HARNESS_SRCS}
  )



SET_TARGET_PROPERTIES(
  ${BINARY_NAME}
  PROPERTIES
 # COMPILE_DEFINITIONS ${CPP_DEFS}
  CXX_STANDARD 11
  VERSION ${PROJECT_VERSION}
  DEBUG_POSTFIX "_dbg"
  )


# Linkage with gtest_main coverage etc.
TARGET_LINK_LIBRARIES(
  ${BINARY_NAME}
  ${DUT_LIBS}   # Units under test
  gtest
  gmock
  gmock_main
  )


if(TEST_COVERAGE)
  TARGET_LINK_LIBRARIES (${BINARY_NAME}  ${COVERAGE_LIB} )
endif(TEST_COVERAGE)

# Call the testBinary with junit-xml output
add_test(junitout "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${BINARY_NAME}"
  --gtest_output=xml:gtestresults.xml )

if(INSTALL_UNIT_TEST_ON_TARGET)
  INSTALL(TARGETS  ${BINARY_NAME} RUNTIME DESTINATION ${TARGET_EXE_INSTALL_DIR} )
endif(INSTALL_UNIT_TEST_ON_TARGET)
