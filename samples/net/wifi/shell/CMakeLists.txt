# SPDX-License-Identifier: Apache-2.0

cmake_minimum_required(VERSION 3.20.0)

# Provide path to the root of Matter repository
if(DEFINED ENV{CHIP_ROOT})
    set(CHIP_ROOT $ENV{CHIP_ROOT})
else()
    get_filename_component(CHIP_ROOT $ENV{HOME}/matter REALPATH)
endif()

get_filename_component(NXP_MATTER_SUPPORT_DIR ${CHIP_ROOT}/third_party/nxp/nxp_matter_support REALPATH)
get_filename_component(GEN_DIR ${CHIP_ROOT}/zzz_generated/ REALPATH)
get_filename_component(ALL_CLUSTERS_COMMON_DIR ${CHIP_ROOT}/examples/all-clusters-app/all-clusters-common REALPATH)
get_filename_component(THERMOSTAT_NXP_ROOT ${CHIP_ROOT}/examples/thermostat/nxp/zephyr REALPATH)
get_filename_component(THERMOSTAT_NXP_COMMON_DIR ${CHIP_ROOT}/examples/thermostat/nxp/common REALPATH)
get_filename_component(THERMOSTAT_NXP_ZAP_DIR ${CHIP_ROOT}/examples/thermostat/nxp/zap REALPATH)
get_filename_component(EXAMPLE_PLATFORM_NXP_COMMON_DIR ${CHIP_ROOT}/examples/platform/nxp/common REALPATH)

# Add the CHIP stack & NXP port libs as an external module
list(APPEND ZEPHYR_EXTRA_MODULES ${CHIP_ROOT}/config/nxp/chip-module)

# Add application prj.conf & Kconfig
#set(CONF_FILE ${CONF_FILE} ${CMAKE_CURRENT_SOURCE_DIR}/prj.conf)
set(KCONFIG_ROOT ${CMAKE_CURRENT_SOURCE_DIR}/Kconfig)

find_package(Zephyr REQUIRED HINTS $ENV{ZEPHYR_BASE})
project(wifi)
enable_language(CXX)

include(${CHIP_ROOT}/config/nxp/app/enable-gnu-std.cmake)
include(${CHIP_ROOT}/src/app/chip_data_model.cmake)

# Add additional configuration
target_compile_definitions(app
    PRIVATE
    DISABLE_LOGGING
)

target_include_directories(app
    PRIVATE
    ${ALL_CLUSTERS_COMMON_DIR}/include
    ${THERMOSTAT_NXP_COMMON_DIR}/main/include
    ${GEN_DIR}/app-common
    ${EXAMPLE_PLATFORM_NXP_COMMON_DIR}/device_manager/include
    ${EXAMPLE_PLATFORM_NXP_COMMON_DIR}/icd/include
    ${EXAMPLE_PLATFORM_NXP_COMMON_DIR}/device_callbacks/include
    ${EXAMPLE_PLATFORM_NXP_COMMON_DIR}/factory_data/include
    ${EXAMPLE_PLATFORM_NXP_COMMON_DIR}/app_task/include
    ${EXAMPLE_PLATFORM_NXP_COMMON_DIR}/matter_button/include
    ${EXAMPLE_PLATFORM_NXP_COMMON_DIR}/app_ble/include
)

# Use "main.cpp" of Matter source
#FILE(GLOB app_sources src/*.c)
target_sources(app
    PRIVATE
    ${app_sources}
    ${THERMOSTAT_NXP_ROOT}/main/main.cpp
    ${THERMOSTAT_NXP_COMMON_DIR}/main/AppTask.cpp
    ${THERMOSTAT_NXP_COMMON_DIR}/main/DeviceCallbacks.cpp
    ${THERMOSTAT_NXP_COMMON_DIR}/main/ZclCallbacks.cpp
    ${EXAMPLE_PLATFORM_NXP_COMMON_DIR}/device_callbacks/source/CommonDeviceCallbacks.cpp
    ${EXAMPLE_PLATFORM_NXP_COMMON_DIR}/device_manager/source/CHIPDeviceManager.cpp
    ${EXAMPLE_PLATFORM_NXP_COMMON_DIR}/app_task/source/AppTaskBase.cpp
    ${EXAMPLE_PLATFORM_NXP_COMMON_DIR}/app_task/source/AppTaskZephyr.cpp
    ${EXAMPLE_PLATFORM_NXP_COMMON_DIR}/matter_button/source/ButtonRegistrationEmpty.cpp
    ${EXAMPLE_PLATFORM_NXP_COMMON_DIR}/icd/source/ICDUtil.cpp
    ${ALL_CLUSTERS_COMMON_DIR}/src/binding-handler.cpp
    ${EXAMPLE_PLATFORM_NXP_COMMON_DIR}/app_ble/source/BLEApplicationManagerEmpty.cpp
 )

if(CONFIG_CHIP_WIFI)
    chip_configure_data_model(app
        INCLUDE_SERVER
        ZAP_FILE ${THERMOSTAT_NXP_ZAP_DIR}/thermostat_matter_wifi.zap
    )
endif()

if (CONFIG_CHIP_LIB_SHELL)
    target_compile_definitions(app PRIVATE
        ENABLE_CHIP_SHELL
    )

    target_sources(app PRIVATE
        ${EXAMPLE_PLATFORM_NXP_COMMON_DIR}/matter_cli/source/AppCLIBase.cpp
        ${EXAMPLE_PLATFORM_NXP_COMMON_DIR}/matter_cli/source/AppCLIZephyr.cpp
        ${CHIP_ROOT}/examples/shell/shell_common/cmd_misc.cpp
        ${CHIP_ROOT}/examples/shell/shell_common/cmd_otcli.cpp
        ${CHIP_ROOT}/examples/shell/shell_common/cmd_server.cpp
    )

    target_include_directories(app PRIVATE
        ${EXAMPLE_PLATFORM_NXP_COMMON_DIR}/matter_cli/include
        ${CHIP_ROOT}/examples/shell/shell_common/include
    )
endif()

if (CONFIG_CHIP_SECURE_DAC_PRIVATE_KEY_STORAGE OR CONFIG_CHIP_ENABLE_SECURE_WHOLE_FACTORY_DATA)
    target_compile_definitions(app PUBLIC
        EXTERNAL_FACTORY_DATA_PROVIDER_IMPL_HEADER=\"platform/nxp/${CONFIG_CHIP_NXP_PLATFORM_FOLDER_NAME}/FactoryDataProviderImpl.h\"
    )
    target_sources(app PRIVATE
        ${EXAMPLE_PLATFORM_NXP_COMMON_DIR}/factory_data/source/AppFactoryDataExample.cpp
    )
else()
    target_compile_definitions(app PUBLIC
        EXTERNAL_FACTORY_DATA_PROVIDER_IMPL_HEADER=\"platform/nxp/common/factory_data/FactoryDataProviderFwkImpl.h\"
    )
    target_sources(app PRIVATE
        ${EXAMPLE_PLATFORM_NXP_COMMON_DIR}/factory_data/source/AppFactoryDataDefaultImpl.cpp
    )
endif()

if (CONFIG_CHIP_OTA_REQUESTOR)
    target_include_directories(app PRIVATE
        ${EXAMPLE_PLATFORM_NXP_COMMON_DIR}/ota_requestor/include
    )
	target_sources(app PRIVATE
        ${EXAMPLE_PLATFORM_NXP_COMMON_DIR}/ota_requestor/source/OTARequestorInitiator.cpp
        ${EXAMPLE_PLATFORM_NXP_COMMON_DIR}/ota_requestor/source/OTARequestorInitiatorCommon.cpp
    )
endif()
#message(STATUS "ZEPHYR_EXTRA_MODULES = ${ZEPHYR_EXTRA_MODULES}")

