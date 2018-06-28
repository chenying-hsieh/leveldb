set(CMAKE_LEGACY_CYGWIN_WIN32 0)
cmake_minimum_required(VERSION 2.8)

set(CMAKE_SYSTEM_NAME "Linux")
PROJECT(leveldb)

include(CheckIncludeFile)
include(FindBoost) # for Windows...

# version

#ENABLE_TESTING()

# Set a default build type for single-configuration
# CMake generators if no build type is set.
if(NOT CMAKE_CONFIGURATION_TYPES AND NOT CMAKE_BUILD_TYPE)
	set(CMAKE_BUILD_TYPE DEBUG)
endif()

include_directories(${CMAKE_CURRENT_SOURCE_DIR} include)

add_definitions(-DLEVELDB_PLATFORM_POSIX -DOS_LINUX)

set(PORT_SRCS port/port_posix.cc util/env_posix.cc)
set(CIF_SRCS db/c.cc)

# FIXME: check cstdatomic here..

set(SRCS
    db/builder.cc
    db/c.cc
    db/db_bench.cc
    db/dbformat.cc
    db/db_impl.cc
    db/db_iter.cc
    db/dumpfile.cc
    db/filename.cc
    db/leveldbutil.cc
    db/log_reader.cc
    db/log_writer.cc
    db/memtable.cc
    db/repair.cc
    db/table_cache.cc
    db/version_edit.cc
    db/version_set.cc
    db/write_batch.cc
    table/block_builder.cc
    table/block.cc
    table/filter_block.cc
    table/format.cc
    table/iterator.cc
    table/merger.cc
    table/table_builder.cc
    table/table.cc
    table/two_level_iterator.cc
    util/arena.cc
    util/bloom.cc
    util/cache.cc
    util/coding.cc
    util/comparator.cc
    util/crc32c.cc
    util/env.cc
    util/env_posix.cc
    util/filter_policy.cc
    util/hash.cc
    util/histogram.cc
    util/logging.cc
    util/options.cc
    util/status.cc
    ${PORT_SRCS}
    ${CIF_SRCS})

set(TESTHARNESS_SRCS
    util/testutil.cc
    util/testharness.cc)

include_directories(${CMAKE_CURRENT_BINARY_DIR})

set(PUBLIC_INCLUDE_DIR "include/leveldb")
set(PUBLIC_HEADERS
${PUBLIC_INCLUDE_DIR}/cache.h
${PUBLIC_INCLUDE_DIR}/c.h
${PUBLIC_INCLUDE_DIR}/comparator.h
${PUBLIC_INCLUDE_DIR}/db.h
${PUBLIC_INCLUDE_DIR}/dumpfile.h
${PUBLIC_INCLUDE_DIR}/env.h
${PUBLIC_INCLUDE_DIR}/filter_policy.h
${PUBLIC_INCLUDE_DIR}/iterator.h
${PUBLIC_INCLUDE_DIR}/options.h
${PUBLIC_INCLUDE_DIR}/slice.h
${PUBLIC_INCLUDE_DIR}/status.h
${PUBLIC_INCLUDE_DIR}/table_builder.h
${PUBLIC_INCLUDE_DIR}/table.h
${PUBLIC_INCLUDE_DIR}/write_batch.h
)
add_library(leveldb STATIC ${SRCS} )
set_target_properties (leveldb PROPERTIES PUBLIC_HEADER "${PUBLIC_HEADERS}")
# CHECK_INCLUDE_FILE(stdint.h HAVE_STDINT_H)


# TESTS
add_library(leveldb-testharness STATIC ${TESTHARNESS_SRCS})
macro(leveldb_tests)
	foreach(t ${ARGN})
		get_filename_component(_tname ${t} NAME_WE)
		add_executable(${_tname} ${t})
		target_link_libraries(${_tname}
			leveldb-testharness
			leveldb
		)
	endforeach()
endmacro()

leveldb_tests(
	db/db_bench.cc
	util/arena_test.cc
	db/c_test.c
	util/cache_test.cc
	util/coding_test.cc
	db/corruption_test.cc
	util/crc32c_test.cc
	db/db_test.cc
	db/dbformat_test.cc
	util/env_test.cc
	db/log_test.cc
	table/table_test.cc
	db/skiplist_test.cc
	db/version_edit_test.cc
	db/version_set_test.cc
	db/write_batch_test.cc)

# TODO: memenv_test

install(TARGETS leveldb
        EXPORT leveldb-targets
        RUNTIME DESTINATION bin
        PUBLIC_HEADER DESTINATION include/leveldb/
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)

