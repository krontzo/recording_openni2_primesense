cmake_minimum_required (VERSION 3.10 FATAL_ERROR)
project (tools_openni2)

set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

include_directories(${CMAKE_CURRENT_BINARY_DIR})
include_directories( ${PROJECT_SOURCE_DIR}/src )

set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/Modules/" ${CMAKE_MODULE_PATH})

# On macOS, Homebrew's VTK has an issue with JsonCpp
# ❯ brew list --versions | rg -i 'vtk|jsoncpp'
# jsoncpp 1.9.5
# vtk 9.2.6
if (CMAKE_HOST_SYSTEM_NAME MATCHES "Darwin")
    message("\n\nBuilding on macos\n\n")
    execute_process(
        COMMAND brew --prefix --installed vtk
        RESULT_VARIABLE BREW_VTK
        OUTPUT_VARIABLE BREW_VTK_PREFIX
        OUTPUT_STRIP_TRAILING_WHITESPACE
    )
    if (BREW_VTK EQUAL 0 AND EXISTS "${BREW_VTK_PREFIX}")
        set(BREW_VTK_LIB_CMAKE ${BREW_VTK_PREFIX}/lib/cmake)
        file(GLOB_RECURSE
            BREW_VTK_CMAKE_LST "${BREW_VTK_LIB_CMAKE}/*/FindJsonCpp.cmake")
        list(POP_FRONT BREW_VTK_CMAKE_LST BREW_VTK_CMAKE_DIR)
        cmake_path(GET BREW_VTK_CMAKE_DIR PARENT_PATH BREW_JSONCPP)
        message(STATUS "Hack: JsonCpp for VTK needs ${BREW_JSONCPP} in MODULE_PATH")
        set(CMAKE_MODULE_PATH "${BREW_JSONCPP}"  ${CMAKE_MODULE_PATH})
        find_package(JsonCpp REQUIRED)
    endif()
endif()


# PCL required
find_package(PCL 1.7 REQUIRED)
include_directories(${PCL_INCLUDE_DIRS})
link_directories(${PCL_LIBRARY_DIRS})
add_definitions(${PCL_DEFINITIONS})

message("\n\nFound PCL\n\n")

# Opencv required
find_package(OpenCV REQUIRED)
include_directories(${OPENCV_INCLUDE_DIRS})
link_directories(${OPENCV_LIBRARY_DIRS})
add_definitions(${OPENCV_DEFINITIONS})

message("\n\nFound OpenCV\n\n")

# Openni2 required

find_package(OpenNI2 REQUIRED)
include_directories( ${OPENNI2_INCLUDE_DIRS} )
add_definitions(${OPENNI2_DEFINITIONS})

find_package(Qt5 REQUIRED COMPONENTS Core Widgets)

#set the default path for built executables to the "bin" directory
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
#set the default path for built libraries to the "lib" directory
set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)

# set the sources needed
set( VIEWER_SOURCES
                src/main.cpp
		src/viewer.cpp
                src/viewer_error.cpp
		src/viewer_visualizer.cpp
		src/viewer_keyio.cpp
		src/viewer_pcl.cpp
		src/viewer_save.cpp
		src/sliders.cpp
)

# set QT headers
SET(QT_HEADERS include/sliders.h)

#set QT forms
SET(QT_FORMS src/sliders.ui)

# create moc for QT
#QT4_WRAP_CPP(QT_MOC ${QT_HEADERS})

# process ui
#QT4_WRAP_UI(QT_FORMS_HEADERS ${QT_FORMS})

ADD_EXECUTABLE(viewer ${VIEWER_SOURCES} ${QT_FORMS})


TARGET_LINK_LIBRARIES(viewer Qt5::Widgets ${PCL_LIBRARIES} ${OpenCV_LIBS} ${OPENNI2_LIBRARIES})


