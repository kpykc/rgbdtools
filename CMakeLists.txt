cmake_minimum_required(VERSION 2.8)

project(rgbdtools)
set(CMAKE_CXX_FLAGS_RELEASE "-O3")
set(CMAKE_BUILD_TYPE Release)
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
set(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake-modules/")

include_directories(${PROJECT_SOURCE_DIR}/include)

####################################################
# Dependencies:
find_package(PkgConfig)

# Dependencies - Boost:
find_package(Boost REQUIRED)
include_directories(${Boost_INCLUDE_DIRS})

# Dependencies - OpenCV:
FIND_PACKAGE( OpenCV REQUIRED )
include_directories(${OpenCV_INCLUDE_DIRS})
link_directories(${OpenCV_LIBRARY_DIRS})

# Dependencies - PCL:
find_package(PCL REQUIRED)
include_directories(${PCL_INCLUDE_DIRS})
link_directories(${PCL_LIBRARY_DIRS})

# Dependencies - Suitesparse:
FIND_PACKAGE(SuiteSparse REQUIRED)

if (${CMAKE_SYSTEM_NAME} MATCHES "Darwin") # small hack for osx/yosemite
  set(SUITESPARSE_LIBRARY_DIRS /usr/local/Cellar/suite-sparse/4.2.1/lib/)
  set(SUITESPARSE_LIBRARIES
    cxsparse cxsparse cholmod suitesparseconfig
  )
endif()

include_directories(${SUITESPARSE_INCLUDE_DIRS})
link_directories(${SUITESPARSE_LIBRARY_DIRS})

# Dependencies - G2O:
# Use script to find g2o. If it's not installed, set G2O_INCLUDE_DIRS manually 
# using -DG2O_INCLUDE_DIRS.
find_package(G2O REQUIRED)

if (${CMAKE_SYSTEM_NAME} MATCHES "Darwin") # small hack for osx/yosemite
  set(G2O_LIBRARY_DIRS /usr/local/Cellar/g2o/13./lib/)
  # set(G2O_LIBRARY_DIRS /Users/kp/kpykc/devel/lib)
  set(G2O_LIBRARIES
    g2o_cli g2o_ext_freeglut_minimal g2o_simulator g2o_solver_slam2d_linear g2o_types_icp g2o_types_slam2d g2o_core g2o_interface g2o_solver_csparse g2o_solver_structure_only g2o_types_sba g2o_types_slam3d g2o_csparse_extension g2o_opengl_helper g2o_solver_dense g2o_stuff g2o_types_sclam2d  g2o_parser g2o_solver_pcg g2o_types_data g2o_types_sim3
  )
endif()

include_directories(${G2O_INCLUDE_DIR})
link_directories(${G2O_LIBRARY_DIRS})

####################################################
# Build rgbdtools library.

add_library (rgbdtools SHARED
  src/rgbd_frame.cpp
  src/rgbd_keyframe.cpp
  src/rgbd_util.cpp
  src/map_util.cpp
  src/features/feature_detector.cpp
  src/features/orb_detector.cpp
  src/features/surf_detector.cpp
  src/features/gft_detector.cpp
  src/features/star_detector.cpp
  src/registration/motion_estimation.cpp
  src/registration/motion_estimation_icp_prob_model.cpp
  src/registration/motion_estimation_pairwise_ransac.cpp
  src/graph/keyframe_graph_detector.cpp
  src/graph/keyframe_graph_solver.cpp
  src/graph/keyframe_graph_solver_g2o.cpp)

target_link_libraries (rgbdtools
  ${Boost_LIBRARIES}
  ${OpenCV_LIBRARIES}
  ${PCL_LIBRARIES}
  ${SUITESPARSE_LIBRARIES}
  ${G2O_LIBRARIES}
  # g2o_cli g2o_ext_freeglut_minimal g2o_simulator g2o_solver_slam2d_linear g2o_types_icp g2o_types_slam2d g2o_core g2o_interface g2o_solver_csparse g2o_solver_structure_only g2o_types_sba g2o_types_slam3d g2o_csparse_extension g2o_opengl_helper g2o_solver_dense g2o_stuff g2o_types_sclam2d  g2o_parser g2o_solver_pcg g2o_types_data g2o_types_sim3 
  # cxsparse cxsparse cholmod suitesparseconfig
)
  
install(TARGETS rgbdtools DESTINATION lib)
install(DIRECTORY include/rgbdtools DESTINATION include)

####################################################
# Build global cloud align applications.

add_executable(global_cloud_align apps/global_cloud_align.cpp)
target_link_libraries (global_cloud_align rgbdtools) 
install(TARGETS global_cloud_align DESTINATION bin)

####################################################
# Build keyframe associations applications.

add_executable(keyframe_associations apps/keyframe_associations.cpp)
target_link_libraries (keyframe_associations rgbdtools)
install(TARGETS keyframe_associations DESTINATION bin)

