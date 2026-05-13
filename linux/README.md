Linux Kernel
------------

````
export CROSS_COMPILE=arm-linux-gnueabihf-
export ARCH=arm

git clone git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git
cd linux-stable
# update the branch to an updated version
git checkout v5.10.255


# rk3188 strictly related
# copy the correct defconfig from this repository to arch/arm/configs/
make radxa_rock_defconfig

# compilation with device tree blob
make -j4 zImage dtbs

# kernel modules
make -j4 modules

sudo make INSTALL_MOD_PATH=/ABS_PATH_TO/linuxroot/ modules_install
````

Now you just have to copy the zImage and the dtb files in the boot partition of the SD card, and create the loader/entries path for boot.conf.
In the next section we'll create the partitions on SD card.


Rootfs
------

````
apt install binfmt-support debootstrap losetup

export TARGET_DIR=rootfs

sudo debootstrap --arch=armhf --foreign buster $TARGET_DIR

sudo chroot $TARGET_DIR
export LANG=C
/debootstrap/debootstrap --second-stage

apt update
apt install locales dialog openssh-server ntpdate htop net-tools sudo resolvconf udev ifupdown libpam-systemd i2c-tools
dpkg-reconfigure locales

# add users
adduser rock

# change root passwd
passwd

# customize networking in /etc/network/interfaces
# auto enx00133b8514df
# iface enx00133b8514df inet dhcp

# put some net informations
export HOSTNAME=radxa
echo $HOSTNAME > /etc/hostname
echo 127.0.0.1	localhost > /etc/hosts
echo 127.0.1.1	$HOSTNAME >> /etc/hosts
hostname $HOSTNAME

# finished
exit
````

Create an image where to store the brand new rootfs (700MB)

````
dd if=/dev/zero of=rootfs.img bs=700K count=1024
losetup /dev/loop15 rootfs.img 
mkfs.ext4 /dev/loop15

# mount  and copy rootfs
mkdir mnt
mount /dev/loop15 mnt
rsync -aHAX rootfs/* mnt/

# umount
sync
umount mnt

# do a fs check, a resize 
gparted /dev/sdg

# write partition to sd card
dd if=rootfs.img of=/dev/sdg2 bs=4K conv=sync status=progress
````
