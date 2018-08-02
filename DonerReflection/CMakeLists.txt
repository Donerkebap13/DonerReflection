cmake_minimum_required(VERSION 3.6)
set(MAJOR_VERSION 1)
set(MINOR_VERSION 0)
set(PATCH_VERSION 0)
set(CURRENT_VERSION "${MAJOR_VERSION}.${MINOR_VERSION}.${PATCH_VERSION}")
set(CMAKE_CXX_EXTENSIONS OFF)

include(CheckCXXSourceCompiles)

set(project_name "DonerReflection")
set(ide_group "${project_name}")

project("${project_name}")

message(STATUS "${project_name} version ${CURRENT_VERSION}")

set(header_file "${CMAKE_CURRENT_SOURCE_DIR}/include/donerreflection/DonerReflection.h")
source_group ("include\\donerreflection" FILES "${header_file}")

include_directories("include")

# Library installation directory
if(NOT DEFINED CMAKE_INSTALL_LIBDIR)
    set(CMAKE_INSTALL_LIBDIR lib)
endif()
set(libdir ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR})

add_library("${project_name}" INTERFACE)
target_sources("${project_name}" INTERFACE "${header_file}")

target_compile_features("${project_name}" INTERFACE cxx_std_14)

target_include_directories("${project_name}" INTERFACE
    "${CMAKE_CURRENT_SOURCE_DIR}/include"
)

set(install_libs "${project_name}")
set_property(TARGET "${project_name}" APPEND PROPERTY INTERFACE_INCLUDE_DIRECTORIES
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>)

add_definitions(-D_CRT_SECURE_NO_WARNINGS)

target_compile_features("${project_name}" INTERFACE cxx_auto_type)
target_compile_features("${project_name}" INTERFACE cxx_nullptr)
target_compile_features("${project_name}" INTERFACE cxx_static_assert)
target_compile_features("${project_name}" INTERFACE cxx_decltype)
target_compile_features("${project_name}" INTERFACE cxx_constexpr)
target_compile_features("${project_name}" INTERFACE cxx_sizeof_member)
target_compile_features("${project_name}" INTERFACE cxx_variadic_templates)
target_compile_features("${project_name}" INTERFACE cxx_rvalue_references)
target_compile_features("${project_name}" INTERFACE cxx_long_long_type)
target_compile_features("${project_name}" INTERFACE cxx_lambdas)
target_compile_features("${project_name}" INTERFACE cxx_func_identifier)

if (NOT WINDOWS OR CYGWIN)
    set(DonerReflection_libs -lDonerReflection)

    install(
        FILES ${CMAKE_CURRENT_BINARY_DIR}/DonerReflection.pc
        DESTINATION "${libdir}/pkgconfig"
        )
endif()

install(
	DIRECTORY "${project_name}"
	DESTINATION "include"
	FILES_MATCHING PATTERN "*.h"
	)

install(
	TARGETS "${install_libs}"
	LIBRARY DESTINATION "${libdir}"
	ARCHIVE DESTINATION "${libdir}"
	RUNTIME DESTINATION "bin"
	)
	