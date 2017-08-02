# Mitosis Keyboard Firmware
Firmware for Nordic MCUs used in the Mitosis Keyboard, contains precompiled .hex files, as well as sources buildable with the Nordic SDK


## IAR Support

Original build used blue Waveshare Core51822 (B)
modules ([$6.99](http://www.waveshare.com/core51822-b.htm)), but aliexpress has slightly smaller black YJ-14015
modules that are also much cheaper (about [$3.50](https://www.aliexpress.com/item/BLE4-0-Bluetooth-2-4GHz-Wireless-Module-NRF51822-Board-Core51822-B/32633417101.html),
you can also find them on [ebay](http://www.ebay.com/itm/BLE4-0-Bluetooth-2-4GHz-Wireless-Module-NRF51822-Board-Core51822-B-/282575577879)).
It seems like precompiled firmware that works on Core51822 (B), doesn't work on YJ-14015 at all.
Not really sure why it doesn't work after GCC toolchain, but you can build a working firmware in IAR, using provided project.
It also needs defined `CLOCK_ENABLED` and `RTC0_ENABLED` in the receiver firmware (`nrf_drv_config.h`),
otherwise it doesn't compile. IAR 7.50.2 compatible project is based upon `peripheral/blinky`
sample from nRF5_SDK_11. You may also use precompiled firmware from the `precompiled_iar` folder.
This version also makes use of the keyswitch board LED (it blinks two times on powering up to indicate that firmware works).


## Install dependencies

Tested on Ubuntu 16.04.2, but should be able to find alternatives on all distros. 

```
sudo apt install openocd gcc-arm-none-eabi
```

## Download Nordic SDK

Nordic does not allow redistribution of their SDK or components, so download and extract from their site:

https://www.nordicsemi.com/eng/nordic/Products/nRF5-SDK/nRF5-SDK-v12-zip/54291

Firmware written and tested with version 11

```
unzip nRF5_SDK_11.0.0_89a8197.zip  -d nRF5_SDK_11
cd nRF5_SDK_11
```

## Toolchain set-up

A cofiguration file that came with the SDK needs to be changed. Assuming you installed gcc-arm with apt, the compiler root path needs to be changed in /components/toolchain/gcc/Makefile.posix, the line:
```
GNU_INSTALL_ROOT := /usr/local/gcc-arm-none-eabi-4_9-2015q1
```
Replaced with:
```
GNU_INSTALL_ROOT := /usr/
```

## Clone repository
Inside nRF5_SDK_11/
```
git clone https://github.com/reversebias/mitosis
```

## Install udev rules
```
sudo cp mitosis/49-stlinkv2.rules /etc/udev/rules.d/
```
Plug in, or replug in the programmer after this.

## OpenOCD server
The programming header on the side of the keyboard, from top to bottom:
```
SWCLK
SWDIO
GND
3.3V
```
It's best to remove the battery during long sessions of debugging, as charging non-rechargeable lithium batteries isn't recommended.

Launch a debugging session with:
```
openocd -f mitosis/nrf-stlinkv2.cfg
```
Should give you an output ending in:
```
Info : nrf51.cpu: hardware has 4 breakpoints, 2 watchpoints
```
Otherwise you likely have a loose or wrong wire.


## Manual programming
From the factory, these chips need to be erased:
```
echo reset halt | telnet localhost 4444
echo nrf51 mass_erase | telnet localhost 4444
```
From there, the precompiled binaries can be loaded:
```
echo reset halt | telnet localhost 4444
echo flash write_image `readlink -f precompiled-basic-left.hex` | telnet localhost 4444
echo reset | telnet localhost 4444
```

## Automatic make and programming scripts
To use the automatic build scripts:
```
cd mitosis/mitosis-keyboard-basic
./program.sh
```
An openocd session should be running in another terminal, as this script sends commands to it.



















