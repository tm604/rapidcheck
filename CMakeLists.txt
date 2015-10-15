cmake_minimum_required(VERSION 2.8.7)
project(rapidcheck)
set (VERSION_MAJOR 0)
set (VERSION_MINOR 0)
set (VERSION_PATCH 1)
set (PROJECT_VERSION "${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}")

set (PROJECT_VERSION_REVISION 1)
enable_testing()

option(RC_ENABLE_TESTS "Build RapidCheck tests" OFF)
option(RC_ENABLE_EXAMPLES "Build RapidCheck examples" OFF)

if(MSVC)
  # /bigobj - some object files become very large so we need this
  # /wd4503 - truncation of decorated name, not much we can do about it so
  #           disable it
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /bigobj /wd4503 /WX")
  # /RTC* is incompatible with /O2 needed for Random.cpp to speed it up
  string(REGEX REPLACE "/RTC(su|[1su])" ""
    CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG}")
else()
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Werror -std=c++11")
endif()

add_library(rapidcheck
  src/BeforeMinimalTestCase.cpp
  src/Check.cpp
  src/Classify.cpp
  src/GenerationFailure.cpp
  src/Random.cpp
  src/Show.cpp
  src/detail/Any.cpp
  src/detail/Assertions.cpp
  src/detail/Configuration.cpp
  src/detail/FrequencyMap.cpp
  src/detail/ImplicitParam.cpp
  src/detail/LogTestListener.cpp
  src/detail/MapParser.cpp
  src/detail/Platform.cpp
  src/detail/Property.cpp
  src/detail/PropertyContext.cpp
  src/detail/Results.cpp
  src/detail/TestParams.cpp
  src/detail/Testing.cpp
  src/gen/Numeric.cpp
  src/gen/Text.cpp
  src/gen/detail/ExecHandler.cpp
  src/gen/detail/GenerationHandler.cpp
  src/gen/detail/Recipe.cpp
  src/gen/detail/ScaleInteger.cpp
  )

# Random is used a LOT so it should preferrably be really fast.
if(MSVC)
  set_property(SOURCE src/Random.cpp
    APPEND_STRING PROPERTY COMPILE_FLAGS " /O2")
else()
  set_property(SOURCE src/Random.cpp
    APPEND_STRING PROPERTY COMPILE_FLAGS " -O3")
endif()

target_include_directories(rapidcheck PUBLIC include)
install(
	DIRECTORY include/
	DESTINATION "include"
)
install(
	TARGETS rapidcheck
	ARCHIVE DESTINATION lib
)

add_subdirectory(ext)

if (RC_ENABLE_TESTS)
  add_subdirectory(test)
endif()

if (RC_ENABLE_EXAMPLES)
  add_subdirectory(examples)
endif()

add_subdirectory(extras)

include (InstallRequiredSystemLibraries)
set (CPACK_PACKAGE_NAME "librapidcheck")
set (CPACK_PACKAGE_VENDOR "")
set (CPACK_PACKAGE_VERSION_MAJOR "${VERSION_MAJOR}")
set (CPACK_PACKAGE_VERSION_MINOR "${VERSION_MINOR}")
set (CPACK_PACKAGE_VERSION_PATCH "${VERSION_PATCH}")
set (CPACK_PACKAGE_CONTACT "Tom Molesworth <tom@perlsite.co.uk>")
set (CPACK_PACKAGE_DESCRIPTION_SUMMARY "C++ quickcheck-style test framework")
set (CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_CURRENT_SOURCE_DIR}/README.debian")
set (CPACK_GENERATOR "TGZ;DEB")
string(TOLOWER "${CPACK_PACKAGE_NAME}" CPACK_PACKAGE_NAME_LOWERCASE)
find_program(DPKG_PROGRAM dpkg)
if(DPKG_PROGRAM)
	execute_process(
		COMMAND ${DPKG_PROGRAM} --print-architecture
		OUTPUT_VARIABLE CPACK_DEBIAN_PACKAGE_ARCHITECTURE
		OUTPUT_STRIP_TRAILING_WHITESPACE
		)
	set(CPACK_PACKAGE_FILE_NAME "${CPACK_PACKAGE_NAME_LOWERCASE}_${PROJECT_VERSION}-${PROJECT_VERSION_REVISION}_${CPACK_DEBIAN_PACKAGE_ARCHITECTURE}")   
else(DPKG_PROGRAM)
	set(CPACK_PACKAGE_FILE_NAME "${CPACK_PACKAGE_NAME_LOWERCASE}_${PROJECT_VERSION}-${PROJECT_VERSION_REVISION}_${CMAKE_SYSTEM_NAME}")
endif(DPKG_PROGRAM)
# set (CPACK_DEBIAN_PACKAGE_SHLIBDEPS "ON")
include (CPack)
