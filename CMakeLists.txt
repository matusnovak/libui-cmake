cmake_minimum_required(VERSION 3.1)
project(libui C)

option(BUILD_EXAMPLES "Build libui examples" ON)
option(LIBUI_BUILD_SHARED_LIBS "Build shared libraries" ${BUILD_SHARED_LIBS})

file(GLOB_RECURSE COMMON_SOURCES ${CMAKE_SOURCE_DIR}/libui/common/*.c)
file(GLOB_RECURSE COMMON_HEADERS ${CMAKE_SOURCE_DIR}/libui/common/*.h)
file(GLOB_RECURSE UNIX_SOURCES ${CMAKE_SOURCE_DIR}/libui/unix/*.c)
file(GLOB_RECURSE UNIX_HEADERS ${CMAKE_SOURCE_DIR}/libui/unix/*.h)
file(GLOB_RECURSE WINDOWS_SOURCES ${CMAKE_SOURCE_DIR}/libui/windows/*.c)
file(GLOB_RECURSE WINDOWS_HEADERS ${CMAKE_SOURCE_DIR}/libui/windows/*.h)
file(GLOB_RECURSE DARWIN_SOURCES ${CMAKE_SOURCE_DIR}/libui/darwin/*.m)
file(GLOB_RECURSE DARWIN_HEADERS ${CMAKE_SOURCE_DIR}/libui/darwin/*.h)
set(TO_REMOVE ${CMAKE_SOURCE_DIR}/libui/common/OLD_table.c)

if(WIN32)
  set(SOURCES ${COMMON_SOURCES} ${WINDOWS_SOURCES})
  set(HEADERS ${COMMON_HEADERS} ${WINDOWS_HEADERS})
elseif(APPLE)
  set(TO_REMOVE ${TO_REMOVE} ${CMAKE_SOURCE_DIR}/libui/darwin/OLD_table.m)
  set(SOURCES ${COMMON_SOURCES} ${DARWIN_SOURCES})
  set(HEADERS ${COMMON_HEADERS} ${DARWIN_HEADERS})
elseif(UNIX)
  set(TO_REMOVE ${TO_REMOVE} ${CMAKE_SOURCE_DIR}/libui/unix/OLD_table.c)
  set(SOURCES ${COMMON_SOURCES} ${UNIX_SOURCES})
  set(HEADERS ${COMMON_HEADERS} ${UNIX_HEADERS})
endif()

foreach(ITEM ${TO_REMOVE})
  list(REMOVE_ITEM SOURCES "${ITEM}")
endforeach()

if(LIBUI_BUILD_SHARED_LIBS)
  add_library(${PROJECT_NAME} ${SOURCES} ${HEADERS})
  add_compile_definitions(${PROJECT_NAME} PUBLIC libui_EXPORTS=1)
else()
  add_library(${PROJECT_NAME} STATIC ${SOURCES} ${HEADERS})
endif()

set_target_properties(${PROJECT_NAME} PROPERTIES LINKER_LANGUAGE C)

if(WIN32)
  if(MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE /bigobj)
  endif()
elseif(APPLE)
  find_library(FOUNDATION_LIBRARY Foundation)
  find_library(COCOA_LIBRARY Cocoa)
  target_link_libraries(${PROJECT_NAME}
    ${FOUNDATION_LIBRARY}
    ${COCOA_LIBRARY}
  )
elseif(UNIX)
  find_package(PkgConfig REQUIRED)
  pkg_check_modules(GTK3 REQUIRED gtk+-3.0)
  pkg_search_module(GLIB REQUIRED glib-2.0)
  target_include_directories(${PROJECT_NAME} PRIVATE 
    ${GTK3_INCLUDE_DIRS}
    ${GLIB_INCLUDE_DIRS}
  )
endif()

if(BUILD_EXAMPLES)
  set(EXAMPLES controlgallery datetime drawtext histogram timer)
  foreach(EXAMPLE ${EXAMPLES})
    file(GLOB EXAMPLE_SOURCES ${CMAKE_SOURCE_DIR}/libui/examples/${EXAMPLE}/main.*)
    message("Example: ${EXAMPLE} sources: ${EXAMPLE_SOURCES}")
    add_executable(${EXAMPLE} ${EXAMPLE_SOURCES})
    target_include_directories(${EXAMPLE} PRIVATE ${CMAKE_SOURCE_DIR}/libui)
    target_link_libraries(${EXAMPLE} ${PROJECT_NAME})
    
    if(WIN32)
      if(MSVC)
        target_compile_options(${EXAMPLE} PRIVATE /bigobj)
      endif()
    elseif(APPLE)
      target_link_libraries(${EXAMPLE}
        ${FOUNDATION_LIBRARY}
        ${COCOA_LIBRARY}
      )
    elseif(UNIX)
      target_link_libraries(${EXAMPLE} 
        ${GTK3_LIBRARIES} 
        ${GLIB_LDFLAGS} 
        ${CMAKE_DL_LIBS}
        m
      )
    endif()
  endforeach()
endif()
