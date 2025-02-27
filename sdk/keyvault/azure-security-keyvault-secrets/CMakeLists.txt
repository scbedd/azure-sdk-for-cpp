# Copyright (c) Microsoft Corporation. All rights reserved.
# SPDX-License-Identifier: MIT

# setting CMAKE_TOOLCHAIN_FILE must happen before creating the project
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../../../cmake-modules")
include(AzureVcpkg)
az_vcpkg_integrate()

cmake_minimum_required (VERSION 3.13)
project(azure-security-keyvault-secrets LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)

include(AzureVersion)
include(AzureCodeCoverage)
include(AzureTransportAdapters)
include(AzureDoxygen)
include(AzureGlobalCompileOptions)
include(AzureConfigRTTI)
# Add create_map_file function
include(CreateMapFile)

if(NOT AZ_ALL_LIBRARIES)
  find_package(azure-core-cpp "1.2.0" CONFIG QUIET)
  if(NOT azure-core-cpp_FOUND)
    find_package(azure-core-cpp "1.2.0" REQUIRED)
  endif()
endif()

set(
  AZURE_SECURITY_KEYVAULT_SECRETS_HEADER
    inc/azure/keyvault/secrets/dll_import_export.hpp
    inc/azure/keyvault/secrets/keyvault_secret.hpp
    inc/azure/keyvault/secrets/keyvault_secret_properties.hpp
    inc/azure/keyvault/secrets/secret_client.hpp
    inc/azure/keyvault/keyvault_secrets.hpp
    inc/azure/keyvault/secrets/keyvault_deleted_secret.hpp
    inc/azure/keyvault/secrets/keyvault_secret_paged_response.hpp
    inc/azure/keyvault/secrets/keyvault_backup_secret.hpp
    inc/azure/keyvault/secrets/keyvault_operations.hpp
    inc/azure/keyvault/secrets/keyvault_options.hpp
)

set(
  AZURE_SECURITY_KEYVAULT_SECRETS_SOURCE
    src/private/package_version.hpp
    src/private/keyvault_protocol.hpp
    src/private/secret_constants.hpp
    src/private/secret_serializers.hpp
    src/keyvault_protocol.cpp
    src/secret_client.cpp
    src/secret_serializers.cpp
    src/keyvault_operations.cpp
    src/keyvault_secret_paged_response.cpp
    src/keyvault_secret_properties.cpp
)

add_library(azure-security-keyvault-secrets ${AZURE_SECURITY_KEYVAULT_SECRETS_HEADER} ${AZURE_SECURITY_KEYVAULT_SECRETS_SOURCE})
add_library(Azure::azure-security-keyvault-secrets ALIAS azure-security-keyvault-secrets)

target_include_directories(
  azure-security-keyvault-secrets
    PUBLIC
      $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/inc>
      $<INSTALL_INTERFACE:include>
)

# Include shared source code
# NOTE: Use shared-code only within .cpp files. DO NEVER consume the shared-code from header files.
target_include_directories(
  azure-security-keyvault-secrets
    PRIVATE
      $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../azure-security-keyvault-shared/inc>
)

target_link_libraries(azure-security-keyvault-secrets PUBLIC Azure::azure-core)

# coverage. Has no effect if BUILD_CODE_COVERAGE is OFF
create_code_coverage(keyvault azure-security-keyvault-secrets azure-security-keyvault-secrets-test "tests?/*;samples?/*")

get_az_version("${CMAKE_CURRENT_SOURCE_DIR}/src/private/package_version.hpp")
generate_documentation(azure-security-keyvault-secrets ${AZ_LIBRARY_VERSION})
if(BUILD_TESTING)

  if (NOT AZ_ALL_LIBRARIES)
    include(AddGoogleTest)
    enable_testing ()
  endif()
  
  add_subdirectory(test/ut)
  add_subdirectory(test/test-app)
endif()

if (BUILD_PERFORMANCE_TESTS)
  #add_subdirectory(test/perf)
endif()

if(BUILD_SAMPLES)
  add_subdirectory(test/samples)
endif()

az_vcpkg_export(
    azure-security-keyvault-secrets
    SECURITY_KEYVAULT_SECRETS
    "azure/keyvault/secrets/dll_import_export.hpp"
  )

az_rtti_setup(
  azure-security-keyvault-secrets
  SECURITY_KEYVAULT_SECRETS
  "azure/keyvault/secrets/rtti.hpp"
)
