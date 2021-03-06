# ----------------------------------------------------------------------------
#  CMake file for libtiff. See root CMakeLists.txt
#
# ----------------------------------------------------------------------------
include(CheckFunctionExists)
include(CheckIncludeFile)
include(CheckLibraryExists)
include(TestBigEndian)

check_include_file(assert.h HAVE_ASSERT_H)
check_include_file(fcntl.h HAVE_FCNTL_H)
check_include_file(io.h HAVE_IO_H)
check_function_exists(jbg_newlen HAVE_JBG_NEWLEN)
check_function_exists(mmap HAVE_MMAP)
check_include_file(search.h HAVE_SEARCH_H)
check_include_file(string.h HAVE_STRING_H)
check_include_file(sys/types.h HAVE_SYS_TYPES_H)
check_include_file(unistd.h HAVE_UNISTD_H)

test_big_endian(WORDS_BIGENDIAN)

if(WIN32 AND NOT HAVE_WINRT)
  set(USE_WIN32_FILEIO 1)
endif()

configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/tif_config.h.cmakein"
    "${CMAKE_CURRENT_BINARY_DIR}/tif_config.h"
    @ONLY
)

hunter_add_package(ZLIB)
find_package(ZLIB REQUIRED CONFIG)

include_directories(
    "${CMAKE_CURRENT_SOURCE_DIR}"
    "${CMAKE_CURRENT_BINARY_DIR}"
)

set(
    lib_srcs
    tif_aux.c
    tif_close.c
    tif_codec.c
    tif_color.c
    tif_compress.c
    tif_dir.c
    tif_dirinfo.c
    tif_dirread.c
    tif_dirwrite.c
    tif_dumpmode.c
    tif_error.c
    tif_extension.c
    tif_fax3.c
    tif_fax3sm.c
    tif_flush.c
    tif_getimage.c
    tif_jbig.c
    tif_jpeg_12.c
    tif_jpeg.c
    tif_luv.c
    tif_lzma.c
    tif_lzw.c
    tif_next.c
    tif_ojpeg.c
    tif_open.c
    tif_packbits.c
    tif_pixarlog.c
    tif_predict.c
    tif_print.c
    tif_read.c
    tif_strip.c
    tif_swab.c
    tif_thunder.c
    tif_tile.c
    tif_version.c
    tif_warning.c
    tif_write.c
    tif_zip.c
    tif_stream.cxx
    t4.h
    tif_dir.h
    tif_fax3.h
    tiff.h
    tiffio.h
    tiffiop.h
    tiffvers.h
    tif_predict.h
    uvcode.h
    tiffio.hxx
    "${CMAKE_CURRENT_BINARY_DIR}/tif_config.h"
)

if(WIN32 AND NOT HAVE_WINRT)
  list(APPEND lib_srcs tif_win32.c)
else()
  list(APPEND lib_srcs tif_unix.c)
endif()

add_library(libtiff ${lib_srcs})
target_link_libraries(libtiff PUBLIC ZLIB::zlib)

# Create separate static library if BUILD_SHARED_LIBS is on
# as target libtiff will be converted to shared library
if(BUILD_SHARED_LIBS)
  add_library(libtiff_static STATIC ${lib_srcs})
  target_link_libraries(libtiff_static PUBLIC ZLIB::zlib)
  set_target_properties(libtiff_static PROPERTIES OUTPUT_NAME libtiff)
endif()

# Some platforms (e.g. Linux) need separate math library
check_library_exists(m pow "" LIB_M_REQUIRED)
if(LIB_M_REQUIRED)
  target_link_libraries(libtiff PRIVATE m)
  if(BUILD_SHARED_LIBS)
    target_link_libraries(libtiff_static PRIVATE m)
  endif()
endif()

if(UNIX AND CMAKE_COMPILER_IS_GNUCXX)
  set_target_properties(libtiff PROPERTIES POSITION_INDEPENDENT_CODE TRUE)
  if(BUILD_SHARED_LIBS)
    set_target_properties(libtiff_static PROPERTIES POSITION_INDEPENDENT_CODE TRUE)
  endif()
endif()

if(ENABLE_SOLUTION_FOLDERS)
  set_target_properties(libtiff PROPERTIES FOLDER "3rdparty")
  if(BUILD_SHARED_LIBS)
    set_target_properties(libtiff_static PROPERTIES FOLDER "3rdparty")
  endif()
endif()

####
# Installation (https://github.com/forexample/package-example)

set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(include_install_dir "include")

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)

# Note: PROJECT_VERSION is used as a VERSION
write_basic_package_version_file(
    "${version_config}" COMPATIBILITY SameMajorVersion
)

# Note: variable 'targets_export_name' used
configure_file("cmake/Config.cmake.in" "${project_config}" @ONLY)

install(
    TARGETS libtiff
    EXPORT "${targets_export_name}"
    LIBRARY DESTINATION "lib"
    ARCHIVE DESTINATION "lib"
    RUNTIME DESTINATION "bin"
    INCLUDES DESTINATION "${include_install_dir}"
)

if(BUILD_SHARED_LIBS)
  install(
    TARGETS libtiff_static
    EXPORT "${targets_export_name}"
    LIBRARY DESTINATION "lib"
    ARCHIVE DESTINATION "lib"
    RUNTIME DESTINATION "bin"
    INCLUDES DESTINATION "${include_install_dir}"
  )
endif()

install(
    FILES "${CMAKE_CURRENT_BINARY_DIR}/tif_config.h" tiffio.h tiff.h tiffvers.h
    DESTINATION "${include_install_dir}"
)

install(
    FILES "${project_config}" "${version_config}"
    DESTINATION "${config_install_dir}"
)

install(
    EXPORT "${targets_export_name}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}"
)
