# -----------------------------------------------------------------------------
# micro-os-plus-iii-devices — drivers and silicon support for µOS++ III SMP.
#
# Deliberately NOT part of micro-os-plus-iii-smp. That repository is the RTOS
# kernel, and is kept merge-clean against upstream micro-os-plus-iii; FatFs, an
# SD driver and a DWC2 USB stack are not the RTOS, they are drivers that use it.
#
# A build includes this project only when it needs it -- a Cortex-M target has
# no use for BCM2837 support.
#
# Consumed as:
#   add_subdirectory (micro-os-plus-iii-devices)
#   target_link_libraries (my-app PRIVATE micro-os-plus::devices)
#
# Requires micro-os-plus::iii (the kernel) to have been defined first.
#
# Targets exported:
#   micro-os-plus::devices          drivers over the BCM2837 SD host + DWC2
#   micro-os-plus::devices-rk3506   the same drivers over the RK3506 SD host
#   micro-os-plus::soc-bcm2837      BCM2837 silicon support
# -----------------------------------------------------------------------------

cmake_minimum_required (VERSION 3.20)

project (
  micro-os-plus-iii-devices
  DESCRIPTION "µOS++ III SMP — drivers and SoC support"
  LANGUAGES C CXX
)

# -----------------------------------------------------------------------------
# Drivers. One copy serves every architecture: measured identical across
# AArch32 and AArch64, and verified to compile for both.

add_library (micro-os-plus-iii-devices-interface INTERFACE)

target_include_directories (
  micro-os-plus-iii-devices-interface INTERFACE "include" "fatfs"
)

target_sources (
  micro-os-plus-iii-devices-interface
  INTERFACE src/sd.cpp
            src/flatfs.cpp
            src/usb_dwc2.cpp
            fatfs/ff.c
            fatfs/diskio_sd.cpp
            fatfs/fatfs_hw.cpp
)

add_library (micro-os-plus::devices ALIAS micro-os-plus-iii-devices-interface)

# -----------------------------------------------------------------------------
# The same drivers over the RK3506's SD host (Luckfox Lyra B).
#
# A SEPARATE target, not a branch inside the one above, for the reason a board
# is a directory and not an `elseif`: what a Raspberry Pi compiles must not
# change because a second board exists. micro-os-plus::devices is byte for byte
# the target it was; a Lyra links micro-os-plus::devices-rk3506 instead, and the
# two share every line of filesystem code.
#
# The SD driver underneath -- soc/rk3506/src/sdmmc.cpp -- is the predecessor's
# cortex-a7 driver, carried over unchanged: CRU gating and the 24 MHz
# cclk_src_sdmmc select, the GPIO3_A pinmux, the DesignWare reset/FIFO
# sequence, the sample and drive phases, the PIO transfer path. It was brought
# up against the card; none of it is re-derived here.
#
# usb_dwc2.cpp is absent on purpose: the RK3506 gadget stack is a separate
# port, still to be carried over from the predecessor's src/usb/.

add_library (micro-os-plus-iii-devices-rk3506-interface INTERFACE)

target_include_directories (
  micro-os-plus-iii-devices-rk3506-interface
  INTERFACE "include" "fatfs" "soc/rk3506/include"
)

target_compile_definitions (
  micro-os-plus-iii-devices-rk3506-interface INTERFACE SD_BACKEND_DWMMC
)

target_sources (
  micro-os-plus-iii-devices-rk3506-interface
  INTERFACE soc/rk3506/src/sdmmc.cpp
            soc/rk3506/src/sd_dwmmc.cpp
            src/flatfs.cpp
            fatfs/ff.c
            fatfs/diskio_sd.cpp
            fatfs/fatfs_hw.cpp
)

add_library (micro-os-plus::devices-rk3506
             ALIAS micro-os-plus-iii-devices-rk3506-interface)

# -----------------------------------------------------------------------------
# BCM2837 silicon support, shared by the aarch32 and aarch64 projects.
# Separate target so a build takes it only when the silicon is a BCM2837.

add_library (micro-os-plus-iii-soc-bcm2837-interface INTERFACE)

target_include_directories (
  micro-os-plus-iii-soc-bcm2837-interface INTERFACE "soc/bcm2837/include"
)

target_sources (
  micro-os-plus-iii-soc-bcm2837-interface INTERFACE soc/bcm2837/src/mailbox.cpp
)

add_library (micro-os-plus::soc-bcm2837
             ALIAS micro-os-plus-iii-soc-bcm2837-interface)

# -----------------------------------------------------------------------------
message (STATUS "µOS++ III devices: micro-os-plus::devices micro-os-plus::devices-rk3506 micro-os-plus::soc-bcm2837")
