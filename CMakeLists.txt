# Copyright (c) 2009-2024 LG Electronics, Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
# SPDX-License-Identifier: Apache-2.0

cmake_minimum_required(VERSION 2.8.7)
project(ApplicationInstallerUtility C CXX)

# In the days before Open webOS we used ipkg to manage packages.
#
# The following configuration variable lets us select the correct paths
# for the package manager in use.
#
# We default to oe-core's opkg package manager
if(WEBOS_USE_LEGACY_PACKAGE_MANAGER)
    set(PACKAGE_MANAGER "ipkg" CACHE STRING "Packaging tool to use in updates")
else()
    set(PACKAGE_MANAGER "opkg" CACHE STRING "Packaging tool to use in updates")
endif()

include(webOS/webOS)

webos_modules_init(1 0 0 QUALIFIER RC4)
webos_component(3 0 0)

include(FindPkgConfig)

pkg_check_modules(GLIB2 REQUIRED glib-2.0)
include_directories(${GLIB2_INCLUDE_DIRS})
webos_add_compiler_flags(ALL ${GLIB2_CFLAGS_OTHER})

# check if we have PmLogLib
pkg_check_modules(PMLOGLIB REQUIRED PmLogLib)
include_directories(${PMLOGLIB_INCLUDE_DIRS})
webos_add_compiler_flags(ALL ${PMLOGLIB_CFLAGS_OTHER})

# Turn on all warnings except deprecated declarations and make them into errors
webos_add_compiler_flags(ALL -Wall -Wno-deprecated-declarations)

# Require that all undefined symbols are satisfied by the libraries from target_link_libraries()
webos_add_linker_options(ALL --no-undefined)

# XXX_ The libraries that come with gcc fail when --no-allow-shlib-undefined is specified
#webos_add_linker_options(ALL --no-allow-shlib-undefined)

# install the script files
set (SCRIPT_LOCATION files/support_scripts)
webos_configure_source_files(SCRIPT_FILES 
                     ${SCRIPT_LOCATION}/pmServicePostInstall.sh
                     ${SCRIPT_LOCATION}/pmServicePostRemove.sh)
install(PROGRAMS ${SCRIPT_LOCATION}/find_file_in_archive.sh DESTINATION ${WEBOS_INSTALL_BINDIR})
install(PROGRAMS ${SCRIPT_FILES} DESTINATION ${WEBOS_INSTALL_BINDIR})

include_directories(src)
webos_configure_source_files(SOURCE_FILES src/Main.cpp)
add_executable(ApplicationInstallerUtility ${SOURCE_FILES})

target_link_libraries(ApplicationInstallerUtility
                      rt
                      ${GLIB2_LDFLAGS}
                      ${PMLOGLIB_LDFLAGS}
                       )

webos_build_program()

