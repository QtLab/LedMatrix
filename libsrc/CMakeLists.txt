MESSAGE(STATUS "Checking ${CMAKE_CURRENT_SOURCE_DIR} ")

# name of binary (without lib- prefix)
string(TOLOWER "${PROJECT_NAME}" LIBRARY_NAME)

#Since the "standard" cmake template works with component name set it
set(COMPONENT_NAME ${PROJECT_NAME})

#includes etc. in folder transmog
SET(COMPONENT_PATH ${LIBRARY_NAME})

# Interface/binary version
SET(LIBRARY_VERSION ${PROJECT_VERSION})
# add . to the includes during the build
set(CMAKE_INCLUDE_CURRENT_DIR ON)

#------------------------------
# add compile definitions
#------------------------------
SET(CPP_DEFS "")
LIST(APPEND CPP_DEFS ""  )

#------------------------------
# normal sources
#------------------------------
set(SRCS
   ${CMAKE_CURRENT_SOURCE_DIR}/rain.cpp
)


#------------------------------
# Output a library
#------------------------------
ADD_LIBRARY(${LIBRARY_NAME} STATIC ${SRCS})

set_target_properties(
  ${LIBRARY_NAME} PROPERTIES
#  COMPILE_DEFINITIONS ${CPP_DEFS}
  VERSION ${LIBRARY_VERSION}
  SOVERSION ${LIBRARY_VERSION}
  )

target_compile_definitions(${LIBRARY_NAME}
  PUBLIC
  "FOO_BAR=1"
  )

target_include_directories(
  ${LIBRARY_NAME}
  PRIVATE
  $<BUILD_INTERFACE:${PROJECT_INCLUDE_DIR}>
  PUBLIC
  $<BUILD_INTERFACE:${PROJECT_INCLUDE_DIR}>
  )

if(CMAKE_COMPILER_IS_GNUXX)
target_link_libraries(${LIBRARY_NAME}
  m #link libmath
  )
endif(CMAKE_COMPILER_IS_GNUXX)

#-----
# Install
#-----
MESSAGE(STATUS "** Generating Package Configurations **")

include(CMakePackageConfigHelpers)
WRITE_BASIC_PACKAGE_VERSION_FILE(
  ${version_config}
  VERSION ${LIBRARY_VERSION}
  COMPATIBILITY SameMajorVersion
)

# Configure '<PROJECT-NAME>Config.cmake'
# Note: variable 'targets_export_name' used
CONFIGURE_FILE("${CMAKE_SOURCE_DIR}/cmake/Config.cmake.in"
  "${component_config}" @ONLY)

INSTALL(TARGETS ${LIBRARY_NAME}
  EXPORT ${targets_export_name}
  COMPONENT DEVELOP
  ARCHIVE DESTINATION ${INSTALL_LIB_DIR}
  LIBRARY DESTINATION ${INSTALL_LIB_DIR}
  RUNTIME DESTINATION ${INSTALL_BIN_DIR}
  # this will add -I<prefix>include/transmog to client compile flags
  #INCLUDES DESTINATION ${INSTALL_INCLUDE_DIR}/${COMPONENT_PATH}
  INCLUDES DESTINATION ${INSTALL_INCLUDE_DIR}
  )


INSTALL(DIRECTORY
  ${PROJECT_INCLUDE_DIR}/
  COMPONENT DEVELOP
  DESTINATION ${INSTALL_INCLUDE_DIR}/${COMPONENT_PATH}
  )

INSTALL(
  EXPORT ${targets_export_name}
  COMPONENT DEVELOP
  NAMESPACE "${COMPONENT_NAME}::"
  DESTINATION "${INSTALL_CMAKE_DIR}/${COMPONENT_NAME}"
  )

INSTALL(
  FILES "${component_config}" "${version_config}"
  COMPONENT DEVELOP
  DESTINATION "${INSTALL_CMAKE_DIR}/${COMPONENT_NAME}"
  )
