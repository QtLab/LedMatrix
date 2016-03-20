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
  #If Google Test is compiled from source
  include(ExternalProject)

  #--------------------------------------------------------------------------
  # Google Test as external project
  # Why - see http://code.google.com/p/googletest/wiki/FAQ#
  # How - borrowed form stackexchange
  #		(http://stackoverflow.com/questions/9689183/cmake-googletest)
  # http://stackoverflow.com/questions/9689183/cmake-googletest
  # http://www.kitware.com/media/html/BuildingExternalProjectsWithCMake2.8.html
  #--------------------------------------------------------------------------
  set(GTEST_PREFIX "gtest")
  ExternalProject_Add(
	GTestExternal
	GIT_REPOSITORY 	https://github.com/google/googletest.git
	# URL https://github.com/google/googlemock/archive/release-1.7.0.zip
	# URL_HASH
	TIMEOUT 10
	# PATCH_COMMMAND "" # Thomas: empty patch we dont patch google test
	# PATCH_COMMAND svn patch ${CMAKE_SOURCE_DIR}/gtest.patch
	#			${CMAKE_BINARY_DIR}/ThirdParty/src/googletest
	# Force separate output paths for debug and release builds to allow easy
	# identification of correct lib in subsequent TARGET_LINK_LIBRARIES commands
	CMAKE_ARGS -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
	-DCMAKE_ARCHIVE_OUTPUT_DIRECTORY_DEBUG:PATH=${CMAKE_LIBRARY_OUTPUT_DIRECTORY}
	-DCMAKE_ARCHIVE_OUTPUT_DIRECTORY_RELEASE:PATH=${CMAKE_LIBRARY_OUTPUT_DIRECTORY}
	-DBUILD_SHARED_LIBS=Off
	-DGTEST_HAS_PTHREAD=0
	# -DCMAKE_ARCHIVE_OUTPUT_DIRECTORY_RELEASE:PATH=${CMAKE_LIBRARY_OUTPUT_DIRECTORY}
	# -Dgtest_force_shared_crt=ON
	PREFIX "${GTEST_PREFIX}"
	# Disable install step
	INSTALL_COMMAND ""
	# Empty update command to speed up make - we don't need to constantly check
 	UPDATE_COMMAND ""
 	)

  # ExternalProject_Get_Property(GTestExternal source_dir)
  # include_directories(${source_dir}/include)

  set(LIBPREFIX "${CMAKE_STATIC_LIBRARY_PREFIX}")
  set(LIBSUFFIX "${CMAKE_STATIC_LIBRARY_SUFFIX}")
  set(GTEST_LOCATION "${GTEST_PREFIX}/src/GTestExternal-build")
  set(GTEST_INCLUDES "${GTEST_PREFIX}/src/GTestExternal/include")
  set(GTEST_LIBRARY  "${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/${LIBPREFIX}gtest${LIBSUFFIX}")
  set(GTEST_MAINLIB  "${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/${LIBPREFIX}gtest_main${LIBSUFFIX}")

  add_library(GTest IMPORTED STATIC GLOBAL)

  set_target_properties(GTest PROPERTIES
	IMPORTED_LOCATION                 "${GTEST_LIBRARY}"
	INTERFACE_INCLUDE_DIRECTORIES     "${GTEST_INCLUDES}"
	IMPORTED_LINK_INTERFACE_LIBRARIES "${CMAKE_THREAD_LIBS_INIT}")

  add_library(GTestMain IMPORTED STATIC GLOBAL)
  set_target_properties(GTestMain PROPERTIES
	IMPORTED_LOCATION "${GTEST_MAINLIB}"
	IMPORTED_LINK_INTERFACE_LIBRARIES  "${GTEST_LIBRARY};${CMAKE_THREAD_LIBS_INIT}")

  add_dependencies(GTest GTestExternal)
  add_dependencies(GTestMain GTestExternal)

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
 # COMPILE_FLAGS "--std=c++11"
  VERSION ${PROJECT_VERSION}
  DEBUG_POSTFIX "_dbg"
  )


# Linkage with gtest_main coverage etc.
TARGET_LINK_LIBRARIES(
  ${BINARY_NAME}
  ${DUT_LIBS}   # Units under test
  # ${GTEST_MAIN_LIBRARY}
  # ${GTEST_LIBRARY}
  GTestMain
  GTest
  #${CMAKE_THREAD_LIBS_INIT}
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
