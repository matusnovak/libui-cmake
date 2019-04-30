cmake_minimum_required(VERSION 3.1)
project(libui C)

option(BUILD_EXAMPLES "Build libui examples" ON)
if(NOT WIN32)
  option(LIBUI_BUILD_SHARED_LIBS "Build shared libraries" ${BUILD_SHARED_LIBS})
else()
  set(LIBUI_BUILD_SHARED_LIBS OFF)
  add_compile_options(-bigobj)
endif()

file(GLOB_RECURSE COMMON_SOURCES ${CMAKE_SOURCE_DIR}/libui/common/*.c)
file(GLOB_RECURSE COMMON_HEADERS ${CMAKE_SOURCE_DIR}/libui/common/*.h)
file(GLOB_RECURSE UNIX_SOURCES ${CMAKE_SOURCE_DIR}/libui/unix/*.c)
file(GLOB_RECURSE UNIX_HEADERS ${CMAKE_SOURCE_DIR}/libui/unix/*.h)
file(GLOB_RECURSE WINDOWS_SOURCES ${CMAKE_SOURCE_DIR}/libui/windows/*.c)
file(GLOB_RECURSE WINDOWS_HEADERS ${CMAKE_SOURCE_DIR}/libui/windows/*.h)
set(TO_REMOVE ${CMAKE_SOURCE_DIR}/libui/common/OLD_table.c)

if(UNIX)
  set(TO_REMOVE ${TO_REMOVE} ${CMAKE_SOURCE_DIR}/libui/unix/OLD_table.c)
  set(SOURCES ${COMMON_SOURCES} ${UNIX_SOURCES})
  set(HEADERS ${COMMON_HEADERS} ${UNIX_HEADERS})
elseif(WIN32)
  set(SOURCES ${COMMON_SOURCES} ${WINDOWS_SOURCES})
  set(HEADERS ${COMMON_HEADERS} ${WINDOWS_HEADERS})
endif()

foreach(ITEM ${TO_REMOVE})
  list(REMOVE_ITEM SOURCES "${ITEM}")
endforeach()

if(LIBUI_BUILD_SHARED_LIBS)
  add_library(${PROJECT_NAME} SHARED ${SOURCES} ${HEADERS})
else()
  add_library(${PROJECT_NAME} STATIC ${SOURCES} ${HEADERS})
endif()

set_target_properties(${PROJECT_NAME} PROPERTIES LINKER_LANGUAGE C)

if(UNIX)
  find_package (PkgConfig REQUIRED)
  pkg_check_modules(GTK3 REQUIRED gtk+-3.0)
  pkg_search_module(GLIB REQUIRED glib-2.0)
  target_include_directories(${PROJECT_NAME} PRIVATE 
    ${GTK3_INCLUDE_DIRS}
    ${GLIB_INCLUDE_DIRS}
  )
elseif(WIN32)
  target_compile_options(${PROJECT_NAME} /bigobj)
endif()

if(BUILD_EXAMPLES)
  set(EXAMPLES controlgallery datetime drawtext histogram timer)
  foreach(EXAMPLE ${EXAMPLES})
    file(GLOB EXAMPLE_SOURCES ${CMAKE_SOURCE_DIR}/libui/examples/${EXAMPLE}/main.*)
    message("Example: ${EXAMPLE} sources: ${EXAMPLE_SOURCES}")
    add_executable(${EXAMPLE} ${EXAMPLE_SOURCES})
    target_include_directories(${EXAMPLE} PRIVATE ${CMAKE_SOURCE_DIR}/libui)
    target_link_libraries(${EXAMPLE} ${PROJECT_NAME})
    
    if(UNIX)
      target_link_libraries(${EXAMPLE} 
        ${GTK3_LIBRARIES} 
        ${GLIB_LDFLAGS} 
        ${CMAKE_DL_LIBS}
        m
      )
    elseif(WIN32)
      target_compile_options(${EXAMPLE} /bigobj)
    endif()
  endforeach()
endif()
