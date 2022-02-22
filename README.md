# Radxa Zero SPI Display Driver Install

credits to: [HejinYo](https://github.com/HejinYo) for sharing the initial steps that can be found [here](https://github.com/HejinYo/ss/blob/master/doc/radxa/radxa-zero-spi-led.md).
<br/>

I just made some updates, and added step to compile kernel for the displays
<br/>

It should be able to support most of the available display drivers in the kernel. (Check the kernel directory for the available source) <br/>
We just need to modify compile and update into Radxa Zero.

## Dependencies
```bash
# install dtc
sudo apt-get install device-tree-compiler

# optional / useful to test display
# framebuffer imageviewer
sudo apt-get install fbi
```

## Create device tree files
```bash
# navigate to device tree path
# depends on your kernel version
cd /boot/dtbs/5.10.xx-x-amlogic-xxxxxxxxxxxxx/amlogic/overlay

# create new device tree file and 
# copy the contents of meson-g12a-spi-lcd.dts
# then configure pin assignment
sudo nano meson-g12a-spi-lcd.dts

# pin assignment, check values for A and B
<pin> = <0x2b A B>

# to get value of A, for example your pin is 36
# goto https://wiki.radxa.com/Zero/hardware/gpio
# get the ping number (Function 1 pin),
# ex. pin 36 -> GPIOH_8, then
# goto https://github.com/radxa/kernel/blob/linux-5.10.y-radxa-zero/include/dt-bindings/gpio/meson-g12a-gpio.h

# get the equivalent pin defined, for GPIOH_8, its 8, set it for A
# to get value of B, if it is initialized as HIGH, set to 1
# if initialized as LOW, set to 0

# save the dts file and compile to dtbo
# there may be some warnings, but its fine
sudo dtc -@ -I dts -O dtb -o meson-g12a-spi-lcd.dtbo meson-g12a-spi-lcd.dts

# update /boot/uEnv.txt
# add the new overlay to the overlays
# add the params
overlays=meson-g12a-spi-lcd
param_spidev_spi_bus=1
param_spidev_max_freq=10000000

# Compile the kernel, go to kernel-linux-5.10.y-radxa-zero/drivers/staging/fbtft/ directory
# if your display is st7789, you should take that .ko file
# be sure to change xxxx's to your own path
make && sudo cp {fb_st7789v.ko,fbtft.ko} /usr/lib/modules/5.10.xx-x-amlogic-xxxxxxxxxxxxx/kernel/drivers/staging/fbtft/


# lastly, reboot
sudo reboot now

# Connect the pins accordingly

# To verify the kernel is running
dmesg | grep fbtft

# to test display, get a sample image
sudo fbi -d /dev/fb1 -T 1 -noverbose -a <sample_image>