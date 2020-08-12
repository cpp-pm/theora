cmake_minimum_required(VERSION 3.2)

option(HUNTER_ENABLED "Enable Hunter package manager" OFF)

if(HUNTER_ENABLED)
	include(cmake/HunterGate.cmake)
	HunterGate(
		URL "https://github.com/cpp-pm/hunter/archive/v0.23.263.tar.gz"
		SHA1 "9b96c376bf3526163e81b7adb8db5a0ea8c7e1bf"
	)
endif()

project(theora VERSION 1.1.1)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules")

if(HUNTER_ENABLED)
	hunter_add_package(libogg)
	find_package(libogg CONFIG REQUIRED)
	set(OGG_LIBRARIES libogg::ogg)
	get_target_property(OGG_INCLUDE_DIRS libogg::ogg INTERFACE_INCLUDE_DIRECTORIES)
else()
	find_package(Ogg REQUIRED)
endif()

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

set(config_install_dir "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}")
set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

add_subdirectory(lib)

install(
	DIRECTORY ${CMAKE_SOURCE_DIR}/include/
	DESTINATION ${CMAKE_INSTALL_FULL_INCLUDEDIR}
	FILES_MATCHING PATTERN "*.h*"
)

write_basic_package_version_file(
	"${version_config}" COMPATIBILITY SameMajorVersion
)

configure_package_config_file(
	"cmake/template/Config.cmake.in"
	"${project_config}"
	INSTALL_DESTINATION "${config_install_dir}"
)

install(
	FILES "${project_config}" "${version_config}"
	DESTINATION "${config_install_dir}"
)

install(
	EXPORT "${targets_export_name}"
	NAMESPACE "${namespace}"
	DESTINATION "${config_install_dir}"
)
