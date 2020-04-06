+++
author = "Kali Prasad"
title = "CH559 Programming (Part 5): Working with GPIO"
date = "2020-04-02"
description = "Bare Metal programming with CH559"
images = ["/adv_blink.png"]
tags = [
    "Bare Metal",
    "CH559",
    "8051",
    "MCS-51",
    "UART",
]
[image]
    url = "/adv_blink.png"
    width = 800
    height = 446
+++

![Advanced Blinky](/adv_blink.png)
In this part, we will go through GPIO in detail. we had briefly touched upon GPIOs in our [first tutorial](/posts/bare-metal-ch559-pt1/). Now we will rewrite the LED blink example in a more structured way later in this tutorial.

CH559 has 45 I/O pins, some of them have alternative functions. All the GPIOs are group into ports of up to 8 pins. P0 to P3 all registers for the GPIO are bit addressable. There are a P4 and P5 in Ch559 but will talk about them in future tutorials.

## GPIO Registers

There is mainly one power configuration register(PORT_CFG) for P0~P3 and three configuration registers for each port. Let's start with the PORT_CFG register, what it does and how we can configure it.

Here is the snippet from [Datasheet](https://kprasadvnsi.github.io/CH559_Doc_English/docs/10-gpio/).

![PORT_CFG register](/port_cfg.png)

The register bits are divided into two parts. lower 4 bits(bPn_OC) are for configuring output mode and upper 4 bits(bPn_DRV) are for configuring output current for each port. If you are wondering what Push-Pull and Open-Drain are then you can check this [article](https://open4tech.com/open-drain-output-vs-push-pull-output/).

Next are the Pn, Pn_DIR, and P0=n_PU registers. Again, from the [Datasheet](https://kprasadvnsi.github.io/CH559_Doc_English/docs/10-gpio/), we can see what these registers do. All registers and bits in this section are expressed in a common format: the lowercase "n" represents the serial number of the port (n = 0, 1, 2, 3), and the lowercase "x" represents the serial number of the bit (x = 0, 1, 2, 3, 4).

![Port Registers](/port_registers.png)

The configuration of the Pn port is implemented by the combination of the bit bPn_OC in the PORT_CFG, the port direction control register Pn_DIR, and the port pull-up enable register Pn_PU, as follows.

![Port config table](/port_congif_table.png)

## Making an Advanced Blinky

We will make a LED blink example that has 8 LEDs and will drive it using port P0 of CH559. Copy the LED blink example from the [Part-2](/posts/bare-metal-ch559-pt2/) tutorial and rename it to advance_blink. Open the main.c file and make the following changes.

{{< highlight c "linenos=table">}}
// Blink LEDs connected to port P0

#include <stdint.h>
#include <ch559.h>

#define MASK_P0_OC 0x0F

static inline void delay() {
    uint32_t i;
    for (i = 0; i < (5 * 12000UL); i++){}
        __asm__("nop");
}

void run_led () {
	uint8_t i;
	for(i=0; i<8; i++) {
		P0 = (1 << i);
		delay();
	}
}

void main() {

	PORT_CFG |= bP0_DRV | (MASK_P0_OC & ~bP0_OC); // Set port P0 driving current and output mode.
    P0_DIR = 0xFF; // Set all bits to output mode

	// Turning all bits to high
	P0 = 0xFF;
	delay();
	delay();
	delay();
	delay();


	// Turning all bits to low
	P0 = 0x00;
	delay();
	delay();
	delay();
	delay();

	// Turning alternate bits to high
	P0 = 0xAA;
	delay();
	delay();
	delay();
	delay();

	// Turning upper half bits to high
	P0 = 0xF0;
	delay();
	delay();
	delay();
	delay();

	// Turning lower half bits to high
	P0 = 0x0F;
	delay();
	delay();
	delay();
	delay();

	while (1) {
		run_led();
	}
}
{{< /highlight >}}

Congratulations! We've just written yet another LED blink example.

{{< youtubelite id="sj0Br56J-HU" title="CH559 LED sequence demo for port P0">}}