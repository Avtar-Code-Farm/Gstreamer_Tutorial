cmake_minimum_required(VERSION 3.0.0)
project(GstreamerLearning VERSION 0.1.0)

set(CMAKE_CXX_STANDARD 14) #setting C++ 14 standard
find_package(PkgConfig) #finding pkg-config is a helper tool

#using pkg-config to getting Gstreamer
find_package (PkgConfig REQUIRED)
pkg_check_modules(GTK3 REQUIRED gtk+-2.0)

pkg_search_module(GLIB REQUIRED glib-2.0) 
pkg_check_modules(GSTREAMER REQUIRED gstreamer-1.0)
pkg_check_modules(GST_APP REQUIRED gstreamer-app-1.0)
pkg_check_modules(GST_VIDEO REQUIRED gstreamer-video-1.0)



include_directories (${GTK2_INCLUDE_DIRS})

include_directories("/Library/Frameworks/GStreamer.framework/Headers/")

link_directories (${GTK2_LIBRARY_DIRS})
link_directories("/Library/Frameworks/GStreamer.framework/Headers/")

pkg_search_module(GST REQUIRED gstreamer-1.0>=1.4
        gstreamer-sdp-1.0>=1.4
        gstreamer-app-1.0>=1.4
        gstreamer-video-1.0>=1.4
        )


#including GStreamer header files directory
include_directories(
        ${GSTREAMER_INCLUDE_DIRS}
)

#linking GStreamer library directory
link_directories(
        ${GLIB_LIBRARY_DIRS}
        ${GSTREAMER_LIBRARY_DIRS}
        ${GTK2_INCLUDE_DIRS} ${GST_INCLUDE_DIRS}
)

#building target executable
add_executable(${PROJECT_NAME} main.cpp)

add_definitions (${GTK2_CFLAGS_OTHER})

#linking Gstreamer library with target executable
target_link_libraries(${PROJECT_NAME} ${GSTREAMER_LIBRARIES} ${GTK2_LIBRARIES} ${GST_LIBRARIES} )