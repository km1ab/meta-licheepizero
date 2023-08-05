# meta-licheepizero

## Instruction how to build an image for Lichee Pi Zero and Lichee Pi Zero Dock in Yocto

### Products:

![Schematic](Lichee_Pi_Zero.png) <br>
Lichee Pi Zero Version <br>
<br>
![Schematic](Lichee_Pi_Zero_Dock.jpg) <br>
Lichee Pi Zero Dock Version <br>
<br>

## General Note:
Assumed that Linux Ubuntu is installed

## List of tested elements

Ethernet <br>
Led  <br>
Headphone <br>

## List of not tested elements

WiFi <br>
Lcd <br>
Touchscreen(aligned driver for kernel 5.15) <br>
Backlight for Lcd <br>
Microphone <br>
Bluetooth - appears during system boot up <br>

## How to build an images

1. First make sure to following packages are installed in system

    ```sh
    sudo apt-get install gawk wget git-core diffstat unzip \
    texinfo gcc-multilib build-essential chrpath socat libsdl1.2-dev \
    xterm libmpc-dev libgmp3-dev
    ```
    **Note:**
    More informations can be found on Yocto reference manual.

2. Download necessary Yocto packaged listed below. Be sure to be in root of home folder.
    ```sh
    mkdir yocto
    cd yocto 
    mkdir build 
    git clone git://git.yoctoproject.org/poky --depth 1 -b kirkstone 
    cd poky 
    git clone git://git.openembedded.org/meta-openembedded --depth 1 -b kirkstone 
    git clone https://github.com/meta-qt5/meta-qt5.git --depth 1 -b kirkstone 
    git clone https://github.com/km1ab/meta-licheepizero.git --depth 1 -b kirkstone-base 
    ```

3. Select directory to build Linux

    - Zero version 
    ```sh
	source oe-init-build-env ~/yocto/build/licheepizero 
    ```
    - Zero Dock version 
    ```sh
	source oe-init-build-env ~/yocto/build/licheepizero-dock 
    ```

1. Modify bblayers.conf(located in ~/yocto/build/licheepizero/conf(or licheepizero-dock/conf))

    command execute:
    ``` sh
    find ../../poky/ -type d -name meta* | \
    grep -v "perl\|webserver\|gnome\|recipes\|dynamic\|openembedded$\|openembedded\/meta$\|initramfs\|filesystems\|xfce\|selftest\|skeleton" | \
    xargs bitbake-layers add-layer
    ```

    or direct edit:
    ```
        BBLAYERS ?= " \
        ${HOME}/yocto/poky/meta \
        ${HOME}/yocto/poky/meta-poky \
        ${HOME}/yocto/poky/meta-openembedded/meta-oe \
        ${HOME}/yocto/poky/meta-openembedded/meta-networking \
        ${HOME}/yocto/poky/meta-openembedded/meta-python \
        ${HOME}/yocto/poky/meta-openembedded/meta-multimedia \
        ${HOME}/yocto/poky/meta-qt5 \
        ${HOME}/yocto/poky/meta-licheepizero \
        "
    ```
    **Note:** Please adapt PATH of conf/bblayers.conf if necessary. <br>

1. Modify local.conf(located in ~/yocto/build/licheepizero/conf(or licheepizero-dock/conf)) file

    - modify line with "MACHINE ??" to add "licheepizero-dock" or "licheepizero"
      
    - add at the end following records(maybe MUST!!)
    ```
    RM_OLD_IMAGE = "1" 
    INHERIT += "rm_work" 
    MACHINEOVERRIDES .= ":use-mailine-graphics" 
    LICENSE_FLAGS_ACCEPTED = "commercial" 
    ```

    - Option (thread and core setting; Set up for your environment.)
    ```
    BB_NUMBER_THREADS = '8' 
    PARALLEL_MAKE = '-j 4' 
    ```

    **Note:** Please adapt rest of conf/local.conf parameters if necessary. <br>

2. Build objects
   - console image
   ```
   bitbake console-image
   ```

   - qt5 image
   ```
   bitbake qt5-image
   ```

   - qt5 toolchain sdk
   ```
   bitbake meta-toolchain-qt5
   ```

3. Build time
    Build times for each recipe are as follows
    - console image ---> 2.5 hours
    - qt5 image ---> 
    - qt5 toolchain sdk ---> 

    Build times are approximate results for the following machine performance.
    - CPU : Intel Core i7-7700 CPU @ 3.60GHz × 4 (4cores allocated in VMware)
    - Memory : 32GB (20GB allocated in VMware)
    - Strage : SSD 1TB (200GB allocated in VMware)
    - OS : Ubuntu 20.04 (Running in VMware)
    - VMware : Virtual Box (6.1.28)

1. After compilation images appears in

    Zero version <br>
	*~/yocto/tmp/deploy/images/licheepizero* <br>
    Zero Dock version <br>
	*~/yocto/tmp/deploy/images/licheepizero-dock* <br>

2. Insert SD CARD into dedicated CARD slot and issue following command to write an image

    **Note:** <br>
    Be 100% sure to provide a valid device name (of=/dev/**sde**). Wrong name "/dev/sde" dameage Your system file ! <br> <br>
        Zero version <br>
    	***sudo dd if=~/yocto/tmp/deploy/images/licheepizero/qt5-image-licheepizero.sunxi-sdimg of=/dev/sde bs=1024*** <br>
    	Zero Dock verison <br>
    	***sudo dd if=~/yocto/tmp/deploy/images/licheepizero-dock/qt5-image-licheepizero-dock.sunxi-sdimg of=/dev/sde bs=1024*** <br>

# Audio<br>
To play mp3 or ogg files type <br>
	***mpv ogg-file-name.ogg*** <br>
	***mpv mp3-file-name.mp3*** <br>
	***cvlc mp3-file-name.mp3*** <br>
To change volume <br>
	***amixer set Headphone 10%+*** <br>
	***amixer set Headphone 10%-*** <br>
or just <br>
	***amixer set Headphone 10%*** <br>
<br>
Amixer available options <br>
	***amixer scontrols*** <br>
<br>
Microphone on <br>
	***amixer -c 0 cset numid=12 2*** <br>
<br>
Record some voice from microphone <br>
	***arecord -D hw:0,0 -d 3 -f S16_LE -r 16000 tmp.wav*** <br>
<br>
To get sound device info <br>
	***ls /dev/snd*** <br>

<br>
# Limitation

	- sunxi_mali is probably not working
	- rootfs-resize not working (SD CARD size can be resized manualy)
	- no wiringpi or similar library to driver GPIO in C code
	- discover problem when WiFi connected to access point (probably some drivers issues), nevertheless WiFi works
