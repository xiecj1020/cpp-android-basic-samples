# Copyright (C) 2017 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
##

cmake_minimum_required(VERSION 3.4.1)

# Import the GPGS C++ SDK library
add_library(gpg_sdk STATIC IMPORTED)
set_target_properties(gpg_sdk PROPERTIES IMPORTED_LOCATION
    ${GPG_SDK_PATH}/lib/c++/${ANDROID_ABI}/libgpg.a)

# build native_app_glue as a static lib
add_library(native_app_glue STATIC
    ${ANDROID_NDK}/sources/android/native_app_glue/android_native_app_glue.c)

# build cpufeatures as a static lib
add_library(cpufeatures STATIC
             ${ANDROID_NDK}/sources/android/cpufeatures/cpu-features.c)

#include the native part of JUI Helper
add_subdirectory (${JUI_HELPER_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/libs/${ANDROID_ABI}/libjuihelper")

#include the native part of NDKHelper
add_subdirectory (${NDK_HELPER_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/libs/${ANDROID_ABI}/libndkhelper")

#JSON library
add_library(json STATIC
    ${JSON_PATH}/src/lib_json/json_writer.cpp
 ${JSON_PATH}/src/lib_json/json_reader.cpp
 ${JSON_PATH}/src/lib_json/json_value.cpp
 )
 target_include_directories(json PRIVATE
      ${JSON_PATH}/include
      )

# Export ANativeActivity_onCreate(),
# Refer to: https://github.com/android-ndk/ndk/issues/381.
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -u ANativeActivity_onCreate")


add_library(TBMPSkeletonNativeActivity
             SHARED
             src/main/cpp/TBMPSkeletonNativeActivity.cpp
             src/main/cpp/TBMPSkeletonNativeActivity_Engine.cpp
             ${TEAPOT_RENDERER_PATH}/TeapotRenderer.cpp
           )

target_include_directories(TBMPSkeletonNativeActivity PRIVATE
     ${ANDROID_NDK}/sources/android/native_app_glue
     ${ANDROID_NDK}/sources/android/cpufeatures
     ${GPG_SDK_PATH}/include
     ${JSON_PATH}/include
     ${juihelper_SOURCE_DIR}/src/main/cpp
     ${ndkhelper_SOURCE_DIR}/src/main/cpp
     ${TEAPOT_RENDERER_PATH}
)

# Specifies libraries CMake should link to your target library. You
# can link multiple libraries, such as libraries you define in this
# build script, prebuilt third-party libraries, or system libraries.

target_link_libraries(TBMPSkeletonNativeActivity
                       gpg_sdk
                       native_app_glue
                       cpufeatures
                       juihelper
                       ndkhelper
                       json
                       log
                       android
                       EGL
                       GLESv2
                       z)
