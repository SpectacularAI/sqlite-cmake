PROJECT(sqlite3)
cmake_minimum_required(VERSION 3.3)

include_directories(${CMAKE_SOURCE_DIR}/src)

add_library(sqlite3 STATIC src/sqlite3.c src/sqlite3.h src/sqlite3ext.h)

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

install(FILES src/sqlite3.h src/sqlite3ext.h DESTINATION include)
install(TARGETS sqlite3 LIBRARY DESTINATION lib ARCHIVE DESTINATION lib)
