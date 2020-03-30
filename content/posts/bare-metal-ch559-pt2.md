+++
author = "Kali Prasad"
title = "	"
date = "2020-03-29"
description = "Bare Metal programming with CH559"
images = ["/make_dir_struct.png"]
tags = [
    "Bare Metal",
    "CH559",
    "8051",
    "SDCC",
    "MCS-51",
    "GNU-make",
]
[image]
    url = "/make_dir_struct.png"
    width = 746
    height = 396
+++

In this part, we are going to organize and clean our LED blink example that we wrote in the [Part-1](/posts/bare-metal-ch559-pt1/) and automate the building process using [GNU Make](https://www.gnu.org/software/make/). Complete source code is also available on this [GitHub Repo](https://github.com/kprasadvnsi/ch55x_bare_metal).

We start by creating a directory named **ch559**. Inside directory **ch559** will make another directory **lib** and **examples**. The final directory structure will look similar to the image shown below.

![Directory structure for make project](/make_dir_struct.png)

Under the **lib** directory, we will create a header file `ch559.h` in which will put all the register definitions using SFR and SBIT macros.

`--------------------------- ch559.h ---------------------------`
{{< highlight c "linenos=table">}}
#ifndef __CH559_H__
#define __CH559_H__

#include <compiler.h>

/*  Port Registers  */
SFR(P1,	0x90);	// port 1 input & output
   SBIT(P1_7,	0x90, 7);
   SBIT(P1_6,	0x90, 6);
   SBIT(P1_5,	0x90, 5);
   SBIT(P1_4,	0x90, 4);
   SBIT(P1_3,	0x90, 3);
   SBIT(P1_2,	0x90, 2);
   SBIT(P1_1,	0x90, 1);
   SBIT(P1_0,	0x90, 0);

SFR(P1_DIR,	0xBA);	// port 1 direction
SFR(PORT_CFG,	0xC6);	// port 0/1/2/3 config

#endif  // __CH559_H__
{{< /highlight >}}

Now we will create `Makefile.include` file under the **examples** directory and put the make scripts that are common for all examples.

`----------------------- Makefile.include ----------------------`
{{< highlight make "linenos=table">}}
#######################################################
# toolchain
CC = sdcc
OBJCOPY = objcopy
PACK_HEX = packihx
WCHISP = wchisptool
#######################################################

ifndef FREQ_SYS
FREQ_SYS = 16000000
endif

ifndef XRAM_SIZE
XRAM_SIZE = 0x1800
endif

ifndef XRAM_LOC
XRAM_LOC = 0x0000
endif

ifndef CODE_SIZE
CODE_SIZE = 0xF000
endif

ROOT_DIR := $(dir $(abspath $(lastword $(MAKEFILE_LIST))))

CFLAGS := -V -mmcs51 --model-small \
	--xram-size $(XRAM_SIZE) --xram-loc $(XRAM_LOC) \
	--code-size $(CODE_SIZE) \
	-I$(ROOT_DIR)../lib -DFREQ_SYS=$(FREQ_SYS) \
	$(EXTRA_FLAGS)

LFLAGS := $(CFLAGS)

RELS := $(C_FILES:.c=.rel)

print-%  : ; @echo $* = $($*)

%.rel : %.c
	$(CC) -c $(CFLAGS) $<


$(TARGET).ihx: $(RELS)
	$(CC) $(notdir $(RELS)) $(LFLAGS) -o $(TARGET).ihx

$(TARGET).hex: $(TARGET).ihx
	$(PACK_HEX) $(TARGET).ihx > $(TARGET).hex

$(TARGET).bin: $(TARGET).ihx
	$(OBJCOPY) -I ihex -O binary $(TARGET).ihx $(TARGET).bin

flash: $(TARGET).bin pre-flash
	$(WCHISP) -f $(TARGET).bin -g

.DEFAULT_GOAL := all
all: $(TARGET).bin $(TARGET).hex

clean:
	rm -f \
	$(notdir $(RELS:.rel=.asm)) \
	$(notdir $(RELS:.rel=.lst)) \
	$(notdir $(RELS:.rel=.mem)) \
	$(notdir $(RELS:.rel=.rel)) \
	$(notdir $(RELS:.rel=.rst)) \
	$(notdir $(RELS:.rel=.sym)) \
	$(notdir $(RELS:.rel=.adb)) \
	$(TARGET).lk \
	$(TARGET).map \
	$(TARGET).mem \
	$(TARGET).ihx \
	$(TARGET).hex \
	$(TARGET).bin
{{< /highlight >}}


We will put our LED blink example under the **examples** directory. Create directory **blink** under examples. Now make two files `blink.c` and `Makefile`.

`--------------------------- blink.c ---------------------------`
{{< highlight c "linenos=table">}}
// Blink a LED connected to pin 1.6

#include <stdint.h>
#include <ch559.h>

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
		P1_6 = !P1_6;
	}
}
{{< /highlight >}}

`-------------------------- Makefile ---------------------------`
{{< highlight make "linenos=table">}}
TARGET = blink

C_FILES = \
	blink.c

include ../Makefile.include
{{< /highlight >}}

Its time to build our LED Blink example. Goto the **blink** directory and execute the command make. This will compile our example and produce a binary file for programming the CH559 board.