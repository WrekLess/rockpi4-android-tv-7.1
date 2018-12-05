# Build AOSP for ROCK960

## Build environment setup

Recommend build host is Ubuntu 16.04 64bit, for other hosts, refer official Android documents [Establishing a Build Environment](https://source.android.com/setup/build/initializing).


```shell
$ mkdir -p ~/bin
$ wget 'https://storage.googleapis.com/git-repo-downloads/repo' -P ~/bin
$ chmod +x ~/bin/repo
```

Android's source code primarily consists of Java, C++, and XML files. To compile the source code, you'll need to install OpenJDK 8, GNU C and C++ compilers, XML parsing libraries, ImageMagick, and several other related packages.


```shell
$ sudo apt-get update
$ sudo apt-get install openjdk-8-jdk android-tools-adb bc bison build-essential curl flex g++-multilib gcc-multilib gnupg gperf imagemagick lib32ncurses5-dev lib32readline-dev lib32z1-dev libesd0-dev liblz4-tool libncurses5-dev libsdl1.2-dev libssl-dev libwxgtk3.0-dev libxml2 libxml2-utils lzop pngcrush rsync schedtool squashfs-tools xsltproc yasm zip zlib1g-dev
```

Configure the JAVA environment

```shell
$ export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
$ export PATH=$JAVA_HOME/bin:$PATH
$ export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

## Download source code

```shell
$ mkdir rockpi4-android
$ cd rockpi4-android
```
Then run:

```shell
$ ~/bin/repo init -u https://github.com/radxa/rockpi4-android-tv-7.1.git -m rockchip_tv_nougat_release.xml
$ repo sync -j$(nproc) -c
```
It might take quite a bit of time to fetch the entire AOSP source code(around 86G)!

In China:
Download Repo
```shell
$ curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o repo
$ export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo/'
```

## Build u-boot

```shell
$ cd u-boot
$ make rk3399_box_defconfig
$ ./mkv8.sh
$ cd ..
```

The generated images are **rk3399_loader_v_xxx.bin** and **uboot.img**

## Building kernel

```shell
$ cd kernel
$ make rockchip_defconfig
$ make rk3399-rockpi-4b.img -j$(nproc)
$ cd ..
```

The generated images are **kernel.img** and **resource.img**:

- kernel.img, kernel with rkcrc checksum
- resource.img, contains dtb and boot logo, Rockchip format resource package

## Building AOSP

```shell
$ source build/envsetup.sh
$ lunch rk3399_box-userdebug
$ make -j$(nproc)
```

It takes a long time, take a break and wait...


## Generate update images

```shell
$ ln -s RKTools/linux/Linux_Pack_Firmware/rockdev/ rockdev
$ ./mkimage.sh
```

The generated images under rockdev/Image-rk3399_box are

    boot.img    MiniLoaderAll.bin  parameter.txt        pcba_whole_misc.img  resource.img  trust.img
    kernel.img  misc.img           pcba_small_misc.img  recovery.img         system.img    uboot.img

They can be flashed with fastboot or Rockchip upgrade_tool.

Pack all partitions into one image.

```shell
$ cd rockdev
$ ln -s Image-rk3399_box Image
$ ./mkupdate.sh
```

    Start to make update.img...
    Android Firmware Package Tool v1.62
    ------ PACKAGE ------
    Add file: ./package-file
    Add file: ./Image/MiniLoaderAll.bin
    Add file: ./Image/parameter.txt
    Add file: ./Image/trust.img
    Add file: ./Image/uboot.img
    Add file: ./Image/misc.img
    Add file: ./Image/resource.img
    Add file: ./Image/kernel.img
    Add file: ./Image/boot.img
    Add file: ./Image/recovery.img
    Add file: ./Image/system.img
    Add CRC...
    Make firmware OK!
    ------ OK ------
    ********RKImageMaker ver 1.63********
    Generating new image, please wait...
    Writing head info...
    Writing boot file...
    Writing firmware...
    Generating MD5 data...
    MD5 data generated successfully!
    New image generated successfully!
    Making update.img OK.

**update.img** is the packed image with all partitions.

Proceed to [Installation Instructions](../installation)
