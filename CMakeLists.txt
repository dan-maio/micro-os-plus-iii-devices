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
#   micro-os-plus::devices        drivers, ISA-neutral
#   micro-os-plus::soc-bcm2837    BCM2837 silicon support
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
message (STATUS "µOS++ III devices: micro-os-plus::devices micro-os-plus::soc-bcm2837")
