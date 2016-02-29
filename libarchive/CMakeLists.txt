project(libarchive)

cmake_minimum_required(VERSION 2.4.0)

if(COMMAND cmake_policy)
	cmake_policy(SET CMP0003 NEW)
endif(COMMAND cmake_policy)

include(darling_exe)
add_definitions(-DTARGET_OS_MAC=1)
add_definitions(-D__APPLE__ -D__DYNAMIC__)
add_definitions(-DHAVE_CONFIG_H)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D_POSIX_C_SOURCE -nostdinc -fPIC")
set(CMAKE_EXE_LINKER_FLAGS "-nodefaultlibs -nostdlib -Wl,--version-script=${DARLING_TOP_DIRECTORY}/darwin.map")

include_directories(BEFORE ${CMAKE_CURRENT_SOURCE_DIR}/..)
include_directories(BEFORE ${CMAKE_CURRENT_SOURCE_DIR}/libarchive)
include_directories(BEFORE ${CMAKE_CURRENT_SOURCE_DIR}/libarchive_fe)
include_directories(${DARLING_TOP_DIRECTORY}/src/libc/include/FreeBSD)
include_directories(${DARLING_TOP_DIRECTORY}/src/libc/include/NetBSD)
include_directories(${DARLING_TOP_DIRECTORY}/src/libc/gen)
include_directories(${DARLING_TOP_DIRECTORY}/src/libc/darwin)
include_directories(${DARLING_TOP_DIRECTORY}/src/libc/stdtime/FreeBSD)
include_directories(${DARLING_TOP_DIRECTORY}/src/launchd/liblaunch)
include_directories(${DARLING_TOP_DIRECTORY}/src/external/libdispatch)
include_directories(${DARLING_TOP_DIRECTORY}/src/external/zlib)
include_directories(${DARLING_TOP_DIRECTORY}/src/external/bzip2)
include_directories(${DARLING_TOP_DIRECTORY}/src/copyfile)
include_directories(${DARLING_TOP_DIRECTORY}/src/external/liblzma/src/liblzma/api/)
include_directories(${DARLING_TOP_DIRECTORY}/src/external/libxml2/include/)
include_directories(${CMAKE_BINARY_DIR}/src/external/libxml2/include/)
include_directories(${DARLING_TOP_DIRECTORY}/src/external/icu/icuSources/common/)
include_directories(${DARLING_TOP_DIRECTORY}/src/CommonCrypto)

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

add_library(archive SHARED ${libarchive_sources})
target_link_libraries(archive system lzma xml2 CommonCrypto)

set(libarchive_fe_sources
	libarchive_fe/err.c
	libarchive_fe/line_reader.c
	libarchive_fe/matching.c
	libarchive_fe/pathmatch.c
)

add_library(archive_fe OBJECT ${libarchive_fe_sources})
install(TARGETS archive
	DESTINATION ${CMAKE_INSTALL_LIBDIR}/darling)

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

