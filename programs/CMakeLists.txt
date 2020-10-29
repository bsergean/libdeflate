set(TEST_PROGRAMS_DEFINITIONS _POSIX_C_SOURCE=200809L
                              _FILE_OFFSET_BITS=64
                              HAVE_CONFIG_H)

# Small library for common code
add_library(programs)
target_sources(programs PRIVATE prog_util.c test_util.c tgetopt.c)
target_link_libraries(programs deflate)

# test checksums
set(PROGRAM_TARGET_NAMES
  gzip
  benchmark
  checksum
  test_custom_malloc
  test_incomplete_codes
  test_litrunlen_overflow
  test_slow_decompression
  test_trailing_bytes
)
foreach(PROGRAM_TARGET_NAME ${PROGRAM_TARGET_NAMES})
  add_executable(${PROGRAM_TARGET_NAME})
  target_sources(${PROGRAM_TARGET_NAME} PRIVATE ${PROGRAM_TARGET_NAME}.c ${TEST_PROGRAMS_COMMON_SRC})
  target_link_libraries(${PROGRAM_TARGET_NAME} programs deflate z)
  target_compile_definitions(${PROGRAM_TARGET_NAME} PRIVATE ${TEST_PROGRAMS_DEFINITIONS})
endforeach()
