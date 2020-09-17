cmake_minimum_required(VERSION 3.14)

project(tiledb_fetch_content)

set(CMAKE_CXX_STANDARD 17)

include(FetchContent)

FetchContent_Declare(
        tiledb_super
        GIT_REPOSITORY https://github.com/TileDB-Inc/TileDB.git
        GIT_TAG "dev"
)

FetchContent_GetProperties(tiledb_super)
if(NOT tiledb_super_POPULATED)
    FetchContent_Populate(tiledb_super)
    # Set various TileDB settings for compilation as you want here.
    set(TILEDB_VERBOSE ON CACHE INTERNAL "Enable verbose mode")
    set(TILEDB_S3 OFF CACHE INTERNAL "Enable S3 support")
    set(TILEDB_STATIC ON CACHE INTERNAL "Enable static library building")
    set(TILEDB_SERIALIZATION OFF CACHE INTERNAL "Enable serialization support")
    set(TILEDB_FORCE_ALL_DEPS OFF CACHE INTERNAL "Make shared library self contained")
    set(TILEDB_TBB OFF CACHE INTERNAL "Disable TBB")
    message(STATUS "tiledb_super_SOURCE_DIR=${tiledb_super_SOURCE_DIR}")
    message(STATUS "tiledb_super_BINARY_DIR=${tiledb_super_BINARY_DIR}")
    add_subdirectory(${tiledb_super_SOURCE_DIR} ${tiledb_super_BINARY_DIR})
    include_directories("${tiledb_super_BINARY_DIR}/tiledb/examples")

    # Set tiledb library, use versions (.2.0 suffix) file instead of main symlink. This simplifies installation from cmake, as cmake install command doesn't handle symlinks.
    SET(TILEDB_LIBRARIES "${tiledb_super_BINARY_DIR}/tiledb/tiledb/${CMAKE_SHARED_LIBRARY_PREFIX}tiledb${CMAKE_SHARED_LIBRARY_SUFFIX}.2.0")
    message(STATUS "TILEDB_LIBRARIES=${TILEDB_LIBRARIES}")
endif()

# Get library directory for multiarch linux distros
include(GNUInstallDirs)

#################################################
# Note: RPATH must be set before add_executable #
#################################################

# We need to set RPATH so if tiledb is built as superbuild we correctly can find it after install
SET(CMAKE_SKIP_BUILD_RPATH FALSE)
SET(CMAKE_SKIP_RPATH FALSE)

# when building, don't use the install RPATH already
# (but later on when installing)
SET(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)
# Use a relative path, this could also be set to the installation path
SET(CMAKE_INSTALL_RPATH "$ORIGIN/../${CMAKE_INSTALL_LIBDIR}")
# Alternative
#SET(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR}")

# add the automatically determined parts of the RPATH
# which point to directories outside the build tree to the install RPATH
SET(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

add_executable(main_src main.cpp)
target_link_libraries(main_src ${TILEDB_LIBRARIES})
add_dependencies(main_src tiledb)

# Set base installation directories
install(
        TARGETS main_src
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

# Install TileDB Library
install(FILES ${TILEDB_LIBRARIES}
        DESTINATION ${CMAKE_INSTALL_LIBDIR})