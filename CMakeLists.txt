cmake_minimum_required(VERSION 3.4)
project(libui)

option(BUILD_EXAMPLES "Build libui examples" ON)
option(LIBUI_BUILD_SHARED_LIBS "Build shared libraries" ${BUILD_SHARED_LIBS})

file(GLOB_RECURSE COMMON_SOURCES ${CMAKE_SOURCE_DIR}/libui/common/*.c)
file(GLOB_RECURSE COMMON_HEADERS ${CMAKE_SOURCE_DIR}/libui/common/*.h)
file(GLOB_RECURSE UNIX_SOURCES ${CMAKE_SOURCE_DIR}/libui/unix/*.c)
file(GLOB_RECURSE UNIX_HEADERS ${CMAKE_SOURCE_DIR}/libui/unix/*.h)
file(GLOB_RECURSE WINDOWS_SOURCES ${CMAKE_SOURCE_DIR}/libui/windows/*.cpp)
file(GLOB_RECURSE WINDOWS_HEADERS ${CMAKE_SOURCE_DIR}/libui/windows/*.hpp)
file(GLOB_RECURSE DARWIN_SOURCES ${CMAKE_SOURCE_DIR}/libui/darwin/*.m)
file(GLOB_RECURSE DARWIN_HEADERS ${CMAKE_SOURCE_DIR}/libui/darwin/*.h)
set(TO_REMOVE ${CMAKE_SOURCE_DIR}/libui/common/OLD_table.c)
set(MANIFEST ${CMAKE_SOURCE_DIR}/libui/windows/libui.manifest)

if(WIN32)
  set(SOURCES ${COMMON_SOURCES} ${WINDOWS_SOURCES})
  set(HEADERS ${COMMON_HEADERS} ${WINDOWS_HEADERS})
  set(LIBUI_PLATFORM_HEADER ${CMAKE_SOURCE_DIR}/libui/ui_windows.h)
  set(LIBUI_RC_FILE ${CMAKE_SOURCE_DIR}/libui/windows/resources.rc)
elseif(APPLE)
  set(TO_REMOVE ${TO_REMOVE} ${CMAKE_SOURCE_DIR}/libui/darwin/OLD_table.m)
  set(SOURCES ${COMMON_SOURCES} ${DARWIN_SOURCES})
  set(HEADERS ${COMMON_HEADERS} ${DARWIN_HEADERS})
  set(LIBUI_PLATFORM_HEADER ${CMAKE_SOURCE_DIR}/libui/ui_darwin.h)
else()
  set(TO_REMOVE ${TO_REMOVE} ${CMAKE_SOURCE_DIR}/libui/unix/OLD_table.c)
  set(SOURCES ${COMMON_SOURCES} ${UNIX_SOURCES})
  set(HEADERS ${COMMON_HEADERS} ${UNIX_HEADERS})
  set(LIBUI_PLATFORM_HEADER ${CMAKE_SOURCE_DIR}/libui/ui_unix.h)
endif()
set(LIBUI_HEADER ${CMAKE_SOURCE_DIR}/libui/ui.h)

foreach(ITEM ${TO_REMOVE})
  list(REMOVE_ITEM SOURCES "${ITEM}")
endforeach()

if(LIBUI_BUILD_SHARED_LIBS)
  add_library(${PROJECT_NAME} SHARED ${SOURCES} ${HEADERS} ${LIBUI_RC_FILE})
  add_compile_definitions(${PROJECT_NAME} PRIVATE libui_EXPORTS=1)
else()
  add_library(${PROJECT_NAME} STATIC ${SOURCES} ${HEADERS})
endif()
set_target_properties(${PROJECT_NAME} PROPERTIES PUBLIC_HEADER "${LIBUI_HEADER};${LIBUI_PLATFORM_HEADER}")
set_target_properties(${PROJECT_NAME} PROPERTIES LINKER_LANGUAGE CXX)

if(WIN32)
  if(MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE /bigobj)
    if(LIBUI_BUILD_SHARED_LIBS)
      target_link_options(${PROJECT_NAME} PRIVATE /MANIFEST:NO)
    endif()
  endif()
  target_link_libraries(${PROJECT_NAME} D2d1 Dwrite windowscodecs UxTheme)
elseif(APPLE)
  find_library(FOUNDATION_LIBRARY Foundation)
  find_library(COCOA_LIBRARY Cocoa)
  target_link_libraries(${PROJECT_NAME}
    ${FOUNDATION_LIBRARY}
    ${COCOA_LIBRARY}
  )
else()
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
    
    if(WIN32)
      set(EXAMPLE_SOURCES ${EXAMPLE_SOURCES} ${LIBUI_RC_FILE})
    endif()
    
    add_executable(${EXAMPLE} ${EXAMPLE_SOURCES})
    target_include_directories(${EXAMPLE} PRIVATE ${CMAKE_SOURCE_DIR}/libui)
    target_link_libraries(${EXAMPLE} ${PROJECT_NAME})
    add_dependencies(${EXAMPLE} ${PROJECT_NAME})
    
    if(WIN32)
      if(NOT LIBUI_BUILD_SHARED_LIBS)
        target_link_libraries(${EXAMPLE} D2d1 Dwrite windowscodecs UxTheme)
        target_link_options(${EXAMPLE} PRIVATE /MANIFEST:NO)
      endif()
    elseif(APPLE)
      target_link_libraries(${EXAMPLE}
        ${FOUNDATION_LIBRARY}
        ${COCOA_LIBRARY}
      )
    else()
      target_link_libraries(${EXAMPLE} 
        ${GTK3_LIBRARIES} 
        ${GLIB_LDFLAGS} 
        ${CMAKE_DL_LIBS}
        m
      )
    endif()
  endforeach()
endif()

install(TARGETS ${PROJECT_NAME}
  RUNTIME DESTINATION ${CMAKE_INSTALL_PREFIX}/bin
  LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
  ARCHIVE DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
  COMPONENT library
  PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_PREFIX}/include/libui
)
