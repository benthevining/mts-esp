cmake_minimum_required (VERSION 3.25 FATAL_ERROR)

project (MTS-ESP
		 VERSION 0.0.1
		 DESCRIPTION "A simple but versatile C/C++ library for adding microtuning support to audio and MIDI plugins."
		 HOMEPAGE_URL "https://oddsound.com/usingmtsesp.php"
		 LANGUAGES CXX)

include (FetchContent)

FetchContent_Declare (MTS-ESP
					  GIT_REPOSITORY "https://github.com/ODDSound/MTS-ESP.git"
					  GIT_TAG "origin/main"
					  GIT_PROGRESS TRUE
					  OVERRIDE_FIND_PACKAGE)

FetchContent_MakeAvailable (MTS-ESP)

# client

add_library (MTSESP_Client)

add_library (oddsound::MTSESP_Client ALIAS MTSESP_Client)

set (client_dir "${mts-esp_SOURCE_DIR}/Client")

target_sources (MTSESP_Client
				PRIVATE "${client_dir}/libMTSClient.cpp"
				PUBLIC "$<BUILD_INTERFACE:${client_dir}/libMTSClient.h>"
					   "$<INSTALL_INTERFACE:include/libMTSClient.h>")

target_include_directories (MTSESP_Client PUBLIC
								"$<BUILD_INTERFACE:${client_dir}>"
								"$<INSTALL_INTERFACE:include>")

include (GNUInstallDirs)

install (FILES "${client_dir}/libMTSClient.h"
		 DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
		 COMPONENT mts_esp_client)

install (TARGETS MTSESP_Client
		 EXPORT mts_esp_client_targets
		 COMPONENT mts_esp_client)

install (EXPORT mts_esp_client_targets
		 DESTINATION .
		 NAMESPACE oddsound::
		 COMPONENT mts_esp_client)

include (CPackComponent)

cpack_add_component (mts_esp_client
					 DISPLAY_NAME "MTS-ESP client"
					 DESCRIPTION "MTS-ESP client library"
					 GROUP "MTS-ESP"
					 DEPENDS mts_esp_base)

# master

add_library (MTSESP_Master)

add_library (oddsound::MTSESP_Master ALIAS MTSESP_Master)

set (master_dir "${mts-esp_SOURCE_DIR}/Master")

target_sources (MTSESP_Master
				PRIVATE "${master_dir}/libMTSMaster.cpp"
				PUBLIC "$<BUILD_INTERFACE:${master_dir}/libMTSMaster.h>"
					   "$<INSTALL_INTERFACE:include/libMTSMaster.h>")

target_include_directories (MTSESP_Master PUBLIC
								"$<BUILD_INTERFACE:${master_dir}>"
								"$<INSTALL_INTERFACE:include>")

install (FILES "${master_dir}/libMTSMaster.h"
		 DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
		 COMPONENT mts_esp_master)

install (TARGETS MTSESP_Master
		 EXPORT mts_esp_master_targets
		 COMPONENT mts_esp_master)

install (EXPORT mts_esp_master_targets
		 DESTINATION .
		 NAMESPACE oddsound::
		 COMPONENT mts_esp_master)

add_library (libMTS UNKNOWN IMPORTED)

set (libMTS_dir "${mts-esp_SOURCE_DIR}/libMTS")

if(LINUX)
	set_target_properties (libMTS PROPERTIES IMPORTED_LOCATION "${libMTS_dir}/Linux/x86_64/libMTS.so")
elseif(WIN32)
	if(CMAKE_SIZEOF_VOID_P EQUAL 8)
		set_target_properties (libMTS PROPERTIES IMPORTED_LOCATION "${libMTS_dir}/Win/64bit/LIBMTS.dll")
	else()
		set_target_properties (libMTS PROPERTIES IMPORTED_LOCATION "${libMTS_dir}/Win/32bit/LIBMTS.dll")
	endif()
elseif(APPLE)
	if(NOT ("${CMAKE_SYSTEM_NAME}" STREQUAL "Darwin"))
		message (FATAL_ERROR "Unsupported platform - libMTS not built for current target platform!")
	endif()

	set_target_properties (libMTS PROPERTIES IMPORTED_LOCATION "${libMTS_dir}/Mac/x86_64_ARM/libMTS.dylib")
else()
	message (FATAL_ERROR "Unsupported platform - libMTS not built for current target platform!")
endif()

target_link_libraries (MTSESP_Master PRIVATE libMTS)

get_target_property (mts_lib_file libMTS IMPORTED_LOCATION)

if(LINUX)
	install (FILES "${mts_lib_file}"
			 DESTINATION "${CMAKE_INSTALL_LIBDIR}"
			 COMPONENT mts_esp_master)
elseif(WIN32)
	if(CMAKE_SIZEOF_VOID_P EQUAL 8)
		install (FILES "${mts_lib_file}"
				 DESTINATION "Program Files\\Common Files\\MTS-ESP"
				 COMPONENT mts_esp_master)

		install (FILES "${libMTS_dir}/Win/32bit/LIBMTS.dll"
				 DESTINATION "Program Files (x86)\\Common Files\\MTS-ESP"
				 COMPONENT mts_esp_master)
	else()
		install (FILES "${mts_lib_file}"
				 DESTINATION "Program Files\\Common Files\\MTS-ESP"
				 COMPONENT mts_esp_master)
	endif()
else() # Apple
	install (FILES "${mts_lib_file}"
			 DESTINATION "/Library/Application Support/MTS-ESP"
			 COMPONENT mts_esp_master)
endif()

cpack_add_component (mts_esp_master
					 DISPLAY_NAME "MTS-ESP master"
					 DESCRIPTION "MTS-ESP master library"
					 GROUP "MTS-ESP"
					 DEPENDS mts_esp_base)

#

include (CMakePackageConfigHelpers)

set (generated_version_file "${CMAKE_CURRENT_BINARY_DIR}/MTS-ESP-config-version.cmake")

write_basic_package_version_file ("${generated_version_file}"
								  VERSION "${PROJECT_VERSION}"
								  COMPATIBILITY SameMajorVersion)

set (generated_config_file "${CMAKE_CURRENT_BINARY_DIR}/MTS-ESP-config.cmake")

set (install_dest "${CMAKE_INSTALL_LIBDIR}/cmake/MTS-ESP")

configure_package_config_file ("${CMAKE_CURRENT_SOURCE_DIR}/MTS-ESP-config.cmake.in"
							   "${generated_config_file}"
							   INSTALL_DESTINATION "${install_dest}"
							   NO_SET_AND_CHECK_MACRO)

install (FILES "${generated_version_file}" "${generated_config_file}"
		 DESTINATION "${install_dest}"
		 COMPONENT mts_esp_base)

cpack_add_component (mts_esp_base
					 HIDDEN
					 GROUP "MTS-ESP")

cpack_add_component_group ("MTS-ESP"
						   DISPLAY_NAME "MTS-ESP"
						   DESCRIPTION "${PROJECT_DESCRIPTION}"
						   EXPANDED BOLD_TITLE)
