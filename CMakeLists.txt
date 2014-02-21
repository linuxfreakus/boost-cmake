cmake_minimum_required(VERSION 2.8.11.2)

project(boost-cmake)

#find boost directory
set(boost_root ${CMAKE_CURRENT_SOURCE_DIR}/boost_1_55_0 CACHE PATH "Boost root")

if(NOT EXISTS ${boost_root})
message(FATAL_ERROR
  "${boost_root} does not exist!
    Please download and extract the sources!")
endif()

message(STATUS "find root path is: ${CMAKE_FIND_ROOT_PATH}")

set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR})
find_package(ICU)

set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS} -DNO_BZIP2")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DNO_BZIP2")

set(Boost_INCLUDE_DIRS ${boost_root} ${ICU_INCLUDE_DIRS})
set(Boost_LIBRARIES)
set(Boost_LINK_DIRS ${LIBRARY_OUTPUT_PATH} ${ICU_LIBRARIES} ${ICU_I18N_LIBRARIES})

if (ICU_FOUND)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DBOOST_HAS_ICU")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DBOOST_HAS_ICU")
    set(Boost_INCLUDE_DIRS ${Boost_INCLUDE_DIRS} ${ICU_INCLUDE_DIRS})
    set(Boost_LINK_DIRS ${Boost_LINK_DIRS} ${ICU_LIBRARIES} ${ICU_I18N_LIBRARIES})
endif()

macro(add_boost_lib name)
    file(GLOB_RECURSE lib_srcs "${boost_root}/libs/${name}/src/*.cpp")
    foreach(arg ${ARGV})
        if("${arg}" MATCHES "^\\-")
            string(REGEX REPLACE "^\\-" "${boost_root}/libs/${name}/src/" arg "${arg}")
            list(REMOVE_ITEM lib_srcs "${arg}")
        endif()
    endforeach()
    add_library( boost_${name} ${lib_srcs} )
    list(APPEND Boost_LIBRARIES boost_${name})
    install(TARGETS boost_${name} DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)
endmacro()

macro(add_boost_lib_subdir name src_subdir)
    file(GLOB_RECURSE lib_srcs ${boost_root}/libs/${name}/src/${src_subdir}/*.cpp)
    add_library( boost_${name} ${lib_srcs} )
    list(APPEND Boost_LIBRARIES boost_${name})
    install(TARGETS boost_${name} DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)
endmacro()

include_directories(${boost_root} ${CMAKE_CURRENT_SOURCE_DIR})
install(DIRECTORY ${boost_root}/boost DESTINATION ${CMAKE_INSTALL_PREFIX}/include)

# windows only
list(APPEND filesystem_excludes "-windows_file_codecvt.cpp")

# we disabled bzip2
list(APPEND iostreams_excludes "-bzip2.cpp")

# windows only
list(APPEND program_options_excludes "-winmain.cpp")

# windows or debug info only
list(APPEND regex_excludes "-regex_debug.cpp" "-usinstances.cpp" "-w32_regex_traits.cpp")

# if we cannot find icu then do not bother with the file
if (NOT ICU_FOUND)
    list(APPEND regex_excludes "-icu.cpp")
endif()

# contain no implementation only doc comments
list(APPEND date_time_excludes "-gregorian/gregorian_types.cpp" "-posix_time/posix_time_types.cpp")

add_boost_lib(system)
add_boost_lib(filesystem ${filesystem_excludes})
add_boost_lib(iostreams ${iostreams_excludes})
add_boost_lib(program_options ${program_options_excludes})
add_boost_lib(regex ${regex_excludes})
add_boost_lib(signals)
add_boost_lib_subdir(thread pthread)
add_boost_lib(date_time ${date_time_excludes})

configure_file(${CMAKE_SOURCE_DIR}/BoostConfig.cmake.in
               ${CMAKE_BINARY_DIR}/BoostConfig.cmake @ONLY)
