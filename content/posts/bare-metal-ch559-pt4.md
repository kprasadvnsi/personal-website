+++
author = "Kali Prasad"
title = "CH559 Programming (Part 4): Using UART"
date = "2020-03-31"
description = "Bare Metal programming with CH559"
images = ["/ch559_uart_demo.png"]
tags = [
    "Bare Metal",
    "CH559",
    "8051",
    "MCS-51",
    "UART",
]
[image]
    url = "/ch559_uart_demo.png"
    width = 696
    height = 383
+++

![CH559 UART0 Demo](/ch559_uart_demo.png)

After toggling some I/O pins, the first thing you should get up and running on a new platform is UART. CH559 chip provides two full-duplex asynchronous serial ports: UART0 and UART1. UART0 is a standard MCS51 serial port, and UART1 is an enhanced asynchronous serial port with a lot of features that we talk about in future tutorials. 

We need the datasheet for register definitions. WCH only provides Datasheet in Chinese. I have translated the datasheet, you can find it [here](https://kprasadvnsi.github.io/CH559_Doc_English/). Complete source code is also available on this [GitHub Repo](https://github.com/kprasadvnsi/ch55x_bare_metal).

## UART0 Initialization

First, we need to select the UART mode by setting SM0 and SM1 bit in the SCON register. We are going to use Mode 1 for UART0.

{{< highlight c "linenos=table">}}
// SM0 & SM1: UART0 mode
// 00 - mode 0, shift Register, baud rate fixed at: Fsys/12
// 01 - mode 1, 8-bit UART,     baud rate = variable by timer1 or timer2 overflow rate
// 10 - mode 2, 9-bit UART,     baud rate fixed at: Fsys/128@SMOD=0, Fsys/32@SMOD=1
// 11 - mode 3, 9-bit UART,     baud rate = variable by timer1 or timer2 overflow rate
SM0 = 0;
SM1 = 1;

{{< /highlight >}}

Next, we need to select a clock source from either timer1 or timer2. We will choose timer1 as our clock source for UART0 by setting RCLK and TCLK bits in the T2CON register.

{{< highlight c "linenos=table">}}
// RCLK select UART0 receiving clock: 0=timer1 overflow pulse, 1=timer2 overflow pulse
// TCLK select UART0 transmittal clock: 0=timer1 overflow pulse, 1=timer2 overflow pulse
RCLK = 0;
TCLK = 0;
{{< /highlight >}}

We configure timer1 with 12MHz internal clock in mode 2 for baud rate 9600 by setting bT1_M1 and bT1_M0 bits in the TMOD register.

{{< highlight c "linenos=table">}}
// bT1_M1 & bT1_M0: timer1 mode
// 00: mode 0, 13-bit timer or counter by cascaded TH1 and lower 5 bits of TL1, the upper 3 bits of TL1 are ignored
// 01: mode 1, 16-bit timer or counter by cascaded TH1 and TL1
// 10: mode 2, TL1 operates as 8-bit timer or counter, and TH1 provide initial value for TL1 auto-reload
// 11: mode 3, stop timer1

TMOD = MASK_T1_MOD & (bT1_M1 | ~bT1_M0);
{{< /highlight >}}

CH559 provide different speed mode for UART0 baud rate generation for power saving purpose. We are going to use fast Mode by setting a SMOD bit in the PCON register. We also need to set bTMR_CLK and bT1_CLK in the T2MOD register to get a 12MHz clock.

CH559 provides several ways to generate the baud rate. The baud rate generation formula for different UART0, speeds are listed in the table below.

![Calculation formula of UART0 baud rate generated by T1](/uart0_baud_farmula_table.png)

We will choose the 2nd option in the list above and set TH1 register for 9600 baud rate.

{{< highlight c "linenos=table">}}
PCON |= SMOD;
T2MOD = T2MOD | bTMR_CLK | bT1_CLK;
TH1 = 256 - FREQ_SYS/16/UART0_BAUD;
{{< /highlight >}}

Now, we will start the timer1 by setting the TR1 bit. Set the transmit interrupt flag TI for UART0. and enable the UART0 by setting REN bit in the SCON register.

{{< highlight c "linenos=table">}}
TR1 = 1;
TI = 1;
REN = 1;
{{< /highlight >}}

## Sending Data to UART0

We can send data via UART0 by writing on the SBUF register.

{{< highlight c "linenos=table">}}
// Send one byte data to UART0
void uart0_write(uint8_t data) {
    while (!TI);
    TI = 0;
    SBUF = data & 0xFF;
}
{{< /highlight >}}

## Receiving Data from UART0

We can receive data from UART0 by reading the SBUF register.

{{< highlight c "linenos=table">}}
// Receive one byte data from UART0
uint8_t uart0_read() {
    while(!RI);
    RI = 0;
    return SBUF;
}
{{< /highlight >}}

## Using printf with UART0

Redirecting stdout is easy with SDCC

{{< highlight c "linenos=table">}}
/*
 * Redirect stdout to UART0
 */
int putchar(int c) {
    uart0_write(c);
    return 0;
}

/*
 * Redirect stdin to UART0
 */
int getchar() {
    return uart0_read();
}
{{< /highlight >}}

Now we can use printf() for debugging.