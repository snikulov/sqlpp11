# Copyright (c) 2013-2021, Roland Bock
# Copyright (c) 2016 Christian Dávid
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without modification,
# are permitted provided that the following conditions are met:
#
#   Redistributions of source code must retain the above copyright notice, this
#   list of conditions and the following disclaimer.
#
#   Redistributions in binary form must reproduce the above copyright notice, this
#   list of conditions and the following disclaimer in the documentation and/or
#   other materials provided with the distribution.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
# ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
# WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
# DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR
# ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
# (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
# LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON
# ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
# SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

### Preamble
cmake_minimum_required(VERSION 3.14)
project(sqlpp11 VERSION 0.1 LANGUAGES CXX)

### Project Wide Setup
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake/")

option(MARIADB_CONNECTOR "Build MariaDB Connector" OFF)
option(MYSQL_CONNECTOR "Build MySQL Connector" OFF)
option(POSTGRESQL_CONNECTOR "Build PostgreSQL Connector" OFF)
option(SQLITE3_CONNECTOR "Build SQLite3 Connector" OFF)
option(SQLCIPHER_CONNECTOR "Build SQLite3 Connector with SQLCipher" OFF)

if(MYSQL_CONNECTOR)
  find_package(MySQL REQUIRED)
else()
  message(STATUS "Not building tests for MYSQL_CONNECTOR")
endif()

if(MARIADB_CONNECTOR)
  find_package(MariaDB REQUIRED)
else()
  message(STATUS "Not building tests for MARIAB_CONNECTOR")
endif()

if(POSTGRESQL_CONNECTOR)
  find_package(PostgreSQL REQUIRED)
else()
  message(STATUS "Not building tests for POSTGRESQL_CONNECTOR")
endif()

if(SQLITE3_CONNECTOR)
  find_package(SQLite3 REQUIRED)
else()
  message(STATUS "Not building tests for SQLITE3_CONNECTOR")
endif()

if(SQLCIPHER_CONNECTOR)
  find_package(SQLCipher REQUIRED)
else()
  message(STATUS "Not building tests for SQLCIPHER_CONNECTOR")
endif()

include(CTest)

option(USE_SYSTEM_DATE "\
    Use find_package to find installed HowardHinnant's \
    date library instead of fetching it from github" OFF
)

if(USE_SYSTEM_DATE)
    find_package(date REQUIRED)
endif()

### Dependencies
add_subdirectory(dependencies)

### Core targets
add_library(sqlpp11 INTERFACE)
add_library(sqlpp11::sqlpp11 ALIAS sqlpp11)

target_link_libraries(sqlpp11 INTERFACE date::date)

target_include_directories(sqlpp11 INTERFACE
  $<BUILD_INTERFACE:${sqlpp11_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
  )

target_compile_features(sqlpp11 INTERFACE
	cxx_alias_templates
	cxx_auto_type
	cxx_constexpr
	cxx_decltype
	cxx_defaulted_functions
	cxx_defaulted_move_initializers
	cxx_deleted_functions
	cxx_explicit_conversions
	cxx_lambdas
	cxx_noexcept
	cxx_nullptr
	cxx_range_for
	cxx_right_angle_brackets
	cxx_rvalue_references
	cxx_static_assert
	cxx_trailing_return_types
	cxx_uniform_initialization
	cxx_template_template_parameters
  cxx_variadic_templates
  )

### Packaging
install(DIRECTORY "${PROJECT_SOURCE_DIR}/include/sqlpp11"
        DESTINATION include
)

install(TARGETS sqlpp11
        EXPORT Sqlpp11Targets
)

install(PROGRAMS "${PROJECT_SOURCE_DIR}/scripts/ddl2cpp"
        RENAME sqlpp11-ddl2cpp
        DESTINATION bin
)

include(CMakePackageConfigHelpers)

write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/cmake/Sqlpp11ConfigVersion.cmake"
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY AnyNewerVersion
)

export(EXPORT Sqlpp11Targets
  FILE "${CMAKE_CURRENT_BINARY_DIR}/cmake/Sqlpp11Targets.cmake"
)

configure_file(cmake/Sqlpp11Config.cmake
  "${CMAKE_CURRENT_BINARY_DIR}/cmake/Sqlpp11Config.cmake"
  COPYONLY
)

set(ConfigPackageLocation lib/cmake/Sqlpp11)
install(EXPORT Sqlpp11Targets
  NAMESPACE sqlpp11::
  DESTINATION
    ${ConfigPackageLocation}
)

install(
  FILES
    "cmake/Sqlpp11Config.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/cmake/Sqlpp11ConfigVersion.cmake"
  DESTINATION
    ${ConfigPackageLocation}
)

### Tests
if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME AND BUILD_TESTING)
  add_subdirectory(tests)
endif()
