cmake_minimum_required(VERSION 3.5)
set(LIBNAME sqlite3)
set(MAJOR_VERSION 3)
project(${LIBNAME} VERSION "${MAJOR_VERSION}.47.0")

option(sqlite3_STATIC "Build static lib" OFF)

set(CMAKE_CXX_STANDARD 11)

if (sqlite3_STATIC)
  set(sqlite3_LIBTYPE "STATIC")
else()
  set(sqlite3_LIBTYPE "SHARED")
endif()

set(CMAKE_POSITION_INDEPENDENT_CODE ON)

set(sqlite_SRC src/sqlite3.c)
add_library(${LIBNAME} ${sqlite3_LIBTYPE} ${sqlite_SRC})
target_include_directories(${LIBNAME} PRIVATE src)
set_target_properties(${LIBNAME} PROPERTIES CXX_VISIBILITY_PRESET hidden)

if(NOT MSVC)
  target_compile_options(${LIBNAME} PRIVATE "-Wno-dangling-else")
endif()

include(GNUInstallDirs)

install(FILES
  src/sqlite3.h src/sqlite3ext.h
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${LIBNAME}
  COMPONENT Devel)

# CMake installation boilerplate
install(TARGETS ${LIBNAME}
  EXPORT ${LIBNAME}Targets
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
  PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

set_property(TARGET ${LIBNAME} PROPERTY VERSION ${PROJECT_VERSION})
set_property(TARGET ${LIBNAME} PROPERTY SOVERSION ${MAJOR_VERSION})
set_property(TARGET ${LIBNAME} PROPERTY INTERFACE_${LIBNAME}_MAJOR_VERSION ${MAJOR_VERSION})
set_property(TARGET ${LIBNAME} APPEND PROPERTY COMPATIBLE_INTERFACE_STRING ${LIBNAME}_MAJOR_VERSION)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/${LIBNAME}/${LIBNAME}ConfigVersion.cmake"
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY AnyNewerVersion
)

export(EXPORT ${LIBNAME}Targets
  FILE "${CMAKE_CURRENT_BINARY_DIR}/${LIBNAME}/${LIBNAME}Targets.cmake"
  NAMESPACE ${LIBNAME}::
)
configure_file(cmake/${LIBNAME}Config.cmake
  "${CMAKE_CURRENT_BINARY_DIR}/${LIBNAME}/${LIBNAME}Config.cmake"
  COPYONLY
)

set(ConfigPackageLocation lib/cmake/${LIBNAME})
install(EXPORT ${LIBNAME}Targets
  FILE
    ${LIBNAME}Targets.cmake
  NAMESPACE
    ${LIBNAME}::
  DESTINATION
    ${ConfigPackageLocation}
)
install(
  FILES
    cmake/${LIBNAME}Config.cmake
    "${CMAKE_CURRENT_BINARY_DIR}/${LIBNAME}/${LIBNAME}ConfigVersion.cmake"
  DESTINATION
    ${ConfigPackageLocation}
  COMPONENT
    Devel
)

# CLI
set(BUILD_CLI OFF CACHE STRING "Build commandline tools")
if (BUILD_CLI)
  add_executable(sqlite src/sqlite3.c src/shell.c src/sqlite3.h src/sqlite3ext.h)
  if(WIN32)
    add_custom_command(TARGET sqlite POST_BUILD
      COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:sqlite> ${CMAKE_BINARY_DIR}/sqlite.exe
      DEPENDS sqlite
    )
    install(FILES sqlite.exe DESTINATION bin)
  else()
    include(FindThreads)
    target_link_libraries(sqlite m ${CMAKE_THREAD_LIBS_INIT} ${CMAKE_DL_LIBS})
    install(TARGETS sqlite RUNTIME DESTINATION bin)
  endif()
endif()
