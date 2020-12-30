project(udivone-superbuild)

cmake_minimum_required(VERSION 3.18)

find_package(git REQUIRED)

include(ExternalProject)

########################################################################
# Intel TBB
########################################################################
ExternalProject_Add(oneTBB
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
ExternalProject_Add(Embree
  GIT_REPOSITORY https://github.com/embree/embree.git
  GIT_TAG v3.12.1
  UPDATE_COMMAND ""
  INSTALL_COMMAND ""
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
    -DEMBREE_TASKING_SYSTEM:STRING=TBB
    -DEMBREE_TBB_ROOT:PATH=${CMAKE_INSTALL_PREFIX}
  DEPENDS oneTBB
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
    -DPNG_STATIC:BOOL=OFF
    -DPNG_TESTS:BOOL=OFF
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
  set(BOOST_BUILD_CMD ${CMAKE_CURRENT_BINARY_DIR}/boost-prefix/srd/boost/b2)
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
    variant=release,debug
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
ExternalProject_Add(dcmtk
  GIT_REPOSITORY https://github.com/DCMTK/dcmtk.git
  GIT_TAG DCMTK-3.6.5
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
    -DZLIB_ROOT:PATH=${CMAKE_INSTALL_PREFIX}
    -DDCMTK_WITH_DOXYGEN:BOOL=OFF
)

########################################################################
# googletest
########################################################################
ExternalProject_Add(gtest
  GIT_REPOSITORY https://github.com/google/googletest.git
  GIT_TAG release-1.10.0
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_INSTALL_PREFIX}
)