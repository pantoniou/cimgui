cmake_minimum_required(VERSION 3.1)

if(NOT CMAKE_VERSION VERSION_LESS "3.02")
    cmake_policy(SET CMP0048 NEW)
    project(cimgui VERSION 1.0.0)	# version number
else()
    project(cimgui)
endif()

#general settings
file(GLOB IMGUI_SOURCES
    cimgui.cpp
    imgui/imgui.cpp
    imgui/imgui_draw.cpp
    imgui/imgui_demo.cpp
    imgui/imgui_widgets.cpp
)

set(IMGUI_STATIC "no" CACHE STRING "Build as a static library")
set(IMGUI_FREETYPE "no" CACHE STRING "Build with freetype library")
set(IMGUI_LIBRARIES )

option(CIMGUI_PKGCONFIG "Build and install pkgconfig support" ON)

if(CIMGUI_PKGCONFIG)
    include(GNUInstallDirs)
endif()

#add library and link
if (IMGUI_STATIC)
    add_library(cimgui STATIC ${IMGUI_SOURCES})
else (IMGUI_STATIC)
    add_library(cimgui SHARED ${IMGUI_SOURCES})
endif (IMGUI_STATIC)

target_compile_definitions(cimgui PUBLIC IMGUI_DISABLE_OBSOLETE_FUNCTIONS=1)
if (WIN32)
    target_compile_definitions(cimgui PUBLIC IMGUI_IMPL_API=extern\t\"C\"\t__declspec\(dllexport\))
else (WIN32)
    target_compile_definitions(cimgui PUBLIC IMGUI_IMPL_API=extern\t\"C\"\t)
endif (WIN32)

if(IMGUI_FREETYPE)
	find_package(PkgConfig REQUIRED)
	pkg_check_modules(FREETYPE REQUIRED freetype2)
	add_definitions("-DCIMGUI_FREETYPE=1")
else(IMGUI_FREETYPE)
	set(FREETYPE_LIBRARIES)
	set(FREETYPE_INCLUDE_DIRS)
	set(FREETYPE_CFLAGS_OTHER)
endif(IMGUI_FREETYPE)

target_include_directories(cimgui
	PUBLIC
		$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
		$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/imgui>
		$<INSTALL_INTERFACE:include/imgui>
		$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>)

# target_include_directories(cimgui PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/imgui)

target_link_libraries(cimgui ${FREETYPE_LIBRARIES})
target_include_directories(cimgui PUBLIC ${FREETYPE_INCLUDE_DIRS})
target_compile_options(cimgui PUBLIC ${FREETYPE_CFLAGS_OTHER})
# set_target_properties(cimgui PROPERTIES PREFIX "")
target_link_libraries(cimgui ${IMGUI_LIBRARIES})

#install
#install(TARGETS cimgui
#    RUNTIME DESTINATION  .
#    LIBRARY DESTINATION  .
#    ARCHIVE DESTINATION  .
#)

install(TARGETS cimgui
	EXPORT cimgui
	ARCHIVE DESTINATION lib)
install(FILES
	cimgui.h
	DESTINATION include/cimgui)

#test
set(CIMGUI_TEST "no" CACHE STRING "Enable compilation of a test unit based on imgui null")

if (CIMGUI_TEST)
  add_subdirectory(test)
endif ()

install(
    EXPORT cimgui
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/cimgui
    NAMESPACE cimgui::
    FILE cimguiTargets.cmake
    )

include(CMakePackageConfigHelpers)
configure_package_config_file(
    "Config.cmake.in"
    "cimguiConfig.cmake"
    INSTALL_DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/cimgui"
    PATH_VARS
      CMAKE_INSTALL_LIBDIR
    )

write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/cimguiConfigVersion.cmake"
    VERSION 1.0.0
    COMPATIBILITY SameMajorVersion
    )

### Install Config and ConfigVersion files
install(
    FILES "${CMAKE_CURRENT_BINARY_DIR}/cimguiConfig.cmake"
          "${CMAKE_CURRENT_BINARY_DIR}/cimguiConfigVersion.cmake"
    DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/cimgui"
    )

if(CIMGUI_PKGCONFIG)

    # collect libraries
    set(CIMGUI_DEP_LIBS "")
    if(IMGUI_FREETYPE)
        set(CIMGUI_DEP_LIBS "${CIMGUI_DEP_LIBS} -lfreetype")
    endif()

    configure_file(
        cmake/cimgui.pc.in
        "${CMAKE_BINARY_DIR}/cimgui.pc"
        @ONLY)
    install(
        FILES "${CMAKE_BINARY_DIR}/cimgui.pc"
	DESTINATION "${CMAKE_INSTALL_LIBDIR}/pkgconfig"
    )
endif()
