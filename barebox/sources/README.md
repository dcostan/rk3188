Prepare environment
-------------------

````
apt update
apt install lzop libusb-1.0-0-dev git flex bison build-essential gcc-arm-linux-gnueabihf lzop libncurses5-dev libssl-dev bc wget rsync
````

BareBox bootloader
------------------

````
git clone https://github.com/barebox/barebox.git
cd barebox

export CROSS_COMPILE=arm-linux-gnueabihf-
export ARCH=arm

make rockchip_v7a_defconfig

make -j4

# your image in 
ls images/images/barebox-radxa-rock.img 
````

Barebox SD card

As documented here: https://www.barebox.org/doc/latest/boards/rockchip.html

- Make 2 partitions on SD for boot and root filesystems.
    - make a DOS partition table, each partition would be ext4 with labels "boot" and "linuxroot"
- Checkout and compile https://github.com/apxii/rkboottools or its backup in this repository
    - `git clone https://github.com/apxii/rkboottools.git && cd rkboottools`
    - `make`
    
- Get some RK3188 bootloader from https://github.com/neo-technologies/rockchip-bootloader
- Run “rk-splitboot RK3188Loader(L)_V2.19.bin” command. (for example). You will get FlashData file with others. It’s a DRAM setup blob.
  Otherwise it can be borrowed from RK U-boot sources from https://github.com/linux-rockchip/u-boot-rockchip/blob/u-boot-rk3188/tools/rk_tools/3188_LPDDR2_300MHz_DDR3_300MHz_20130830.bin
  - `./rk-splitboot RK3188Loader(L)_V2.19.bin` -> will give use FlashData and FlashBoot in the working directory
- Run `./rk-makebootable FlashData barebox-radxa-rock.img rrboot.bin`
- `sudo dd if=rrboot.bin of=</dev/sdcard> bs=$((0x200)) seek=$((0x40))`

SD card is ready
