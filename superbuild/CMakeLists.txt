project(udivone-superbuild)

cmake_minimum_required(VERSION 3.18)

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
  GIT_TAG v2021.1.1
  UPDATE_COMMAND ""
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
    -DTBB_TEST:BOOL=OFF
    -DTBB_EXAMPLES:BOOL=OFF
)

########################################################################
# Intel Embree
########################################################################
ExternalProject_Add(embree
  GIT_REPOSITORY https://github.com/embree/embree.git
  GIT_TAG v3.12.1
  UPDATE_COMMAND ""
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
    -DCMAKE_DEBUG_POSTFIX:STRING=${LIB_DEBUG_SUFFIX}
    -DEMBREE_TASKING_SYSTEM:STRING=TBB
    -DEMBREE_TBB_ROOT:PATH=${CMAKE_INSTALL_PREFIX}
    -DEMBREE_TUTORIALS:BOOL=OFF
  DEPENDS onetbb
)

########################################################################
# Intel rkcommon
########################################################################
ExternalProject_Add(rkcommon
  GIT_REPOSITORY https://github.com/ospray/rkcommon.git
  GIT_TAG v1.5.1
  UPDATE_COMMAND ""
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
    -DBUILD_SHARED_LIBS:BOOL=ON
    -DBUILD_TESTING:BOOL=OFF
    -DTBB_ROOT:PATH=${CMAKE_INSTALL_PREFIX}
    -DINSTALL_DEPS:BOOL=OFF
  DEPENDS onetbb
)

########################################################################
# Intel OpenVKL
########################################################################
ExternalProject_Add(openvkl
  GIT_REPOSITORY https://github.com/openvkl/openvkl.git
  GIT_TAG v0.11.0
  UPDATE_COMMAND ""
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
    -DBUILD_TESTING:BOOL=OFF
    -DBUILD_EXAMPLES:BOOL=OFF
    -DBUILD_BENCHMARKS:BOOL=OFF
  DEPENDS rkcommon
)

########################################################################
# Intel OpenImageDenoise
########################################################################
ExternalProject_Add(oidn
  GIT_REPOSITORY https://github.com/OpenImageDenoise/oidn.git
  GIT_TAG v1.2.4
  UPDATE_COMMAND ""
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
    -DTBB_ROOT:PATH=${CMAKE_INSTALL_PREFIX}
    -DOIDN_APPS:BOOL=OFF
  DEPENDS onetbb
)

########################################################################
# Intel ospray
########################################################################
ExternalProject_Add(ospray
  GIT_REPOSITORY https://github.com/ospray/ospray.git
  GIT_TAG v2.4.0
  GIT_SUBMODULES ""
  UPDATE_COMMAND ""
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
    -DTBB_ROOT:PATH=${CMAKE_INSTALL_PREFIX}
    -Drkcommon_DIR:PATH=${CMAKE_INSTALL_PREFIX}/lib/cmake/rkcommon-1.5.1
    -Dopenvkl_DIR:PATH=${CMAKE_INSTALL_PREFIX}/lib/cmake/openvkl-0.11.0
    -Dembree_DIR:PATH=${CMAKE_INSTALL_PREFIX}
    -DOSPRAY_APPS_BENCHMARK:BOOL=OFF
    -DOSPRAY_APPS_EXAMPLES:BOOL=OFF
    -DOSPRAY_APPS_TESTING:BOOL=OFF
    -DOSPRAY_APPS_TUTORIALS:BOOL=OFF
  DEPENDS onetbb embree rkcommon openvkl oidn
)

########################################################################
# ZLib
########################################################################
ExternalProject_Add(zlib
  GIT_REPOSITORY https://github.com/madler/zlib.git
  GIT_TAG v1.2.11
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
)

########################################################################
# libpng
########################################################################
ExternalProject_Add(libpng
  GIT_REPOSITORY https://github.com/glennrp/libpng.git
  GIT_TAG v1.6.37
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
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

ExternalProject_Add(boost
  GIT_REPOSITORY https://github.com/boostorg/boost.git
  GIT_TAG boost-1.75.0
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
# DCMTK
########################################################################
set(DCMTK_PATCH_COMMAND "")
if (MSVC)
  set(DCMTK_PATCH_COMMAND "git apply ${CMAKE_CURRENT_SOURCE_DIR}/dcmtk.patch")
endif ()

ExternalProject_Add(dcmtk
  GIT_REPOSITORY https://github.com/DCMTK/dcmtk.git
  GIT_TAG DCMTK-3.6.5
  PATCH_COMMAND ${DCMTK_PATCH_COMMAND}
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
    -DCMAKE_DEBUG_POSTFIX:STRING=${LIB_DEBUG_SUFFIX}
    -DCMAKE_FIND_ROOT_PATH:PATH=${CMAKE_INSTALL_PREFIX}/lib
    -DBUILD_APPS:BOOL=ON
    -DBUILD_SHARED_LIBS:BOOL=ON
    -DDCMTK_WITH_ZLIB:BOOL=ON
    -DWITH_ZLIBINC:PATH=${CMAKE_INSTALL_PREFIX}
    -DDCMTK_WITH_PNG:BOOL=ON
    -DWITH_LIBPNGINC:PATH=${CMAKE_INSTALL_PREFIX}
  DEPENDS zlib libpng
)

########################################################################
# VTK
########################################################################
ExternalProject_Add(vtk
  GIT_REPOSITORY https://gitlab.kitware.com/vtk/vtk.git
  GIT_TAG v9.0.1
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
    -DCMAKE_DEBUG_POSTFIX:STRING=${LIB_DEBUG_SUFFIX}
    -DBUILD_SHARED_LIBS:BOOL=ON
    -DVTK_BUILD_DOCUMENTATION:BOOL=OFF
    -DVTK_BUILD_TESTING:BOOL=OFF
    -DVTK_BUILD_EXAMPLES:BOOL=OFF
)

########################################################################
# ITK
########################################################################
ExternalProject_Add(itk
  GIT_REPOSITORY https://github.com/InsightSoftwareConsortium/ITK.git
  GIT_TAG v5.1.2
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
    -DVTK_DIR:PATH=${CMAKE_INSTALL_PREFIX}/lib/cmake/vtk-9.0
  DEPENDS vtk
)

########################################################################
# googletest
########################################################################
ExternalProject_Add(gtest
  GIT_REPOSITORY https://github.com/google/googletest.git
  GIT_TAG release-1.10.0
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
    -DCMAKE_DEBUG_POSTFIX:STRING=${LIB_DEBUG_SUFFIX}
)