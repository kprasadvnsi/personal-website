+++
author = "Kali Prasad"
title = "CH559 Programming (Part 1): Setup and blinky"
date = "2020-03-24"
description = "Bare Metal programming with CH559"
images = ["/ch559_dev_board.jpg"]
videos = ["https://www.youtube.com/watch?v=s80IY7E03bQ"]
tags = [
    "Bare Metal",
    "CH559",
    "8051",
    "SDCC",
    "MCS-51",
]
+++

The CH559 is a cheap 8051 based microcontroller with USB Host/Device capability and comes with plenty of Peripherals. CH559 is the most advance MCU in the CH55x series, the reason I am starting with the most advanced one because it is a Flash variant unlike other MCUs in the CH55x series.

In this tutorial, we will learn the bare minimum to get started without using any vendor-supplied libraries. I will start by getting to know the hardware and toolchain, later will write a simple LED blink example.

## Hardware

There are several ways to start working with CH559. I find Electrodragon's [CH559 USB MCU Mini DEV Board](https://www.electrodragon.com/product/ch559-mini-dev-board-ch55x-series/) easiest to get and cheap. It breakout all the pins and comes with 4 LEDs for demo.

![CH559 dev board](/ch559_dev_board.jpg)


## Setting up the toolchain

Before we do anything, we need to find a compiler and a way to program this MCU. Unfortunately, a GCC compiler is not available for 8051 microcontrollers. Few commercial compilers available for these processors, some of them have a free version with a code size limit, but all of them are Windows only. Luckily, **SDCC** support 8051 based MCUs and its Open Source.

To program this MCU, we don't need any extra hardware as it comes with a USB Bootloader built-in. I found Rgwan's **LibreCH551** pretty straight forward to use. Now its time to download all the necessary tools:

1. [SDCC](https://sourceforge.net/projects/sdcc/files/sdcc-linux-amd64/4.0.0/sdcc-4.0.0-amd64-unknown-linux2.5.tar.bz2/download)
2. [LibreCH551](https://github.com/rgwan/librech551)

Extract SDCC under `~/local/sdcc` and add the following lines to your `.bashrc` file (Replace "\<username\>" with your "username").

{{< highlight bash >}}
export PATH=$PATH:/home/<username>/local/sdcc/bin
{{< /highlight >}}

If you have done the above steps correctly, you should be able to run `sdcc --version`.

For the **LibreCH551** have to make it from the source. Clone the Repo and go under `librech551/usbisp` and build the tool using **make** command. You will find the **wchisptool** executable in the same folder.

**Note.** You may need to install the libusb-1.0-dev package if not already installed in your system.

Now copy **wchisptool** binary to `~/local/librech551/bin` and add the following lines to your `.bashrc` file (Replace "\<username\>" with your "username").

{{< highlight bash >}}
export PATH=$PATH:/home/<username>/local/librech551/bin
{{< /highlight >}}

We need to set up udev rules for USB bootloader so that we don't have to run **wchisptool** with root privilege. Create a file `/etc/udev/rules.d/99-ch559.rules` and add the following lines to the file.

```
# CH559
ATTRS{idVendor}=="4348", ATTRS{idProduct}=="55e0", MODE="0666"
```

Now run `udevadm control --reload-rules && udevadm trigger` as a root to restart **udev** service.

## Everything is memory-mapped

You can access all Peripherals I/O and registers using memory addressing. SDCC conveniently provides two Macro `SFR()` and `SBIT()` to access the SFR(Special Function Register) and Bit addressable memory.

We need Datasheet for this MCU to know its pinout and register map. Unfortunately, WCH only provides Datasheet in Chinese. I have translated the Datasheet, you can find it [here](https://kprasadvnsi.github.io/CH559_Doc_English/).

Let's write our first example. Create a file `blink.c` and add the following code.

{{< highlight c >}}
#include <compiler.h>
#include <stdint.h>

SFR(PORT_CFG,	0xC6); // Port Configuration Register
SFR(P1_DIR,	0xBA);	// P1 port direction control register
SFR(P1,	0x90);	// P1 port input and output register

SBIT(LED6, 0x90, 6); // accessing pin 1.6

static inline void delay() {
    uint32_t i;
    for (i = 0; i < (120000UL); i++){}
        __asm__("nop");
}

void main() {
	PORT_CFG = 0b00101101;
    P1_DIR = 0b11110000;
	P1 = 0x00;

	while (1) {
		delay();
		LED6 = !LED6;
	}
}
{{< /highlight >}}

Save the `blink.c` file and compile it using the following command.

{{< highlight bash >}}
sdcc -V -mmcs51 --model-small --xram-size 0x1800 --xram-loc 0x0000 --code-size 0xF000 blink.c
{{< /highlight >}}

We need to convert the blinkihx file to blink.bin for wchisptool to program the CH559.

{{< highlight bash >}}
objcopy -I ihex -O binary blink.ihx blink.bin
{{< /highlight >}}

Now its time to program the CH559 dev board with our blink example. Press the PROG button while plugging the USB cable to your PC then release the PROG button to put CH559 into Bootloader mode. Use the following command to program the MCU.

{{< highlight bash >}}
wchisptool -f blink.bin -g
{{< /highlight >}}

Congratulations! We've just written our first LED blink example.

{{< youtubelite id="s80IY7E03bQ" title="CH559 LED Blink demo">}}
