cmake_minimum_required (VERSION 3.0)

# nvcc GCC 5.3 fix
list(APPEND CUDA_NVCC_FLAGS "-D_FORCE_INLINES" )

# Setup of the dependencies
LIST(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")

SET(OSX FALSE)
IF(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
        SET(OSX TRUE)
        INCLUDE_DIRECTORIES("/usr/local/cuda/include")
        LINK_DIRECTORIES("/usr/local/cuda/lib/")
ENDIF(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")

# OPENGL
FIND_PACKAGE(OpenGL REQUIRED)
INCLUDE_DIRECTORIES(${OPENGL_INCLUDE_DIR})

# Qt library
FIND_PACKAGE(Qt5OpenGL REQUIRED)
FIND_PACKAGE(Qt5Widgets REQUIRED)
INCLUDE_DIRECTORIES(${QT_QTOPENGL_INCLUDE_DIR})

# EIGEN
FIND_PACKAGE(PkgConfig REQUIRED)
PKG_SEARCH_MODULE(EIGEN REQUIRED eigen3)
INCLUDE_DIRECTORIES(${EIGEN_INCLUDE_DIRS})
LINK_DIRECTORIES(${EIGEN_LIBRARY_DIRS})

# Protobuf
FIND_PACKAGE(Protobuf REQUIRED)
INCLUDE_DIRECTORIES(${PROTOBUF_INCLUDE_DIRS})

# OpenCV 3
FIND_PACKAGE(OpenCV 3 REQUIRED)
INCLUDE_DIRECTORIES( ${OpenCV3_INCLUDE_DIRS} )

# VTK 6
FIND_PACKAGE(VTK 6  REQUIRED PATHS ${VTK_FOLDER})
link_directories(${VTK_LIBRARY_DIRS})


# HDF5
FIND_PACKAGE(HDF5 COMPONENTS HL CXX REQUIRED )
INCLUDE_DIRECTORIES(${HDF5_INCLUDE_DIRS})
LINK_DIRECTORIES(${HDF5_LIBRARIES})

# CUDA
find_package(CUDA REQUIRED)
IF(CUDA_FOUND)
MESSAGE(STATUS "Found CUDA")
INCLUDE_DIRECTORIES(${CUDA_INCLUDE_DIRS})
SET(CUDA_LIBRARIES ${CUDA_LIBRARIES} /usr/local/cuda/lib64/libcudart.so)
LINK_DIRECTORIES(${CUDA_LIBRARIES})
MESSAGE(STATUS "Cuda found ${CUDA_INCLUDE_DIRS}, ${CUDA_LIBRARIES} ")
ENDIF(CUDA FOUND)

# Glog
FIND_PACKAGE(GLOG REQUIRED)
INCLUDE_DIRECTORIES(${GLOG_INCLUDE_DIRS})
LINK_DIRECTORIES(${GLOG_LIBRARIES})

# Tensorflow
FIND_PACKAGE(Tensorflow REQUIRED)
INCLUDE_DIRECTORIES(${TensorFlow_INCLUDE_DIRS})
LINK_DIRECTORIES(${TensorFlow_LIBRARIES})

# GLUT
FIND_PACKAGE(GLUT REQUIRED)
INCLUDE_DIRECTORIES(${GLUT_INCLUDE_DIRS})
LINK_DIRECTORIES(${GLUT_LIBRARIES})

# GLEW
FIND_PACKAGE(GLEW REQUIRED)
INCLUDE_DIRECTORIES(${GLEW_INCLUDE_DIRS})
LINK_DIRECTORIES(${GLEW_LIBRARIES})

# BOOST
FIND_PACKAGE(Boost COMPONENTS thread filesystem system REQUIRED)
INCLUDE_DIRECTORIES( ${Boost_INCLUDE_DIR} )

# TBB
FIND_PACKAGE(TBB REQUIRED)
INCLUDE_DIRECTORIES(${TBB_INCLUDE_DIRS})


# Set output folders where to put the executables and libs
SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")
SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")

# Set C++ compile flags
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -std=c++11 -march=native -O3 -Wno-deprecated-declarations --coverage")


# This switch compiles the Address Sanitizer into the binary. For debug purposes....
# Warning: Interfers with GPU code.... CUDA calls are breaking...
SET (ADDRESS_SANITIZER OFF)
IF(ADDRESS_SANITIZER)
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=address -fno-omit-frame-pointer")
SET(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -fsanitize=address")
ENDIF(ADDRESS_SANITIZER)


#Libraries
SET(LIBS_LINKED_LIBS
${VTK_LIBRARIES}
${OpenCV_LIBRARIES}
${OPENGL_LIBRARIES}
${GLEW_LIBRARIES}
${GLUT_LIBRARIES}
${QHULL_LIBRARIES}
${TBB_LIBRARIES}
${GLOG_LIBRARIES}
${TensorFlow_LIBRARIES}
${HDF5_LIBRARIES}
${Boost_LIBRARIES}
${PROTOBUF_LIBRARIES}
${CUDA_LIBRARIES}
)

ADD_LIBRARY(cnnLibs SHARED
)


TARGET_LINK_LIBRARIES(cnnLibs ${LIBS_LINKED_LIBS} cuda )
QT5_USE_MODULES(cnnLibs Widgets OpenGL)

#main
ADD_EXECUTABLE(cnn cnn.cpp)
TARGET_LINK_LIBRARIES(cnn cnnLibs)