# libui - CMake

[![Build status](https://ci.appveyor.com/api/projects/status/mlqaxlgmbre0rf70/branch/master?svg=true)](https://ci.appveyor.com/project/matusnovak/libui-cmake/branch/master) [![Build Status](https://travis-ci.org/matusnovak/libui-cmake.svg?branch=master)](https://travis-ci.org/matusnovak/libui-cmake) [![CircleCI](https://circleci.com/gh/matusnovak/libui-cmake/tree/master.svg?style=svg)](https://circleci.com/gh/matusnovak/libui-cmake/tree/master)

This is a CMake version of [libui](https://github.com/andlabs/libui)

> https://github.com/andlabs/libui
>
> Simple and portable (but not inflexible) GUI library in C that uses the native GUI technologies of each platform it supports. 
>
> by [@andlabs](https://github.com/andlabs)

### Works on

* Visual Studio 2013 or newer (Win32 or Win64)
* MinGW-64 (i686 or x86_64)
* GCC (any meaningful version)
* Clang

### Dependencies

* git
* cmake
* libgtk-3-dev (Linux only, installed via `sudo apt-get install libgtk-3-dev`)

## Build

```bash
# Clone this repository
git clone https://github.com/matusnovak/libui-cmake

# Download the dependencies (the libui)
git submodule update --init

# Create a directory where to generate the files
mkdir build
cd build

# Configure
cmake .. -G "Visual Studio 2017 15 Win64" -DCMAKE_INSTALL_PREFIX=C:\path\to\install

# Build and install
cmake --build . --target install
```

After you run the install target (the `cmake --build . --target install`) then the `libui.lib` and `libui.dll` (or `libui.a/libui.so` on Unix) will be installed in the path you have specified via `CMAKE_INSTALL_PREFIX`. The library will be located in the `C:\path\to\install\lib` folder and headers in the `C:\path\to\install\include\libui` folder.

## Shared and static versions

Shared library is built by default. You can disable building of a shared library and enable static library by adding `-DLIBUI_BUILD_SHARED_LIBS=OFF` to the command line while configuring the project.

## Examples

Examples are built by CMake automatically. You can disable building of examples by adding `-DLIBUI_BUILD_EXAMPLES=OFF` to the command line while configuring the project.

## Linking additional dependencies (static library only)

**This is not needed if you are using a dynamic library (`libui.dll` or `libui.so`)**

### Windows

Your application will need additional libraries. Use the following example:

```cmake
target_link_libraries(${PROJECT_NAME} D2d1 Dwrite windowscodecs UxTheme Comctl32)
if(MSVC)
  target_link_options(${PROJECT_NAME} PRIVATE /MANIFEST:NO)
endif()
```

Make sure that you also add `Microsoft.Windows.Common-Controls` to your assembly file. Libui already comes with such manifest file. You can include it to your project by simply adding `libui/windows/resources.rc` to `add_executable` inside of your CMake file. The CMake is intelligent enough to handle the `*.rc` file.

Or, instead of using `resources.rc`, add the following (Visual Studio only) to your source file:

```c++
#pragma comment(linker,"/manifestdependency:\"type='win32' name='Microsoft.Windows.Common-Controls' version='6.0.0.0' processorArchitecture='*' publicKeyToken='6595b64144ccf1df' language='*'\"")
```

### Linux

You will need to link gtk3, glib, libm, and libdl to your application. Use the example below:

```cmake
find_package(PkgConfig REQUIRED)
pkg_check_modules(GTK3 REQUIRED gtk+-3.0)
pkg_search_module(GLIB REQUIRED glib-2.0)

target_link_libraries(${PROJECT_NAME} 
  ${GTK3_LIBRARIES} 
  ${GLIB_LDFLAGS} 
  ${CMAKE_DL_LIBS}
  m
)
```

### OSX

You will need to link the Foundation and Cocoa frameworks. These come bundled with an installation of the XCode.

```cmake
find_library(FOUNDATION_LIBRARY Foundation)
find_library(COCOA_LIBRARY Cocoa)
target_link_libraries(${PROJECT_NAME}
  ${FOUNDATION_LIBRARY}
  ${COCOA_LIBRARY}
)
```

## F.A.Q

**My application crashes on startup** or **uiInit() fails**

This can happen on Windows. Make sure that you include the manifest in your application. Check `libui/windows/resources.rc` and `libui/windows/libui.manifest` inside of the libui.

**Undefined reference to uiXYZ()**

Have you linked the `libui.lib` (or `libui.a` `libui.so`) to your application?

**Application crashes on startup libui.dll is missing**

You will need to add `libui.dll` to your system `PATH` environment variable, or simply copy the DLL where you executable file is located.
