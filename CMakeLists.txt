# Copyright (c) 2025 Nordic Semiconductor ASA
# SPDX-License-Identifier: LicenseRef-Nordic-5-Clause
#
# This CMake file is picked by the Zephyr build system because it is defined
# as the module CMake entry point (see zephyr/module.yml).

zephyr_include_directories(include)

# The nRF UARTE driver needs the fixes in zephyr/patches (see the README
# there); without them the AT UART can stop receiving for good after line
# errors. Refuse to build an unpatched driver rather than ship that.
if(CONFIG_UART_NRFX_UARTE)
  set(uarte_driver ${ZEPHYR_BASE}/drivers/serial/uart_nrfx_uarte.c)
  set_property(DIRECTORY APPEND PROPERTY CMAKE_CONFIGURE_DEPENDS ${uarte_driver})
  file(READ ${uarte_driver} uarte_source)
  # One line of each patch's current version.
  foreach(marker
      "LOW_POWER_ENABLED(cfg) || IS_CBWT_LEGACY(dev)) {"
      "static uint32_t stoprx_guard_ms(const struct device *dev)")
    string(FIND "${uarte_source}" "${marker}" found)
    if(found EQUAL -1)
      message(FATAL_ERROR
        "${uarte_driver} lacks the Serial Modem UARTE patches.\n"
        "Run \"west patch apply\" in the workspace (after every \"west update\").")
    endif()
  endforeach()
endif()

add_subdirectory(drivers)
add_subdirectory(lib)
