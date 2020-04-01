+++
author = "Kali Prasad"
title = "CH559 Programming (Part 3): Memory Organization"
date = "2020-03-30"
description = "Bare Metal programming with CH559"
images = ["/mem_struct.jpg"]
tags = [
    "Bare Metal",
    "CH559",
    "8051",
    "MCS-51",
]
[image]
    url = "/mem_struct.jpg"
    width = 800
    height = 666
+++

In this part, we are going to learn about the CH559 memory structure and how the various part of memory will be accessed. The CH559 is an 8051 based microcontroller so the basic methods of accessing memory are also applicable here. 

The CH559 has four distinct types of memory – iRAM (internal RAM), SFR (special function registers), ROM (program memory), and xRAM (external data memory).

![Memory Organization](/mem_struct.jpg)

## Internal RAM

Internal RAM (iRAM) has an 8-bit address space, using addresses 0x00 through 0xFF. iRAM from 0x00 to 0x7F can be accessed directly, using an 8-bit absolute address that is part of the instruction. Alternatively, iRAM can be accessed indirectly.

The original 8051 has only 128 bytes of iRAM. The 8052 added iRAM from 0x80 to 0xFF, which can only be accessed indirectly. Direct access to this address range goes to the special function registers. CH559 has a full 256 bytes of iRAM.

Memory addresses from 0x00 to 0x7F are divided into working registers (organized as register banks), Bit-addressable area and general-purpose RAM. Memory addresses 0x00 to 0x1F consists of 32 working registers that are organized as four banks with 8 registers in each bank. The 16 bytes (128 bits) at iRAM locations 0x20–0x2F are bit-addressable.


## Special function registers

Special function registers (SFR) are located in the same address space as iRAM, at addresses 0x80 to 0xFF, and are accessed directly using the same instructions as for the lower half of iRAM. They cannot be accessed indirectly. 16 of the SFRs (those whose addresses are multiples of 8) are also bit-addressable. 

##  Program memory

Program memory (ROM) is up to 64 KB of read-only memory, starting at address 0x0000 in a separate address space. It may be on- or off-chip, depending on the particular model of the chip being used.  

CH559 has on-chip 64 KB of flash-based ROM and can be divided into 60 KB program storage and 1 KB data storage area and 3 KB boot code BootLoader/ISP program area.

## External data memory

External data memory (xRAM) is a third address space, also starting at address 0, and allowing 16 bits of address space. It can also be on- or off-chip. what makes it "external" is that it must be accessed using the MOVX (move external) instruction. Many variants of the 8051 include the standard 256 bytes of iRAM plus a few KB of XRAM on the chip. 

CH559 has 6 KB on-chip xRAM that can be used for large temporary data storage and DMA direct memory access. support for the external expansion of 32 KB external SRAM.

The external data storage space is 64 KB in total, as shown in Figure above, and partially used for 6 KB on-chip expansion of xRAM and xSFR. Except for the two reserved areas, the remaining 0x4000 to 0xFFFF address range is the external bus area.