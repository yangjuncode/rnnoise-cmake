# rnnoise-cmake

use rnnoise as a static lib in cmake

put downloaded rnnoise as a sub dir in your cmake project, then add below to your cmakelists.txt

```cmake
# ----------------------------------------------------------------------------
# RNNoise via ExternalProject (local source under rnnoise/)
# ----------------------------------------------------------------------------
include(ExternalProject)

set(RNNOISE_INSTALL_DIR ${CMAKE_BINARY_DIR}/rnnoise)

# Platform-aware RNNoise optimization flags
set(RNNOISE_CONFIGURE_FLAGS --disable-shared --enable-static --disable-examples --disable-doc)
set(RNNOISE_CFLAGS "-fPIC")
if (CMAKE_SYSTEM_PROCESSOR MATCHES "x86_64|AMD64|i.86")
    list(APPEND RNNOISE_CONFIGURE_FLAGS --enable-x86-rtcd)
    option(RNNOISE_ENABLE_AVX2 "Enable AVX2 (-mavx2) for RNNoise build (x86 only)" OFF)
    if (RNNOISE_ENABLE_AVX2)
        set(RNNOISE_CFLAGS "-O3 -mavx2")
    endif ()
endif ()
string(JOIN " " RNNOISE_CONFIGURE_FLAGS_STR ${RNNOISE_CONFIGURE_FLAGS})

ExternalProject_Add(rnnoise_project
        SOURCE_DIR ${CMAKE_CURRENT_LIST_DIR}/rnnoise
        DOWNLOAD_COMMAND ""

        # Autotools configure (out-of-source) with platform-aware flags
        CONFIGURE_COMMAND sh -c "cd <SOURCE_DIR> && ./autogen.sh && cd <BINARY_DIR> && env CFLAGS='${RNNOISE_CFLAGS}' <SOURCE_DIR>/configure  --prefix=<INSTALL_DIR> ${RNNOISE_CONFIGURE_FLAGS_STR}"

        # Build / Install
        BUILD_COMMAND make
        INSTALL_COMMAND make install

        # Inform CMake/Ninja that this file will be produced by the ExternalProject
        INSTALL_BYPRODUCTS ${RNNOISE_INSTALL_DIR}/lib/librnnoise.a

        INSTALL_DIR ${RNNOISE_INSTALL_DIR}
)

# Create IMPORTED static library target for rnnoise
file(MAKE_DIRECTORY ${RNNOISE_INSTALL_DIR}/include)
file(MAKE_DIRECTORY ${RNNOISE_INSTALL_DIR}/lib)
add_library(rnnoise STATIC IMPORTED)
set_property(TARGET rnnoise PROPERTY IMPORTED_LOCATION ${RNNOISE_INSTALL_DIR}/lib/librnnoise.a)
set_property(TARGET rnnoise PROPERTY INTERFACE_INCLUDE_DIRECTORIES ${RNNOISE_INSTALL_DIR}/include)
if (UNIX AND NOT APPLE)
    set_property(TARGET rnnoise PROPERTY INTERFACE_LINK_LIBRARIES m)
endif ()

# Ensure rnnoise is built before being linked
add_dependencies(rnnoise rnnoise_project)

```
