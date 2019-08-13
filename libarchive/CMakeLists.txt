project(libarchive)

cmake_minimum_required(VERSION 2.4.0)

if(COMMAND cmake_policy)
	cmake_policy(SET CMP0003 NEW)
endif(COMMAND cmake_policy)

include(darling_exe)
add_definitions(-DTARGET_OS_MAC=1)
add_definitions(-D__APPLE__ -D__DYNAMIC__)
add_definitions(-DHAVE_CONFIG_H)

add_definitions(
	-Wno-macro-redefined
	-Wno-absolute-value
)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D_POSIX_C_SOURCE -nostdinc -fPIC")

include_directories(BEFORE ${CMAKE_CURRENT_SOURCE_DIR}/..)
include_directories(BEFORE ${CMAKE_CURRENT_SOURCE_DIR}/libarchive)
include_directories(BEFORE ${CMAKE_CURRENT_SOURCE_DIR}/libarchive_fe)
include_directories(${CMAKE_SOURCE_DIR}/src/libc/include/FreeBSD)
include_directories(${CMAKE_SOURCE_DIR}/src/libc/include/NetBSD)
include_directories(${CMAKE_SOURCE_DIR}/src/libc/gen)
include_directories(${CMAKE_SOURCE_DIR}/src/libc/darwin)
include_directories(${CMAKE_SOURCE_DIR}/src/libc/stdtime/FreeBSD)
include_directories(${CMAKE_SOURCE_DIR}/src/launchd/liblaunch)
include_directories(${CMAKE_SOURCE_DIR}/src/external/libdispatch)
include_directories(${CMAKE_SOURCE_DIR}/src/external/zlib)
include_directories(${CMAKE_SOURCE_DIR}/src/external/bzip2)
include_directories(${CMAKE_SOURCE_DIR}/src/copyfile)
include_directories(${CMAKE_SOURCE_DIR}/src/external/liblzma/src/liblzma/api/)
include_directories(${CMAKE_SOURCE_DIR}/src/external/libxml2/include/)
include_directories(${CMAKE_BINARY_DIR}/src/external/libxml2/include/)
include_directories(${CMAKE_SOURCE_DIR}/src/external/icu/icuSources/common/)
include_directories(${CMAKE_SOURCE_DIR}/src/CommonCrypto)

set(libarchive_sources
	libarchive/archive_check_magic.c
	libarchive/archive_entry.c
	libarchive/archive_entry_copy_bhfi.c
	libarchive/archive_entry_copy_stat.c
	libarchive/archive_entry_link_resolver.c
	libarchive/archive_entry_stat.c
	libarchive/archive_entry_strmode.c
	libarchive/archive_entry_xattr.c
	libarchive/archive_read.c
	libarchive/archive_read_data_into_fd.c
	libarchive/archive_read_disk.c
	libarchive/archive_read_disk_entry_from_file.c
	libarchive/archive_read_disk_set_standard_lookup.c
	libarchive/archive_read_extract.c
	libarchive/archive_read_open_fd.c
	libarchive/archive_read_open_file.c
	libarchive/archive_read_open_filename.c
	libarchive/archive_read_open_memory.c
	libarchive/archive_read_support_compression_all.c
	libarchive/archive_read_support_compression_bzip2.c
	libarchive/archive_read_support_compression_compress.c
	libarchive/archive_read_support_compression_gzip.c
	libarchive/archive_read_support_compression_none.c
	libarchive/archive_read_support_compression_program.c
	libarchive/archive_read_support_compression_rpm.c
	libarchive/archive_read_support_compression_uu.c
	libarchive/archive_read_support_compression_xz.c
	libarchive/archive_read_support_format_all.c
	libarchive/archive_read_support_format_ar.c
	libarchive/archive_read_support_format_cpio.c
	libarchive/archive_read_support_format_empty.c
	libarchive/archive_read_support_format_iso9660.c
	libarchive/archive_read_support_format_mtree.c
	libarchive/archive_read_support_format_raw.c
	libarchive/archive_read_support_format_tar.c
	libarchive/archive_read_support_format_xar.c
	libarchive/archive_read_support_format_zip.c
	libarchive/archive_string.c
	libarchive/archive_string_sprintf.c
	libarchive/archive_util.c
	libarchive/archive_virtual.c
	libarchive/archive_write.c
	libarchive/archive_write_disk.c
	libarchive/archive_write_disk_set_standard_lookup.c
	libarchive/archive_write_open_fd.c
	libarchive/archive_write_open_file.c
	libarchive/archive_write_open_filename.c
	libarchive/archive_write_open_memory.c
	libarchive/archive_write_set_compression_bzip2.c
	libarchive/archive_write_set_compression_compress.c
	libarchive/archive_write_set_compression_gzip.c
	libarchive/archive_write_set_compression_none.c
	libarchive/archive_write_set_compression_program.c
	libarchive/archive_write_set_compression_xz.c
	libarchive/archive_write_set_format_ar.c
	libarchive/archive_write_set_format_by_name.c
	libarchive/archive_write_set_format.c
	libarchive/archive_write_set_format_cpio.c
	libarchive/archive_write_set_format_cpio_newc.c
	libarchive/archive_write_set_format_mtree.c
	libarchive/archive_write_set_format_pax.c
	libarchive/archive_write_set_format_shar.c
	libarchive/archive_write_set_format_ustar.c
	libarchive/archive_write_set_format_zip.c
	libarchive/filter_fork.c
)

set(DYLIB_INSTALL_NAME "/usr/lib/libarchive.2.dylib")
set(DYLIB_COMPAT_VERSION "9.0.0")
set(DYLIB_CURRENT_VERSION "9.2.0")

add_darling_library(archive SHARED ${libarchive_sources})
make_fat(archive)
target_link_libraries(archive system lzma xml2 z bz2)
set_target_properties(archive PROPERTIES OUTPUT_NAME "archive.2")

InstallSymlink("libarchive.2.dylib" "${CMAKE_INSTALL_PREFIX}/libexec/darling/usr/lib/libarchive.dylib")

set(libarchive_fe_sources
	libarchive_fe/err.c
	libarchive_fe/line_reader.c
	libarchive_fe/matching.c
	libarchive_fe/pathmatch.c
)

add_darling_object_library(archive_fe ${libarchive_fe_sources})
install(TARGETS archive
	DESTINATION libexec/darling/usr/lib)

if (NOT DARLING_NO_EXECUTABLES)
	add_darling_executable(cpio cpio/cmdline.c cpio/cpio.c $<TARGET_OBJECTS:archive_fe>)
	target_link_libraries(cpio archive)

	set(bsdtar_sources
		tar/bsdtar.c
		tar/cmdline.c
		tar/getdate.c
		tar/read.c
		tar/subst.c
		tar/tree.c
		tar/util.c
		tar/write.c
	)
	add_darling_executable(bsdtar ${bsdtar_sources} $<TARGET_OBJECTS:archive_fe>)
	target_link_libraries(bsdtar archive)

	install(TARGETS bsdtar cpio
		DESTINATION libexec/darling/usr/bin)

	install(FILES symlinks/tar
		DESTINATION libexec/darling/usr/bin)
endif (NOT DARLING_NO_EXECUTABLES)

set(man1
	contrib/shar/shar.1
	doc/man/bsdcpio.1
	symlinks/cpio.1
	doc/man/bsdtar.1
	symlinks/tar.1
)

install(FILES ${man1} DESTINATION libexec/darling/usr/share/man/man1)

set(man3
	doc/man/archive_entry.3
	doc/man/archive_read.3
	doc/man/archive_read_disk.3
	doc/man/archive_util.3
	doc/man/archive_write.3
	doc/man/archive_write_disk.3
	doc/man/libarchive.3
	doc/man/libarchive_internals.3
)

install(FILES ${man3} DESTINATION libexec/darling/usr/share/man/man3)

set(man5
	doc/man/cpio.5
	doc/man/libarchive-formats.5
	doc/man/mtree.5
	doc/man/tar.5
)

install(FILES ${man5} DESTINATION libexec/darling/usr/share/man/man5)
