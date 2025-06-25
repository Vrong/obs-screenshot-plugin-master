cmake_minimum_required(VERSION 3.16...3.21)

#project(obs-screenshot-filter VERSION 1.4.0)
set(MODNAME obs-screenshot-filter)
add_library(obs-screenshot-filter MODULE)

#set(PROJECT_FULL_NAME "Screenshot filter")

set(PLUGIN_AUTHOR "synap5e")

# Add your custom source files here - header files are optional and only required for visibility
# e.g. in Xcode or Visual Studio
target_sources(${MODNAME} PRIVATE screenshot-filter.c)

# Import libobs as main plugin dependency
#find_package(libobs REQUIRED)
#include(cmake/ObsPluginHelpers.cmake)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/Modules/")

find_package(FFmpeg REQUIRED COMPONENTS avcodec avutil avformat swscale swresample)
target_include_directories(${MODNAME} PRIVATE ${FFMPEG_INCLUDE_DIRS})

target_link_libraries(${MODNAME} PRIVATE OBS::libobs wininet)

if(MSVC)
  target_include_directories(${MODNAME}
                             PRIVATE ${CMAKE_SOURCE_DIR}/../obs-studio/deps/w32-pthreads/)
  target_link_libraries(${MODNAME} PRIVATE OBS::w32-pthreads)
endif()

target_link_libraries(obs-screenshot-filter PRIVATE FFmpeg::avcodec FFmpeg::avutil FFmpeg::avformat
                                                    FFmpeg::swscale FFmpeg::swresample)

target_compile_options(${MODNAME} PRIVATE /WX-)
# --- Windows-specific build settings and tasks ---
if(OS_WINDOWS)
  configure_file(cmake/bundle/windows/installer-Windows.iss.in
                 ${CMAKE_BINARY_DIR}/installer-Windows.generated.iss)

  configure_file(cmake/bundle/windows/resource.rc.in ${CMAKE_BINARY_DIR}/obs-screenshot-filter.rc)
  target_sources(${MODNAME} PRIVATE ${CMAKE_BINARY_DIR}/obs-screenshot-filter.rc)

  if(MSVC)
  
	get_target_property(flags ${MODNAME} COMPILE_OPTIONS)
	if(flags)
		list(REMOVE_ITEM flags "/WX" "/WX-")
		set_target_properties(${MODNAME} PROPERTIES COMPILE_OPTIONS "${flags}")
	endif()
	
    target_compile_options(${MODNAME} PRIVATE /WX-)
  endif()
  # --- End of section ---
endif()
get_target_property(MAIN_CFLAGS ${MODNAME} COMPILE_OPTIONS)
# also see: COMPILE_DEFINITIONS INCLUDE_DIRECTORIES
message("-- xxxx Target compiler flags are: ${MAIN_CFLAGS}")

set_target_properties_obs(obs-screenshot-filter PROPERTIES FOLDER plugins PREFIX "")
#setup_plugin_target(obs-screenshot-filter)
