#/****************************************************************************
# Copyright (c) 2013 cocos2d-x.org
# Copyright (c) 2014 martell malone
#
# http://www.cocos2d-x.org
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:

# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.

# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
# THE SOFTWARE.
# ****************************************************************************/

cmake_minimum_required(VERSION 2.8)
#project (cocos2d-x-samples)

# The version number
set(COCOS2D_X_VERSION 3.3-rc0)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/libs/cocos2d-x/cmake/Modules/")
include(CocosBuildHelpers)

message(${BUILDING_STRING})

set(USE_WEBP_DEFAULT ON)
if(WINRT OR WP8)
  set(USE_WEBP_DEFAULT OFF)
endif()

set(USE_PREBUILT_LIBS_DEFAULT ON)
if(MINGW)
  set(USE_PREBUILT_LIBS_DEFAULT OFF)
endif()

option(USE_CHIPMUNK "Use chipmunk for physics library" OFF)
option(USE_BOX2D "Use box2d for physics library" ON)
option(USE_WEBP "Use WebP codec" ${USE_WEBP_DEFAULT})
option(BUILD_SHARED_LIBS "Build shared libraries" OFF)
option(DEBUG_MODE "Debug or release?" ON)
option(BUILD_EXTENSIONS "Build extension library" ON)
option(BUILD_EDITOR_SPINE "Build editor support for spine" ON)
option(BUILD_EDITOR_COCOSTUDIO "Build editor support for cocostudio" ON)
option(BUILD_EDITOR_COCOSBUILDER "Build editor support for cocosbuilder" ON)
option(BUILD_LIQUIDFUN_TEST "Build LiquidFun test" ON)
option(BUILD_GAFEXAMPLES "Build GAFExamples" ON)
option(USE_PREBUILT_LIBS "Use prebuilt libraries in external directory" ${USE_PREBUILT_LIBS_DEFAULT})

if(USE_PREBUILT_LIBS AND MINGW)
  message(FATAL_ERROR "Prebuilt windows libs can't be used with mingw, please use packages.")
endif()

if(DEBUG_MODE)
  set(CMAKE_BUILD_TYPE DEBUG)
else(DEBUG_MODE)
  set(CMAKE_BUILD_TYPE RELEASE)
endif(DEBUG_MODE)

set(CMAKE_C_FLAGS_DEBUG "-DCOCOS2D_DEBUG=1")
set(CMAKE_CXX_FLAGS_DEBUG ${CMAKE_C_FLAGS_DEBUG})
set(USE_CHIPMUNK 0)
set(BUILD_GAFEXAMPLES 0)
set(USE_BOX2D 1)

# Compiler options
if(MSVC)
  ADD_DEFINITIONS(-D_CRT_SECURE_NO_WARNINGS -D_SCL_SECURE_NO_WARNINGS
                  -wd4251 -wd4244 -wd4334 -wd4005 -wd4820 -wd4710
                  -wd4514 -wd4056 -wd4996 -wd4099)
else()
  set(CMAKE_C_FLAGS_DEBUG "-g -Wall -DCOCOS2D_DEBUG=1")
  set(CMAKE_CXX_FLAGS_DEBUG ${CMAKE_C_FLAGS_DEBUG})
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fno-exceptions -std=c99")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-exceptions -std=c++11 -Wno-deprecated-declarations -Wno-reorder")
  if(CLANG)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++")
  endif()
endif(MSVC)

set(COCOS_EXTERNAL_DIR ${CMAKE_CURRENT_SOURCE_DIR}/libs/cocos2d-x/external)

# Some macro definitions
if(WINDOWS)

  if(BUILD_STATIC)
    ADD_DEFINITIONS (-DCC_STATIC)
  else()
    ADD_DEFINITIONS (-D_USRDLL -D_EXPORT_DLL_ -D_USEGUIDLL -D_USREXDLL -D_USRSTUDIODLL)
  endif()

  ADD_DEFINITIONS (-DCOCOS2DXWIN32_EXPORTS -D_WINDOWS -DWIN32)
  set(PLATFORM_FOLDER win32)
elseif(MACOSX OR APPLE)
  ADD_DEFINITIONS (-DCC_TARGET_OS_MAC)
  ADD_DEFINITIONS (-DUSE_FILE32API)
  set(PLATFORM_FOLDER mac)
elseif(LINUX)
  ADD_DEFINITIONS(-DLINUX)
  set(PLATFORM_FOLDER linux)
elseif(ANDROID)
  ADD_DEFINITIONS (-DUSE_FILE32API)
  set(PLATFORM_FOLDER android)
else()
  message( FATAL_ERROR "Unsupported platform, CMake will exit" )
endif()

if(MINGW)
  add_definitions(-DGLEW_STATIC)
  add_definitions(-D__SSIZE_T)

  if(CLANG)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fno-exceptions ")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-exceptions ")
  endif()
endif()

# architecture
if ( CMAKE_SIZEOF_VOID_P EQUAL 8 )
  set(ARCH_DIR "64-bit")
elseif ( CMAKE_SIZEOF_VOID_P EQUAL 4 )
  set(ARCH_DIR "32-bit")
else()
  message( FATAL_ERROR "Unsupported architecture, CMake will exit" )
endif()

include_directories(
  ${PLATFORM_INCLUDE_DIR}
  ${CMAKE_CURRENT_SOURCE_DIR}/libs/cocos2d-x/cocos
  ${CMAKE_CURRENT_SOURCE_DIR}/libs/cocos2d-x
  ${CMAKE_CURRENT_SOURCE_DIR}/libs/cocos2d-x/deprecated
  ${CMAKE_CURRENT_SOURCE_DIR}/libs/cocos2d-x/cocos/platform
  ${CMAKE_CURRENT_SOURCE_DIR}/libs/cocos2d-x/extensions
  ${CMAKE_CURRENT_SOURCE_DIR}/libs/liquidfun/liquidfun/liquidfun/Box2D
  ${CMAKE_CURRENT_SOURCE_DIR}/libs/cocos2d-x/external
)

if(USE_PREBUILT_LIBS)
  include(CocosUsePrebuiltLibs)
endif()

# GLFW3 used on Mac, Windows and Linux desktop platforms
if(LINUX OR MACOSX OR WINDOWS)
   cocos_find_package(OpenGL OPENGL REQUIRED)
   if(LINUX OR WINDOWS)
     cocos_find_package(GLEW GLEW REQUIRED)
   endif()
   cocos_find_package(GLFW3 GLFW3 REQUIRED)
   include_directories(${GLFW3_INCLUDE_DIRS})
   if(LINUX)
     set(CMAKE_THREAD_PREFER_PTHREAD TRUE)
     find_package(Threads REQUIRED)
     set(THREADS_LIBRARIES ${CMAKE_THREAD_LIBS_INIT})

     cocos_find_package(FMODEX FMODEX REQUIRED)
     cocos_find_package(Fontconfig FONTCONFIG REQUIRED)
   endif()
   if(WINDOWS)
     cocos_find_package(Vorbis VORBIS REQUIRED)
     cocos_find_package(MPG123 MPG123 REQUIRED)
     cocos_find_package(OpenAL OPENAL REQUIRED)
     # because FindOpenAL.cmake set include dir for '#include <al.h>' for portability (not for '#include <AL/al.h>'
     set(OPENAL_DEFINITIONS "-DOPENAL_PLAIN_INCLUDES")
   endif()
endif(LINUX OR MACOSX OR WINDOWS)

# Freetype required on all platforms
cocos_find_package(Freetype FREETYPE REQUIRED)

# Chipmunk
if(USE_CHIPMUNK)
  cocos_find_package(Chipmunk CHIPMUNK REQUIRED)
  add_definitions(-DCC_ENABLE_CHIPMUNK_INTEGRATION=1)
  if(IOS OR MACOSX)
    # without this chipmunk will try to use apple defined geometry types, that conflicts with cocos
    add_definitions(-DCP_USE_CGPOINTS=0)
  endif()
  message(STATUS "Chipmunk come in")
else(USE_CHIPMUNK)
  add_definitions(-DCC_USE_PHYSICS=0)
endif(USE_CHIPMUNK)

# Box2d (not prebuilded, exists as source)
if(USE_BOX2D)
  if(USE_PREBUILT_LIBS)
    add_subdirectory(libs/liquidfun)
    set(Box2D_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}libs/liquidfun/liquidfun/liquidfun/Box2D)
    set(Box2D_LIBRARIES box2d)
  else()
    find_package(Box2D REQUIRED CONFIG)
    # actually Box2D in next line is not a library, it is target exported from Box2DConfig.cmake
    set(Box2D_LIBRARIES Box2D)
  endif()
  message(STATUS "Box2D include dirs: ${Box2D_INCLUDE_DIRS}")
  add_definitions(-DCC_ENABLE_BOX2D_INTEGRATION=1)
else()
  message(STATUS "Box2d not USE")
  add_definitions(-DCC_ENABLE_BOX2D_INTEGRATION=0)
endif(USE_BOX2D)

# Tinyxml2 (not prebuilded, exists as source)
if(USE_PREBUILT_LIBS)
  add_subdirectory(libs/cocos2d-x/external/tinyxml2)
  set(TinyXML2_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/libs/cocos2d-x/external/tinyxml2)
  set(TinyXML2_LIBRARIES tinyxml2)
else()
  cocos_find_package(TinyXML2 TinyXML2 REQUIRED)
endif()
message(STATUS "TinyXML2 include dirs: ${TinyXML2_INCLUDE_DIRS}")

# libjpeg
cocos_find_package(JPEG JPEG REQUIRED)
cocos_find_package(ZLIB ZLIB REQUIRED)


# minizip (we try to migrate to minizip from https://github.com/nmoinvaz/minizip)
# only msys2 currently provides package for this variant, all other
# dists have packages from zlib, thats very old for us.
# moreover our embedded version modified to quick provide
# functionality needed by cocos.
if(USE_PREBUILT_LIBS OR NOT MINGW)
  add_subdirectory(libs/cocos2d-x/external/unzip)
  set(MINIZIP_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/libs/cocos2d-x/external/unzip)
  set(MINIZIP_LIBRARIES unzip)
  message(STATUS "MINIZIP include dirs: ${MINIZIP_INCLUDE_DIRS}")
else()
  cocos_find_package(MINIZIP MINIZIP REQUIRED)
  # double check that we have needed functions
  include(CheckLibraryExists)
  check_library_exists(${MINIZIP_LIBRARIES} "unzGoToFirstFile2" "" MINIZIP_HAS_GOTOFIRSTFILE2)
  if(NOT MINIZIP_HAS_GOTOFIRSTFILE2)
    message(FATAL_ERROR "Minizip library on you system very old. Please use recent version from https://github.com/nmoinvaz/minizip or enable USE_PREBUILT_LIBS")
  endif()
  add_definitions(-DMINIZIP_FROM_SYSTEM)
endif()

cocos_find_package(PNG PNG REQUIRED)
cocos_find_package(TIFF TIFF REQUIRED)
cocos_find_package(WEBSOCKETS WEBSOCKETS REQUIRED)
cocos_find_package(CURL CURL REQUIRED)

# protobuf-lite (not prebuilded, exists as source)
# TODO: for now we can't use upstream protobuf because these files:
#  cocos/editor-support/cocostudio/CSParseBinary.pb.h
#  cocos/editor-support/cocostudio/CSParseBinary.pb.cc
# was generated by concrete version of protobuf compiler
# and source file not provided. So these files can be
# compiled only with our in-source version of protobuf-lite
## if(USE_PREBUILT_LIBS)
  add_subdirectory(libs/cocos2d-x/external/protobuf-lite)
  set(PROTOBUF_LITE_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/libs/cocos2d-x/external/protobuf-lite/src)
  set(PROTOBUF_LITE_LIBRARIES protobuf)
## else()
##  cocos_find_package(Protobuf REQUIRED PROTOBUF_LITE_LIBRARIES)
##  set(PROTOBUF_LITE_INCLUDE_DIRS ${PROTOBUF_INCLUDE_DIRS})
## endif()
message(STATUS "Protobuf lite libs: ${PROTOBUF_LITE_LIBRARIES}")
message(STATUS "Protobuf include dirs: ${PROTOBUF_LITE_INCLUDE_DIRS}")

# flatbuffers (not prebuilded, exists as source)
# TODO: for now we can't use upstream flatbuffers because these files:
#  cocos/editor-support/cocostudio/CSParseBinary_generated.h
# was generated by concrete version of flatbuffers compiler
# and source file not provided. So these files can be
# compiled only with our in-source version of flatbuffers
## if(USE_PREBUILT_LIBS)
#  add_subdirectory(external)
#  set(FLATBUFFERS_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/external)
#  set(FLATBUFFERS_LIBRARIES flatbuffers)
## else()
##  cocos_find_package(Flatbuffers REQUIRED FLATBUFFERS_LIBRARIES)
##  set(FLATBUFFERS_INCLUDE_DIRS ${FLATBUFFERS_INCLUDE_DIRS})
## endif()
#message(STATUS "Flatbuffers libs: ${FLATBUFFERS_LIBRARIES}")
#message(STATUS "Flatbuffers include dirs: ${FLATBUFFERS_INCLUDE_DIRS}")

# build for 3rd party libraries
if(LINUX OR APPLE)
add_subdirectory(libs/cocos2d-x/external/xxhash)
endif()

# libcocos2d.a
add_subdirectory(libs/cocos2d-x/cocos)

# build LiquidFun-EyeCandy and LiquidFun-Testbed
if(BUILD_LIQUIDFUN_TEST)
  add_subdirectory(samples/LiquidFun-EyeCandy)
  add_subdirectory(samples/LiquidFun-Testbed)
endif(BUILD_LIQUIDFUN_TEST)

# build GAFExamples
if(BUILD_GAFEXAMPLES)
  set(COCOS_ROOT ${CMAKE_CURRENT_SOURCE_DIR}/libs/cocos2d-x/cocos)
  set(GAF_ROOT ${CMAKE_CURRENT_SOURCE_DIR}/libs/GAF/Library)
  include(libs/GAF/Library/CMakeLists.txt)
  #add_subdirectory(libs/GAF/Library)
  add_subdirectory(libs/GAF/Examples)
endif(BUILD_GAFEXAMPLES)

