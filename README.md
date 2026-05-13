# Mainline Linux for Radxa Rock

This document describes the procedure used to build a working system image based on **mainline Linux** for a Radxa Rock-compatible board. The setup was tested on a clone of the **Rikomagic MK802IV**.

## Overview

Booting from the SD card is handled by **Barebox**. Barebox must be written into the very first bytes of the SD card itself, and it is **not** stored in NAND.

In practice, all custom kernels tested so far, including `linux-rockchip`, never worked correctly on this hardware. For that reason, using **mainline Linux only** is essential.

## SD Card Preparation

The SD card must be partitioned before installing the system. Create two partitions, both formatted as **ext4**:

- The first partition with label `boot`
- The second partition with label `linuxroot`

It is important to leave some free space at the beginning of the card, before the first partition, so that Barebox can be written there without overlapping the filesystem data.

## Barebox and Linux Build

**Barebox** can be built using the standard configuration if the submitted pull request has been merged upstream. Otherwise, the custom `defconfig` and Device Tree Source (DTS) provided in this repository must be used.

For **Linux**, using the custom `defconfig` is mandatory. Additionally, enabling CPU frequency scaling requires a custom Device Tree, because testing showed that only two voltage-frequency pairs from the OPP tables are currently working.

## Tested Hardware

This procedure was tested on a clone board of the **Rikomagic MK802IV**.

The serial interface is exposed on a PCB pin marked `TX`. When using an Arduino as a USB-to-TTL adapter:

- Connect the board `TX` pin to the Arduino `TX`
- Keep the Arduino reset grounded

## Workflow

1. Prepare the SD card and leave some unallocated space at the beginning.
2. Create two ext4 partitions labeled `boot` and `linuxroot`.
4. Build Barebox using the `rockchip_v7a_defconfig` and, if still needed, the custom DTS from this repository.
3. Write Barebox into the initial bytes of the SD card.
5. Build **mainline Linux** using the `radxa_rock_defconfig` from this repository.
6. Install the resulting boot and root filesystem contents onto the SD card partitions.
7. Use the serial console for debugging and first boot validation.
