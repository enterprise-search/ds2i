cmake_minimum_required(VERSION 3.20)
project(DS2I)

configure_file(
  ${DS2I_SOURCE_DIR}/ds2i_config.hpp.in
  ${DS2I_SOURCE_DIR}/ds2i_config.hpp
  ESCAPE_QUOTES)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "Release")
endif()

if (UNIX)
   # C++11
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++20")

   # For hardware popcount and other special instructions
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -march=native")

   # Extensive warnings
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Wno-missing-braces")
   # Silence a warning bug in Boost
   if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
     set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-unused-local-typedefs")
   endif ()
   # set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wconversion")

   if (USE_SANITIZERS)
     set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=address -fno-omit-frame-pointer")
   endif ()

   #set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -ggdb") # Add debug info anyway

endif()

# set(Boost_USE_STATIC_LIBS ON)
find_package(Boost COMPONENTS iostreams unit_test_framework filesystem system REQUIRED)
include_directories(${Boost_INCLUDE_DIRS})
link_directories(${Boost_LIBRARY_DIRS})

# stxxl
find_library(STXXL
	NAME stxxl
	REQUIRED
	)
set(STXXL_CXX_FLAGS -fopenmp)
set(STXXL_LIBRARIES stxxl)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${STXXL_CXX_FLAGS}")

# add the root directory to include path to make includes absolute
include_directories(${DS2I_SOURCE_DIR})

add_subdirectory(succinct EXCLUDE_FROM_ALL)
add_subdirectory(FastPFor EXCLUDE_FROM_ALL)

add_executable(create_freq_index create_freq_index.cpp)
target_link_libraries(create_freq_index
  ${Boost_LIBRARIES}
  FastPFor_lib
  )

add_executable(optimal_hybrid_index optimal_hybrid_index.cpp)
target_link_libraries(optimal_hybrid_index
  ${Boost_LIBRARIES}
  FastPFor_lib
  ${STXXL_LIBRARIES}
)

add_executable(create_wand_data create_wand_data.cpp)
target_link_libraries(create_wand_data
  ${Boost_LIBRARIES}
  )

add_executable(queries queries.cpp)
target_link_libraries(queries
  ${Boost_LIBRARIES}
  FastPFor_lib
  )

add_executable(profile_queries profile_queries.cpp)
target_link_libraries(profile_queries
  ${Boost_LIBRARIES}
  FastPFor_lib
  )

add_executable(profile_decoding profile_decoding.cpp)
target_link_libraries(profile_decoding
  ${Boost_LIBRARIES}
  FastPFor_lib
  )

enable_testing()
add_subdirectory(test)
