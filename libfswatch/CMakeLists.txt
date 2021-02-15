#
# Copyright (c) 2014-2016 Enrico M. Crisostomo, 2021 Mikhail Komarov <nemo@nil.foundation>
#
# This program is free software; you can redistribute it and/or modify it under
# the terms of the GNU General Public License as published by the Free Software
# Foundation; either version 3, or (at your option) any later version.
#
# This program is distributed in the hope that it will be useful, but WITHOUT
# ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS
# FOR A PARTICULAR PURPOSE.  See the GNU General Public License for more
# details.
#
# You should have received a copy of the GNU General Public License along with
# this program.  If not, see <http://www.gnu.org/licenses/>.
#

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake"
    "${CMAKE_CURRENT_LIST_DIR}/cmake/packages"
    "${CMAKE_CURRENT_LIST_DIR}/cmake/modules/share/modules/cmake")

cm_find_package(CM)

include(CMConfig)
include(CMDeploy)
include(CMSetupVersion)

macro(cm_find_package NAME)
    if(NOT "${NAME}" MATCHES "^${CMAKE_WORKSPACE_NAME}_.*$" AND NOT "${NAME}" STREQUAL CM)
        find_package(${ARGV})
    else()
        set(${ARGV0}_FOUND ON CACHE BOOL "")
    endif()
endmacro()

cm_project(libfswatch LANGUAGES CXX)

cm_setup_version(VERSION 0.1.0)

list(APPEND ${CURRENT_PROJECT_NAME}_HEADERS
    include/libfswatch/c/cevent.h
    include/libfswatch/c/cfilter.h
    include/libfswatch/c/cmonitor.h
    include/libfswatch/c/error.h
    include/libfswatch/c/libfswatch.h
    include/libfswatch/c/libfswatch_log.h
    include/libfswatch/c/libfswatch_types.h

    include/libfswatch/c++/event.hpp
    include/libfswatch/c++/filter.hpp
    include/libfswatch/c++/libfswatch_exception.hpp
    include/libfswatch/c++/libfswatch_map.hpp
    include/libfswatch/c++/libfswatch_set.hpp
    include/libfswatch/c++/monitor.hpp
    include/libfswatch/c++/monitor_factory.hpp
    include/libfswatch/c++/path_utils.hpp
    include/libfswatch/c++/poll_monitor.hpp
    include/libfswatch/c++/string/string_utils.hpp
    include/libfswatch/gettext.h
    include/libfswatch/gettext_defs.h)

list(APPEND ${CURRENT_PROJECT_NAME}_SOURCES
    src/libfswatch/c/cevent.cpp
    src/libfswatch/c/libfswatch.cpp
    src/libfswatch/c/libfswatch_log.cpp
    src/libfswatch/c++/event.cpp
    src/libfswatch/c++/filter.cpp
    src/libfswatch/c++/libfswatch_exception.cpp
    src/libfswatch/c++/monitor.cpp
    src/libfswatch/c++/monitor_factory.cpp
    src/libfswatch/c++/path_utils.cpp
    src/libfswatch/c++/poll_monitor.cpp
    src/libfswatch/c++/string/string_utils.cpp)

set(${CURRENT_PROJECT_NAME}_COMPILE_DEFINITIONS "")

if (EXISTS ../libfswatch_config.h)
    list(APPEND ${CURRENT_PROJECT_NAME}_HEADERS ../libfswatch_config.h)

    list(APPEND ${CURRENT_PROJECT_NAME}_COMPILE_DEFINITIONS -DHAVE_CONFIG_H)
endif ()

include(CheckIncludeFiles)

CHECK_INCLUDE_FILES(sys/inotify.h HAVE_SYS_INOTIFY_H)

if(HAVE_SYS_INOTIFY_H)
    list(APPEND ${CURRENT_PROJECT_NAME}_HEADERS
        include/libfswatch/c++/inotify_monitor.hpp)
    list(APPEND ${CURRENT_PROJECT_NAME}_SOURCES
        src/libfswatch/c++/inotify_monitor.cpp)
    list(APPEND ${CURRENT_PROJECT_NAME}_COMPILE_DEFINITIONS
        -DHAVE_SYS_INOTIFY_H)
endif(HAVE_SYS_INOTIFY_H)

CHECK_INCLUDE_FILES(sys/event.h HAVE_SYS_EVENT_H)

if (HAVE_SYS_EVENT_H)
    list(APPEND ${CURRENT_PROJECT_NAME}_HEADERS
        include/libfswatch/c++/kqueue_monitor.hpp)
    list(APPEND ${CURRENT_PROJECT_NAME}_SOURCES
        src/libfswatch/c++/kqueue_monitor.cpp)
    list(APPEND ${CURRENT_PROJECT_NAME}_COMPILE_DEFINITIONS
        -DHAVE_SYS_EVENT_H)
endif(HAVE_SYS_EVENT_H)

CHECK_INCLUDE_FILES(port.h HAVE_PORT_H)

if (HAVE_PORT_H)
    list(APPEND ${CURRENT_PROJECT_NAME}_HEADERS
        include/libfswatch/c++/fen_monitor.hpp)
    list(APPEND ${CURRENT_PROJECT_NAME}_SOURCES
        src/libfswatch/c++/fen_monitor.cpp)

    list(APPEND ${CURRENT_PROJECT_NAME}_COMPILE_DEFINITIONS 
        -DHAVE_PORT_H)
endif(HAVE_PORT_H)

if (WIN32)
    CHECK_INCLUDE_FILES(sys/cygwin.h HAVE_CYGWIN)

    list(APPEND ${CURRENT_PROJECT_NAME}_COMPILE_DEFINITIONS -DHAVE_STRUCT_STAT_ST_MTIME)

    if (HAVE_CYGWIN)
        list(APPEND ${CURRENT_PROJECT_NAME}_HEADERS
            include/libfswatch/c++/windows/win_directory_change_event.hpp
            include/libfswatch/c++/windows/win_error_message.hpp
            include/libfswatch/c++/windows/win_handle.hpp
            include/libfswatch/c++/windows/win_paths.hpp
            include/libfswatch/c++/windows/win_strings.hpp
            include/libfswatch/c++/windows_monitor.hpp)

        list(APPEND ${CURRENT_PROJECT_NAME}_SOURCES
            src/libfswatch/c++/windows/win_directory_change_event.cpp
            src/libfswatch/c++/windows/win_error_message.cpp
            src/libfswatch/c++/windows/win_handle.cpp
            src/libfswatch/c++/windows/win_paths.cpp
            src/libfswatch/c++/windows/win_strings.cpp
            src/libfswatch/c++/windows_monitor.cpp)

        add_definitions(-DHAVE_WINDOWS)
    endif(HAVE_CYGWIN)
elseif(APPLE)
    CHECK_INCLUDE_FILES(CoreServices/CoreServices.h HAVE_FSEVENTS_FILE_EVENTS)

    list(APPEND ${CURRENT_PROJECT_NAME}_COMPILE_DEFINITIONS -DHAVE_STRUCT_STAT_ST_MTIME)

    if (HAVE_FSEVENTS_FILE_EVENTS)
        find_library(CORESERVICES_LIBRARY CoreServices)
        set(EXTRA_LIBS ${CORESERVICES_LIBRARY})

        list(APPEND ${CURRENT_PROJECT_NAME}_HEADERS
            include/libfswatch/c++/fsevents_monitor.hpp)

        list(APPEND ${CURRENT_PROJECT_NAME}_SOURCES
            src/libfswatch/c++/fsevents_monitor.cpp)
    endif(HAVE_FSEVENTS_FILE_EVENTS)

    list(APPEND ${CURRENT_PROJECT_NAME}_COMPILE_DEFINITIONS -DHAVE_STRUCT_STAT_ST_MTIMESPEC)
else () # Other Unix I suppose
    list(APPEND ${CURRENT_PROJECT_NAME}_COMPILE_DEFINITIONS -DHAVE_STRUCT_STAT_ST_MTIMESPEC)
endif()

add_library(${CURRENT_PROJECT_NAME} 
    ${${CURRENT_PROJECT_NAME}_HEADERS} 
    ${${CURRENT_PROJECT_NAME}_SOURCES})
target_include_directories(${CURRENT_PROJECT_NAME} PUBLIC 
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>)
target_link_libraries(${CURRENT_PROJECT_NAME} PUBLIC ${EXTRA_LIBS})
target_compile_definitions(${CURRENT_PROJECT_NAME} PRIVATE 
    ${${CURRENT_PROJECT_NAME}_COMPILE_DEFINITIONS})

cm_deploy(TARGETS ${CURRENT_PROJECT_NAME} 
    INCLUDE include)
