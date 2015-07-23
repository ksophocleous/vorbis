cmake_minimum_required(VERSION 3.0.0)

# Extract project version from configure.ac
file(READ configure.ac CONFIGURE_AC_CONTENTS)
string(REGEX MATCH "AC_INIT\\(\\[libvorbis\\],\\[([0-9]*).([0-9]*).([0-9]*)" DUMMY ${CONFIGURE_AC_CONTENTS})
set(PROJECT_VERSION_MAJOR ${CMAKE_MATCH_1})
set(PROJECT_VERSION_MINOR ${CMAKE_MATCH_2})
set(PROJECT_VERSION_PATCH ${CMAKE_MATCH_3})
set(PROJECT_VERSION ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH})

project(vorbis VERSION ${PROJECT_VERSION})

set(CMAKE_DEBUG_POSTFIX "d")

# Required modules
include(GNUInstallDirs)
include(CheckIncludeFiles)
include(CMakePackageConfigHelpers)

# Build options
option(BUILD_SHARED_LIBS "Build shared library" OFF)
option(VORBIS_LINK_WITH_STATIC_OGG "Link with static libogg" ON)
option(VORBIS_BUILD_EXAMPLES "Build examples" OFF)
if (MSVC)
    option(VORBIS_USE_STATIC_RUNTIME "Use static msvc runtime" OFF)
endif()
if(APPLE)
    option(BUILD_FRAMEWORK "Build Framework bundle for OSX" OFF)
endif()

if (BUILD_FRAMEWORK AND NOT BUILD_SHARED_LIBS)
    message(STATUS "Building framework... so enabling shared library build")
    set(BUILD_SHARED_LIBS TRUE)
endif()

# statically link msvc runtime
if (MSVC AND VORBIS_USE_STATIC_RUNTIME)
	foreach(flag_var
		CMAKE_C_FLAGS CMAKE_C_FLAGS_DEBUG CMAKE_C_FLAGS_RELEASE
		CMAKE_C_FLAGS_MINSIZEREL CMAKE_C_FLAGS_RELWITHDEBINFO)

		if(${flag_var} MATCHES "/MD")
			string(REGEX REPLACE "/MD" "/MT" ${flag_var} "${${flag_var}}")
		endif(${flag_var} MATCHES "/MD")
	endforeach(flag_var)
endif()

if(VORBIS_LINK_WITH_STATIC_OGG)
    set(OGG_LIB_NAME "libogg_static")
else()
    set(OGG_LIB_NAME "libogg")
endif()

# Helper function to get version-info
function(get_version_info result current_var_name age_var_name revision_var_name)
    string(REGEX MATCH "${current_var_name}=([0-9]*)" DUMMY ${CONFIGURE_AC_CONTENTS})
    set(VERSION_INFO_CURRENT ${CMAKE_MATCH_1})

    string(REGEX MATCH "${age_var_name}=([0-9]*)" DUMMY ${CONFIGURE_AC_CONTENTS})
    set(VERSION_INFO_AGE ${CMAKE_MATCH_1})

    string(REGEX MATCH "${revision_var_name}=([0-9]*)" DUMMY ${CONFIGURE_AC_CONTENTS})
    set(VERSION_INFO_REVISION ${CMAKE_MATCH_1})

    math(EXPR VERSION_INFO_CURRENT_MINUS_AGE "${VERSION_INFO_CURRENT} - ${VERSION_INFO_AGE}")

    set(${result} "${VERSION_INFO_CURRENT_MINUS_AGE}.${VERSION_INFO_AGE}.${VERSION_INFO_REVISION}" PARENT_SCOPE)
endfunction()

# Helper function to configure pkg-config files
function(configure_pkg_config_file pkg_config_file_in)
    set(prefix ${CMAKE_INSTALL_PREFIX})
    set(exec_prefix ${CMAKE_INSTALL_FULL_BINDIR})
    set(libdir ${CMAKE_INSTALL_FULL_LIBDIR})
    set(includedir ${CMAKE_INSTALL_FULL_INCLUDEDIR})
    set(VERSION ${PROJECT_VERSION})
    string(REPLACE ".in" "" pkg_config_file ${pkg_config_file_in})
    configure_file(${CMAKE_CURRENT_SOURCE_DIR}/${pkg_config_file_in}
        ${CMAKE_CURRENT_BINARY_DIR}/${pkg_config_file} @ONLY)
endfunction()

message(STATUS "Configuring ${PROJECT_NAME} ${PROJECT_VERSION}")

# Find ogg dependency
find_package(ogg REQUIRED CONFIG)
# find_package(PkgConfig QUIET)
# pkg_check_modules(PC_OGG QUIET ogg)
# find_path(OGG_INCLUDE_DIRS NAMES ogg/ogg.h HINTS ${PC_OGG_INCLUDE_DIRS} PATH_SUFFIXES ogg)
# find_library(OGG_LIBRARIES NAMES ogg HINTS ${PC_OGG_LIBRARY_DIRS})
# include(FindPackageHandleStandardArgs)
# find_package_handle_standard_args(OGG DEFAULT_MSG OGG_INCLUDE_DIRS OGG_LIBRARIES)

add_subdirectory(lib)

if(VORBIS_BUILD_EXAMPLES)
    add_subdirectory(examples)
endif()

configure_pkg_config_file(vorbis.pc.in)
configure_pkg_config_file(vorbisfile.pc.in)

install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/vorbis.pc
    ${CMAKE_CURRENT_BINARY_DIR}/vorbisfile.pc
    DESTINATION lib/pkgconfig)

install(EXPORT ${PROJECT_NAME}Config
	DESTINATION lib/cmake)
