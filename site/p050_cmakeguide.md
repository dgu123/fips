---
layout: page
title: CMake Guide
permalink: cmakeguide.html
---

# Fips CMake Guide

Fips projects need to adhere to a few rules in their CMakeLists.txt file
hierarchy. Fips provides a number of cmake macros, variables and toolchain
files to simplify working with cmake files and implement some under-the-hood
magic.

### Fips CMake Macros

Fips provides the following cmake macros to describe a project structure:

#### fips\_setup()

Initializes the fips build system in a cmake file hierarchy. Must be
called once in the root CMakeLists.txt before any other fips cmake
macros.

#### fips\_finish()

Must be called in the root CMakeLists.txt file after any other fips macros
and does any work that must happen once after each cmake run. Currently
this is macro does nothing.

#### fips\_project(name)

Starts a new project with the given name. This must be called at least
once in a hierarchy of CMakeLists.txt files, usually right after 
fips\_setup(). 

Use the fips\_project() macro instead of cmake's builtin project() macro

#### fips\_ide\_group(name)

Start a new project explorer folder in an IDE. This can be used to 
group related build targets for a clearer layout in the IDE's project 
explorer.

#### fips\_add\_subdirectory(dir)

Include a child CMakeLists.txt file from a subdirectory. Use this instead
of cmake's built-in add\_subdirectory() macro.

#### fips\_begin\_module(name)

Begin defining a fips module. Modules are special high-level static link-libraries
with a few additional features over conventional libs:

* can define dependencies to other modules, which are automatically
  resolved when linking apps
* can contain code-generation python scripts which are added as 
  custom build targets to the build process

After a fips\_begin\_module() the following fips macros are valid:

* fips\_dir()
* fips\_files()
* fips\_deps()
* fips\_end\_module()

#### fips\_end\_module()

This finishes a fips\_begin\_module() block.

#### fips\_begin\_lib(name)

Begin defining a fips library. This is a simple static link library in C or C++
which cannot have dependencies to other libs and cannot contain python
code generation files.

After a fips\_begin\_lib() the following fips macros are valid:

* fips\_dir()
* fips\_files()
* fips\_deps()
* fips\_end\_lib()

#### fips\_end\_lib()

This finishes a fips\_begin\_lib() block.

#### fips\_begin\_app(name type)

Begin defining a fips application. The _type_ argument can be either 'windowed'
or 'cmdline', this only makes a difference on platform with separate
command-line and UI application types, like Windows (WinMain vs main)
or OSX (app bundle vs command line tool).

After a fips\_begin\_app() the following fips macros are valid:

* fips\_dir()
* fips\_files()
* fips\_deps()
* fips\_end\_app()

#### fips\_end\_app()

This finishes a fips\_begin\_app() block.

#### fips\_dir(dir)

Defines a source code subdirectory for the following fips\_files() statements.
This is only necessary if source files are located in subdirectories of the
directory where the current CMakeLists.txt file is located. You don't need
to provide a fips\_dir() statement for files in the same directory as 
their CMakeLists.txt file.

#### fips\_files(file ...)

Add source files in the currently set directory to the current module, lib or app.
This isn't restricted to C/C++ files, but any file that should show
up in the IDE project explorer. The actual build process will ignore any
files with file extensions that cmake doesn't know how to build.

The following file extensions are recognized by the build process:

* **.cc, .cpp**:    C++ source files (compiled with C++11 support)
* **.c**:           C source files
* **.m, .mm**:      Objective-C and Objective-C++ source files
* **.h, .hh**:      C/C++/Obj-C headers
* **.py**:          Python source code generator scripts

#### fips\_deps(dep ...)

Add dependencies to the current app or module. This can be the name
of another fips module or lib, or the name of an existing static link
library. Dependencies added to fips modules will be resolved recursively
when linking apps. Fips will also take care of the dreaded linking order
problem of GCC where symbols can't be resolved if the
order of link libraries is wrong or in case of cyclic dependencies.

### The fips-include.cmake File

A fips project may contain an optional cmake file called **fips-include.cmake**
at the root level of a project (same directory level as the root
CMakeLists.txt file). The fips-include.cmake file should contain all cmake
definitions that need to be visible when using this fips project as an
external dependency in another project. Fips will include this file either
when the project itself is compiled, or the project is imported as an
external dependency in other projects.

Check out the [fips-include.cmake](https://github.com/floooh/oryol/blob/master/fips-include.cmake)
file included in the Oryol 3D engine for a complex example.

### Fips Predefined CMake Variables

Fips defines a number of useful cmake variables:

* **FIPS\_POSIX**: target platform is UNIX-ish (basically anything but Windows)
* **FIPS\_WINDOWS**: target platform is Windows
* **FIPS\_OSX**: target platform is OSX-ish (either OSX 10.x or iOS) 
* **FIPS\_LINUX**: target platform is Linux
* **FIPS\_MACOS**: target platform is OSX 10.x
* **FIPS\_IOS**: target platform is iOS
* **FIPS\_WIN32**: target platform is 32-bit Windows
* **FIPS\_WIN64**: target platform is 64-bit Windows
* **FIPS\_EMSCRIPTEN**: target platform is emscripten
* **FIPS\_PNACL**: target platform is PNaCl
* **FIPS\_ANDROID**: target platform is Android
* **FIPS\_HOST\_WINDOWS**: host platform is Windows
* **FIPS\_HOST\_OSX**: host platform is OSX
* **FIPS\_HOST\_LINUX**: host platform is Linux
* **FIPS\_ROOT\_DIR**: absolute path of the fips root directory
* **FIPS\_PROJECT\_DIR**: absolute path of the current project
* **FIPS\_DEPLOY\_DIR**: absolute path of the deployment directory
* **FIPS\_CONFIG**: name of the current build configuration (e.g. osx-xcode-debug)
* **FIPS\_IMPORT**: set inside import CMakeLists.txt files

### Fips CMake Options

Fips provides a few build options which can be tweaked by running **./fips config**
(requires the ccmake or cmake-gui tools to be in the path).

Besides _./fips config_, cmake options can also be provided in
a build config YAML file, for instance the following config file
sets the FIPS\_UNITTESTS and FIPS\_UNITTESTS\_HEADLESS options to ON:

{% highlight yaml %}
---
platform: emscripten 
generator: Ninja 
build_tool: ninja
build_type: Debug
defines:
    FIPS_UNITTESTS: ON
    FIPS_UNITTESTS_HEADLESS: ON
{% endhighlight %}

### CMakeLists.txt Samples

Here's a very simple root CMakeLists.txt file from the _fips-hello-world_
sample project:

{% highlight cmake %}
cmake_minimum_required(VERSION 2.8)

# include the fips main cmake file
get_filename_component(FIPS_ROOT_DIR "../fips" ABSOLUTE)
include("${FIPS_ROOT_DIR}/cmake/fips.cmake")

fips_setup()
fips_project(fips-hello-world)
fips_add_subdirectory(src)
fips_finish()
{% endhighlight %}

The _src_ subdirectory contains the CMakeLists.txt file which defines
the actual appliction:

{% highlight cmake %}
fips_begin_app(hello cmdline)
    fips_files(hello.cc)
    fips_deps(dep1)
fips_end_app()
{% endhighlight %}

This is a more complex root CMakeLists.txt file from the Oryol 3D engine:

{% highlight cmake %}
#----------------------------------------------------------
#	oryol cmake root file
#
#	See BUILD.md for details how to build oryol.
#----------------------------------------------------------
cmake_minimum_required(VERSION 2.8)

get_filename_component(FIPS_ROOT_DIR "../fips" ABSOLUTE)
include("${FIPS_ROOT_DIR}/cmake/fips.cmake")

option(ORYOL_SAMPLES "Build Oryol samples" ON)

include_directories(code)
include_directories(code/Modules)
include_directories(code/Ext)

fips_setup()
fips_project(oryol)
fips_add_subdirectory(code/Hello)
fips_ide_group(Modules)
fips_add_subdirectory(code/Modules)
fips_ide_group(Ext)
fips_add_subdirectory(code/Ext)
if (ORYOL_SAMPLES)
   fips_ide_group(Samples)
   fips_add_subdirectory(code/Samples)
endif()
fips_finish()

{% endhighlight %}

Next a sample which defines a code module with platform-specific source
code in subdirectories, and towards the end some dependencies to other
fips modules:

{% highlight cmake %}

#----------------------------------------------------------
#   oryol Input module
#----------------------------------------------------------
fips_begin_module(Input)
    fips_files(Input.cc Input.h)
    fips_dir(Core)
    fips_files(
        CursorMode.h
        Gamepad.cc Gamepad.h
        InputSetup.h
        Key.cc Key.h
        Keyboard.cc Keyboard.h
        Mouse.cc Mouse.h
        Sensors.h
        Touchpad.cc Touchpad.h
        inputMgr.h
    )
    fips_dir(base)
    fips_files(inputMgrBase.cc inputMgrBase.h)
    fips_dir(touch)
    fips_files(
        gestureState.h
        panDetector.cc
        panDetector.h
        pinchDetector.cc
        pinchDetector.h
        tapDetector.cc
        tapDetector.h
        touchEvent.cc
        touchEvent.h
    )
    if (FIPS_ANDROID)
        fips_dir(android)
        fips_files(androidInputMgr.cc androidInputMgr.h)
    endif()
    if (FIPS_EMSCRIPTEN)
        fips_dir(emsc)
        fips_files(emscInputMgr.cc emscInputMgr.h)
    endif()
    if (FIPS_IOS)
        fips_dir(ios)
        fips_files(iosInputMgr.cc iosInputMgr.h)
    endif()
    if (FIPS_PNACL)
        fips_dir(pnacl)
        fips_files(pnaclInputMgr.cc pnaclInputMgr.h)
    endif()
    if (FIPS_MACOS OR FIPS_WINDOWS OR FIPS_LINUX)
        fips_dir(glfw)
        fips_files(glfwInputMgr.cc glfwInputMgr.h)
        fips_deps(glfw3)
    endif()
    fips_deps(Core Gfx Time)
fips_end_module()

{% endhighlight %}

Finally an example how to wrap a simple C library with a few custom
C preprocessor defines:

{% highlight cmake %}
fips_begin_lib(zlib)
    fips_files(
        adler32.c
        compress.c
        crc32.c crc32.h
        deflate.c deflate.h
        infback.c 
        inffast.c inffast.h
        inffixed.h
        inflate.c inflate.h
        inftrees.c inftrees.h
        trees.c trees.h
        uncompr.c
        zconf.h
        zlib.h
        zutil.c zutil.h
    )
    add_definitions(-D_NO_FSEEKO)
    add_definitions(-D_CRT_SECURE_NO_DEPRECATE)
    add_definitions(-D_CRT_NONSTDC_NO_DEPRECATE)
fips_end_lib(zlib)
{% endhighlight %}

