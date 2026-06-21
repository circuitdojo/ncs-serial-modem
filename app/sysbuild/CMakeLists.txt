#
# Copyright (c) 2026 Nordic Semiconductor ASA
#
# SPDX-License-Identifier: LicenseRef-Nordic-5-Clause
#

# Resolve cmake variable references (e.g. ${APP_DIR}) in mcuboot overlay paths
# before sysbuild configures the child images. This is needed because
# mcuboot's cmake context does not have APP_DIR defined, so string(CONFIGURE)
# in zephyr_list(NORMALIZE_PATHS) would expand it to an empty string.
# At this point APP_DIR is already set from the -DAPP_DIR= cmake argument.
if(DEFINED mcuboot_EXTRA_DTC_OVERLAY_FILE)
  message(STATUS "sysbuild: mcuboot_EXTRA_DTC_OVERLAY_FILE (raw):      ${mcuboot_EXTRA_DTC_OVERLAY_FILE}")
  message(STATUS "sysbuild: APP_DIR:                                    ${APP_DIR}")
  string(CONFIGURE "${mcuboot_EXTRA_DTC_OVERLAY_FILE}" mcuboot_EXTRA_DTC_OVERLAY_FILE_RESOLVED)
  message(STATUS "sysbuild: mcuboot_EXTRA_DTC_OVERLAY_FILE (resolved):  ${mcuboot_EXTRA_DTC_OVERLAY_FILE_RESOLVED}")
  set(mcuboot_EXTRA_DTC_OVERLAY_FILE "${mcuboot_EXTRA_DTC_OVERLAY_FILE_RESOLVED}"
      CACHE STRING "" FORCE)
else()
  message(STATUS "sysbuild: mcuboot_EXTRA_DTC_OVERLAY_FILE not defined, skipping resolution")
endif()

# The MCUboot S1 variant is configured through Zephyr's variant CMake path and
# does not rediscover module-provided board roots. Propagate the board root for
# the custom Circuit Dojo boards explicitly so the variant can resolve the
# circuitdojo_feather_nrf9151 board.
set(mcuboot_s1_variant_BOARD_ROOT "${CMAKE_CURRENT_LIST_DIR}/../../../nfed"
    CACHE PATH "Board root for MCUboot S1 variant" FORCE)

# The same variant configure step otherwise falls back to /usr/bin/python, which
# may not have Zephyr's Python dependencies installed. Pin it to west's Python.
if(DEFINED WEST_PYTHON)
  set(Python3_EXECUTABLE "${WEST_PYTHON}" CACHE FILEPATH "Python for child images" FORCE)
  get_filename_component(WEST_PYTHON_DIR "${WEST_PYTHON}" DIRECTORY)
  set(ENV{PATH} "${WEST_PYTHON_DIR}:$ENV{PATH}")
endif()

find_package(Sysbuild REQUIRED HINTS $ENV{ZEPHYR_BASE})
project(sysbuild LANGUAGES)

set(MERGED_HEX_APP_IMAGE app)
include(${CMAKE_CURRENT_LIST_DIR}/../../cmake/merged_hex.cmake)
