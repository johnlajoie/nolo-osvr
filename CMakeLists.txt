cmake_minimum_required(VERSION 2.8.12)
project(NoloOSVRPlugin) # Change this line.

# So we can find our FindHIDAPI.cmake file
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake" "${LOCAL_CMAKE_MODULE_DIR}")

# This looks for an osvrConfig.cmake file - most of the time it can be
# autodetected but you might need to create/extend CMAKE_PREFIX_PATH to include something like
# C:/Users/Ryan/Desktop/build/OSVR-Core-vc12 or
# C:/Users/Ryan/Downloads/OSVR-Core-Snapshot-v0.1-406-gaa55515-build54-vs12-32bit
# in the CMake GUI or command line.
find_package(osvr REQUIRED)

# Be able to find our generated header file.
include_directories("${CMAKE_CURRENT_BINARY_DIR}")

if (WIN32)
	# This generates a header file, from the named json file, containing a string literal
	# named com_osvr_Nolo_json (not null terminated)
	# The file must be added as a source file to some target (as below) to be generated.
	osvr_convert_json(com_osvr_Nolo_json
    	com_osvr_Nolo_win.json
    	"${CMAKE_CURRENT_BINARY_DIR}/com_osvr_Nolo_json.h")

	# This is just a helper function wrapping CMake's add_library command that
	# sets up include dirs, libraries, and naming convention (no leading "lib")
	# for an OSVR plugin. It also installs the plugin into the right directory.
	# Pass as many source files as you need. See osvrAddPlugin.cmake for full docs.
	osvr_add_plugin(NAME com_osvr_Nolo
		CPP # indicates we'd like to use the C++ wrapper
		SOURCES
		com_osvr_Nolo_win.cpp
		"${CMAKE_CURRENT_BINARY_DIR}/com_osvr_Nolo_json.h")
	# If you use other libraries, find them and add a line like:
	find_path(NOLO_API_INCLUDE nolo_api.h)
	include_directories("${NOLO_API_INCLUDE}")
	find_library(NOLO_API_LIBRARY NoLo_USBHID.lib)
	target_link_libraries(com_osvr_Nolo "${NOLO_API_LIBRARY}")
else ()

	osvr_convert_json(com_osvr_Nolo_json
    	com_osvr_Nolo.json
    	"${CMAKE_CURRENT_BINARY_DIR}/com_osvr_Nolo_json.h")

	osvr_add_plugin(NAME com_osvr_Nolo
		CPP # indicates we'd like to use the C++ wrapper
		SOURCES
		com_osvr_Nolo.cpp
		"${CMAKE_CURRENT_BINARY_DIR}/com_osvr_Nolo_json.h")
	# If you use other libraries, find them and add a line like:
	find_package(HIDAPI REQUIRED)
	include_directories("${HIDAPI_INCLUDE_DIRS}")
	target_link_libraries(com_osvr_Nolo "${HIDAPI_LIBRARIES}")
endif ()


