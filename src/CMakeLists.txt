CMAKE_MINIMUM_REQUIRED(VERSION 3.14 FATAL_ERROR)
set (CMAKE_CXX_STANDARD 17)

# CMake 3.14+
include(FetchContent)

if (DEFINED ENV{RELIC_MAIN})
  set(RELIC_GIT_TAG "origin/main")
else ()
  set(RELIC_GIT_TAG "1885ae3b681c423c72b65ce1fe70910142cf941c")
endif ()

message(STATUS "Relic will be built from: ${RELIC_GIT_TAG}")

FetchContent_Declare(
  relic
  GIT_REPOSITORY https://github.com/relic-toolkit/relic.git
  GIT_TAG        ${RELIC_GIT_TAG}
)
FetchContent_MakeAvailable(relic)

file(GLOB HEADERS ${CMAKE_CURRENT_SOURCE_DIR}/*.hpp)
source_group("SrcHeaders" FILES ${HEADERS})

include_directories(
  ${INCLUDE_DIRECTORIES}
  ${relic_SOURCE_DIR}/include
  ${relic_BINARY_DIR}/include
  )

if (GMP_FOUND)
  include_directories(${GMP_INCLUDES})
endif(GMP_FOUND)

add_library(bls STATIC
  ${HEADERS}
  ${CMAKE_CURRENT_SOURCE_DIR}/privatekey.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/bls.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/elements.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/schemes.cpp
  )

set(OPREFIX object_)

if (GMP_FOUND)
  message("GMP_FOUND in src/CMakeLists.txt")
  target_link_libraries(bls gmp)
endif()
if (SODIUM_FOUND)
  message("SODIUM_FOUND in src/CMakeLists.txt")
  target_link_libraries(bls sodium)
  target_compile_definitions(bls PRIVATE BLSALLOC_SODIUM=1)
endif()

target_link_libraries(bls relic_s)

file(GLOB includes "${CMAKE_CURRENT_SOURCE_DIR}/*.hpp")
install(DIRECTORY ${relic_SOURCE_DIR}/include/ DESTINATION include/chiabls)
install(DIRECTORY ${relic_BINARY_DIR}/include/ DESTINATION include/chiabls)
install(FILES ${includes} DESTINATION include/chiabls)
install(TARGETS bls DESTINATION lib)

if (BUILD_BLS_TESTS)
  include_directories(${CMAKE_CURRENT_SOURCE_DIR}/../contrib/catch)
  add_executable(runtest test.cpp)
  target_link_libraries(runtest bls)
endif()

if (BUILD_BLS_BENCHMARKS)
  add_executable(runbench test-bench.cpp)
  target_link_libraries(runbench bls)
endif()
