cmake_minimum_required(VERSION 3.16)
if(APPLE OR WIN32)
    project("Crochet Charts")
else()
    project("CrochetCharts")
endif()

set(PROJECT_LONGNAME "Crochet Charts")

if(WIN32)
    set(EXE_NAME "Crochet_Charts")
else()
    set(EXE_NAME ${PROJECT_NAME})
endif()

#Qt6 requires C++17
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Appends the cmake/modules path to MAKE_MODULE_PATH variable...
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules ${CMAKE_MODULE_PATH})

#Forces out of source builds just in case.
include(MacroOutOfSourceBuild)
macro_ensure_out_of_source_build("${PROJECT_NAME} requires an out of source build.")

set(CMAKE_CONFIGURATION_TYPES Debug Release Profile CACHE TYPE INTERNAL FORCE)
if(CMAKE_BUILD_TYPE STREQUAL "")
    set(CMAKE_BUILD_TYPE "Debug")
endif()

set (HUNSPELL_FIND_QUIETLY TRUE)
include(FindHunSpell)

include(GetGitRevisionDescription)
git_describe(VERSION --tags --dirty=w)

string(REGEX REPLACE "^([0-9]+)\\..*" "\\1" VERSION_MAJOR "${VERSION}")
string(REGEX REPLACE "^[0-9]+\\.([0-9]+).*" "\\1" VERSION_MINOR "${VERSION}")
string(REGEX REPLACE "^[0-9]+\\.[0-9]+\\.([0-9]+).*" "\\1" VERSION_PATCH "${VERSION}")
string(REGEX REPLACE "^[0-9]+\\.[0-9]+\\.[0-9]+(.*)" "\\1" VERSION_SHA1 "${VERSION}")
set(VERSION_SHORT "${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}")

set(PROJECT_LIFE         "2010-2013")

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules/version.cpp.in
                ${CMAKE_CURRENT_BINARY_DIR}/version.cpp)
set(crochet_version "${CMAKE_CURRENT_BINARY_DIR}/version.cpp")

option(UNIT_TESTING  "Build the test libraries for this software" OFF)
option(DOCS "Create docbook documentation" OFF)
option(DOXYGEN "Create Doxygen documentation" OFF)

find_package(Qt6 REQUIRED COMPONENTS Core Gui Widgets Svg SvgWidgets Network PrintSupport)
if(UNIT_TESTING)
    find_package(Qt6 REQUIRED COMPONENTS Test)
endif()

message("-------------------------------------------------------")
message(STATUS ${PROJECT_NAME} " Version: " ${VERSION})
message(STATUS "CMake Build Type: " ${CMAKE_BUILD_TYPE})
message(STATUS "Create Documentation: " ${DOCS})
message(STATUS "Unit Testing: " ${UNIT_TESTING})
message(STATUS "Found Hunspell: " ${HUNSPELL_FOUND})
message(STATUS "Doxygen Docs: " ${DOXYGEN})
message(STATUS "Build flags: ${CMAKE_CXX_FLAGS}")
message(STATUS "Linker flags: ${CMAKE_EXE_LINKER_FLAGS}")
message("-------------------------------------------------------")

include_directories(${CMAKE_CURRENT_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR})

set(CMAKE_AUTOMOC TRUE)
set(CMAKE_AUTOUIC TRUE)
set(CMAKE_AUTORCC TRUE)

###################################################################################
#CPack includes to create installers
###################################################################################
include("${CMAKE_CURRENT_SOURCE_DIR}/resources/installers.cmake")

add_subdirectory(docs)
add_subdirectory(src)

if(UNIT_TESTING)
    add_subdirectory(tests)
endif()

set(CPACK_BINARY_DRAGNDROP ON)
include(CPack)
