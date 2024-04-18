project(udivone-superbuild)

cmake_minimum_required(VERSION 3.19)

cmake_policy(SET CMP0097 NEW)

include(ExternalProject)

set(LIB_DEBUG_SUFFIX "")
if (MSVC)
  set(LIB_DEBUG_SUFFIX "d")
endif ()

########################################################################
# Intel TBB
########################################################################
ExternalProject_Add(onetbb
  GIT_REPOSITORY https://github.com/oneapi-src/oneTBB.git
  GIT_TAG v2021.11.0
  UPDATE_COMMAND ""
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
    -DTBB_TEST:BOOL=OFF
    -DTBB_EXAMPLES:BOOL=OFF
)

########################################################################
# Intel level-zero
########################################################################
ExternalProject_Add(levelzero
  GIT_REPOSITORY https://github.com/oneapi-src/level-zero.git
  GIT_TAG v1.16.14
  UPDATE_COMMAND ""
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
)

########################################################################
# Intel Embree
########################################################################
set(EMBREE_VERSION 4.3.1)

ExternalProject_Add(embree
  GIT_REPOSITORY https://github.com/embree/embree.git
  GIT_TAG v${EMBREE_VERSION}
  UPDATE_COMMAND ""
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
    -DCMAKE_DEBUG_POSTFIX:STRING=${LIB_DEBUG_SUFFIX}
    -DEMBREE_TASKING_SYSTEM:STRING=TBB
    -DEMBREE_TBB_ROOT:PATH=${CMAKE_INSTALL_PREFIX}
    -DEMBREE_ISPC_SUPPORT:BOOL=ON
    -DEMBREE_TUTORIALS:BOOL=OFF
  DEPENDS onetbb
)

########################################################################
# Intel rkcommon
########################################################################
set(RKCOMMON_VERSION 1.13.0)

ExternalProject_Add(rkcommon
  GIT_REPOSITORY https://github.com/ospray/rkcommon.git
  GIT_TAG v${RKCOMMON_VERSION}
  UPDATE_COMMAND ""
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
    -DCMAKE_DEBUG_POSTFIX:STRING=${LIB_DEBUG_SUFFIX}
    -DBUILD_SHARED_LIBS:BOOL=ON
    -DBUILD_TESTING:BOOL=OFF
    -DTBB_ROOT:PATH=${CMAKE_INSTALL_PREFIX}
    -DINSTALL_DEPS:BOOL=OFF
  DEPENDS onetbb
)

########################################################################
# Intel OpenVKL
########################################################################
set(OPENVKL_VERSION 2.0.1)

ExternalProject_Add(openvkl
  GIT_REPOSITORY https://github.com/openvkl/openvkl.git
  GIT_TAG v${OPENVKL_VERSION}
  UPDATE_COMMAND ""
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
    -DCMAKE_DEBUG_POSTFIX:STRING=${LIB_DEBUG_SUFFIX}
    -DBUILD_TESTING:BOOL=OFF
    -DBUILD_EXAMPLES:BOOL=OFF
    -DBUILD_BENCHMARKS:BOOL=OFF
    -DTBB_DIR=${CMAKE_INSTALL_PREFIX}/lib/cmake/TBB
    -Drkcommon_DIR=${CMAKE_INSTALL_PREFIX}/lib/cmake/rkcommon-${RKCOMMON_VERSION}
    -Dembree_DIR=${CMAKE_INSTALL_PREFIX}/lib/cmake/embree-${EMBREE_VERSION}
  DEPENDS onetbb rkcommon embree
)

########################################################################
# Intel OpenImageDenoise
########################################################################
set(OIDN_VERSION 2.2.2)

ExternalProject_Add(oidn
  GIT_REPOSITORY https://github.com/OpenImageDenoise/oidn.git
  GIT_TAG v${OIDN_VERSION}
  UPDATE_COMMAND ""
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
    -DCMAKE_DEBUG_POSTFIX:STRING=${LIB_DEBUG_SUFFIX}
    -DTBB_ROOT:PATH=${CMAKE_INSTALL_PREFIX}
  DEPENDS onetbb
)

########################################################################
# Intel ospray
########################################################################
ExternalProject_Add(ospray
  GIT_REPOSITORY https://github.com/ospray/ospray.git
  GIT_TAG v3.1.0
  GIT_SUBMODULES ""
  UPDATE_COMMAND ""
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
    -DCMAKE_DEBUG_POSTFIX:STRING=${LIB_DEBUG_SUFFIX}
    -DTBB_ROOT:PATH=${CMAKE_INSTALL_PREFIX}/lib/cmake/TBB
    -Drkcommon_DIR:PATH=${CMAKE_INSTALL_PREFIX}/lib/cmake/rkcommon-${RKCOMMON_VERSION}
    -Dopenvkl_DIR:PATH=${CMAKE_INSTALL_PREFIX}/lib/cmake/openvkl-${OPENVKL_VERSION}
    -Dembree_DIR:PATH=${CMAKE_INSTALL_PREFIX}/lib/cmake/embree-${EMBREE_VERSION}
    -DLEVEL_ZERO_ROOT:PATH=${CMAKE_INSTALL_PREFIX}
    -DOSPRAY_ENABLE_APPS_BENCHMARK:BOOL=OFF
    -DOSPRAY_ENABLE_APPS_EXAMPLES:BOOL=OFF
    -DOSPRAY_ENABLE_APPS_TESTING:BOOL=OFF
    -DOSPRAY_ENABLE_APPS_TUTORIALS:BOOL=OFF
    -DOSPRAY_ENABLE_VOLUMES:BOOL=ON
    -DOSPRAY_MODULE_DENOISER:BOOL=ON
    -DOpenImageDenoise_DIR:PATH=${CMAKE_INSTALL_PREFIX}/lib/cmake/OpenImageDenoise-${OIDN_VERSION}
  DEPENDS onetbb levelzero embree rkcommon openvkl oidn
)

########################################################################
# ZLib
########################################################################
ExternalProject_Add(zlib
  GIT_REPOSITORY https://github.com/madler/zlib.git
  GIT_TAG v1.3.1
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
)

########################################################################
# libpng
########################################################################
ExternalProject_Add(libpng
  GIT_REPOSITORY https://github.com/glennrp/libpng.git
  GIT_TAG v1.6.43
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
    -DZLIB_ROOT:PATH=${CMAKE_INSTALL_PREFIX}
  DEPENDS zlib
)

########################################################################
# openjpeg
########################################################################
ExternalProject_Add(openjpeg
  GIT_REPOSITORY https://github.com/uclouvain/openjpeg.git
  GIT_TAG v2.5.2
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
    -DCMAKE_DEBUG_POSTFIX:STRING=${LIB_DEBUG_SUFFIX}
    -DBUILD_TESTING:BOOL=OFF
    -DBUILD_SHARED_LIBS:BOOL=ON
    -DBUILD_STATIC_LIBS:BOOL=OFF
    -DZLIB_ROOT:PATH=${CMAKE_INSTALL_PREFIX}
  DEPENDS zlib
)

########################################################################
# Boost
########################################################################
if (WIN32)
  set(BOOST_BOOTSTRAP_CMD ${CMAKE_CURRENT_BINARY_DIR}/boost-prefix/src/boost/bootstrap.bat)
  set(BOOST_BUILD_CMD ${CMAKE_CURRENT_BINARY_DIR}/boost-prefix/src/boost/b2.exe)
elseif (UNIX)
  set(BOOST_BOOTSTRAP_CMD ${CMAKE_CURRENT_BINARY_DIR}/boost-prefix/src/boost/bootstrap.sh)
  set(BOOST_BUILD_CMD ${CMAKE_CURRENT_BINARY_DIR}/boost-prefix/src/boost/b2)
endif ()

set(BOOST_VERSION 1.84.0)

ExternalProject_Add(boost
  GIT_REPOSITORY https://github.com/boostorg/boost.git
  GIT_TAG boost-${BOOST_VERSION}
  CONFIGURE_COMMAND ${BOOST_BOOTSTRAP_CMD}
  BUILD_COMMAND
    ${BOOST_BUILD_CMD}
    --prefix=${CMAKE_INSTALL_PREFIX}
    -sZLIB_LIBRARY_PATH=${CMAKE_INSTALL_PREFIX}/lib
    --without-python
    address-model=64
    variant=release
    link=shared
    runtime-link=shared
    threading=multi
    install
  BUILD_IN_SOURCE 1
  INSTALL_COMMAND ""
)

########################################################################
# C-Blosc
########################################################################
ExternalProject_Add(blosc
  GIT_REPOSITORY https://github.com/Blosc/c-blosc.git
  GIT_TAG v1.21.5
  UPDATE_COMMAND ""
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
    -DCMAKE_DEBUG_POSTFIX:STRING=${LIB_DEBUG_SUFFIX}
    -DBUILD_BENCHMARKS=OFF
    -DBUILD_FUZZERS:BOOL=OFF
    -DBUILD_SHARED:BOOL=ON
    -DBUILD_STATIC:BOOL=OFF
    -DBUILD_TESTS:BOOL=OFF
    -DPREFER_EXTERNAL_ZLIB:BOOL=ON
    -DZLIB_ROOT:PATH=${CMAKE_INSTALL_PREFIX}
  DEPENDS zlib
)

########################################################################
# OpenVDB
########################################################################
ExternalProject_Add(openvdb
  GIT_REPOSITORY https://github.com/AcademySoftwareFoundation/openvdb.git
  GIT_TAG v11.0.0
  UPDATE_COMMAND ""
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
    -DCMAKE_DEBUG_POSTFIX:STRING=${LIB_DEBUG_SUFFIX}
    -DBoost_DIR=${CMAKE_INSTALL_PREFIX}/lib/cmake/Boost-${BOOST_VERSION}
    -DUSE_TBB:BOOL=ON
    -DTBB_ROOT:PATH=${CMAKE_INSTALL_PREFIX}
    -DUSE_BLOSC:BOOL=ON
    -DBlosc_ROOT:PATH=${CMAKE_INSTALL_PREFIX}
    -DUSE_ZLIB:BOOL=ON
    -DZLIB_ROOT:PATH=${CMAKE_INSTALL_PREFIX}
    -DUSE_PNG:BOOL=ON
  DEPENDS boost blosc onetbb zlib libpng
)

########################################################################
# DCMTK
########################################################################
set(DCMTK_PATCH_COMMAND "")

ExternalProject_Add(dcmtk
  GIT_REPOSITORY https://github.com/DCMTK/dcmtk.git
  GIT_TAG DCMTK-3.6.8
  PATCH_COMMAND ${DCMTK_PATCH_COMMAND}
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
    -DCMAKE_DEBUG_POSTFIX:STRING=${LIB_DEBUG_SUFFIX}
    -DBUILD_APPS:BOOL=ON
    -DBUILD_SHARED_LIBS:BOOL=ON
    -DDCMTK_USE_FIND_PACKAGE:BOOL=ON
    -DDCMTK_WITH_ZLIB:BOOL=ON
    -DZLIB_ROOT:PATH=${CMAKE_INSTALL_PREFIX}
    -DDCMTK_WITH_PNG:BOOL=ON
    -DPNG_PNG_INCLUDE_DIR:PATH=${CMAKE_INSTALL_PREFIX}/include
    -DDCMTK_WITH_OPENJPEG:BOOL=ON
    -DJPEG_INCLUDE_DIR:PATH=${CMAKE_INSTALL_PREFIX}/include
  DEPENDS zlib libpng openjpeg
)

########################################################################
# VTK
########################################################################
set(VTK_MAJOR_VERSION 9)
set(VTK_MINOR_VERSION 3)
set(VTK_PATCH_LEVEL 0)

ExternalProject_Add(vtk
  GIT_REPOSITORY https://gitlab.kitware.com/vtk/vtk.git
  GIT_TAG v${VTK_MAJOR_VERSION}.${VTK_MINOR_VERSION}.${VTK_PATCH_LEVEL}
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
    -DCMAKE_DEBUG_POSTFIX:STRING=${LIB_DEBUG_SUFFIX}
    -DCMAKE_FIND_ROOT_PATH:PATH=${CMAKE_INSTALL_PREFIX}/lib
    -DBUILD_SHARED_LIBS:BOOL=ON
    -DVTK_BUILD_DOCUMENTATION:BOOL=OFF
    -DVTK_BUILD_TESTING:BOOL=OFF
    -DVTK_BUILD_EXAMPLES:BOOL=OFF
    -DVTK_MODULE_USE_EXTERNAL_VTK_zlib:BOOL=ON
    -DZLIB_ROOT:PATH=${CMAKE_INSTALL_PREFIX}
  DEPENDS zlib
)

########################################################################
# ITK
########################################################################
ExternalProject_Add(itk
  GIT_REPOSITORY https://github.com/InsightSoftwareConsortium/ITK.git
  GIT_TAG v5.3.0
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
    -DCMAKE_DEBUG_POSTFIX:STRING=${LIB_DEBUG_SUFFIX}
    -DCMAKE_FIND_ROOT_PATH:PATH=${CMAKE_INSTALL_PREFIX}/lib
    -DBUILD_SHARED_LIBS:BOOL=ON
    -DBUILD_TESTING:BOOL=OFF
    -DBUILD_EXAMPLES:BOOL=OFF
    -DBUILD_TESTING:BOOL=OFF
    -DITK_USE_SYSTEM_ZLIB:BOOL=ON
    -DZLIB_ROOT:PATH=${CMAKE_INSTALL_PREFIX}
    -DITK_USE_SYSTEM_PNG:BOOL=ON
    -DPNG_PNG_INCLUDE_DIR:PATH=${CMAKE_INSTALL_PREFIX}/include
    -DModule_ITKTBB:BOOL=ON
    -DTBB_DIR:PATH=${CMAKE_INSTALL_PREFIX}/lib/cmake/TBB
    -DModule_ITKVtkGlue:BOOL=ON
    -DVTK_DIR:PATH=${CMAKE_INSTALL_PREFIX}/lib/cmake/vtk-${VTK_MAJOR_VERSION}.${VTK_MINOR_VERSION}
  DEPENDS onetbb vtk zlib libpng
)

########################################################################
# googletest
########################################################################
ExternalProject_Add(gtest
  GIT_REPOSITORY https://github.com/google/googletest.git
  GIT_TAG v1.14.0
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
    -DCMAKE_DEBUG_POSTFIX:STRING=${LIB_DEBUG_SUFFIX}
)

########################################################################
# Eigen
########################################################################
ExternalProject_Add(eigen
  GIT_REPOSITORY https://gitlab.com/libeigen/eigen.git
  GIT_TAG 3.4.0
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
)
