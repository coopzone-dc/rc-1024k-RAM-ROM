This project was based on this original work:
https://github.com/PickledDog/rc-1024k
Many thanks the the author for all the hard work.

The original electronics design remain the same, this version has a new pcb layout. The main difference being it's now RCBUS instead of RC2014.

The original used KiCad, this version was created using EasyEDA and is pubished here:

https://oshwlab.com/mycoopzone/rcbus-512k-ram-512k-rom


# rc-1024k
 512K+512K RAM/ROM card for RCBUS
![Assembled](/img/assembled.png)

## Overview
This RCBUS-compatible board provides RAM and flash ROM, software-pageable in 32k pages. 16 ROM and 16 RAM pages may be paged into the lower 32k of Z80 memory. This paging scheme is compatible with Wayne Warthen's [RomWBW](https://github.com/wwarthen/RomWBW).

## Memory details
For ROM, a SST39SF040 512k x 8 flash chip is used. It can be programmed in-place using Will Sowerbutts' FLASH4 utility (included with RomWBW). For RAM, an AS6C4008 512k x 8 SRAM is used, providing 512K of addressable memory. Both are divided into 32k pages, and any page of either chip can be paged into the lower 32k of the address space. The upper 32k is permanently mapped to the last page of the RAM chip.

## Paging
Upon reset, the first page of the ROM is mapped to the lower 32k. If left alone, the board will behave like any 32k ROM+32k RAM RCBUS system. A config register is used to set the currently mapped page, presented as an I/O device at 0x78 (the address used by most "large" Z80 memory systems). The low nibble sets the page number (0-15), and high bit 7 selects the chip (0 for ROM, 1 for RAM).

The port address and layout are chosen to be (mostly) compatible with the [RetroBrew SBC](https://www.retrobrewcomputers.org/doku.php?id=boards:sbc:sbc_v2:sbc_v2-003). The SBC has separate ROM and RAM registers (on 0x78 and 0x7C); this board uses a single register for both, which can be written to from either address.

## Firmware
Many [RCBUS ROM images](https://github.com/RCBUSZ80/RCBUS/tree/master/ROMs) can be used (selecting the right image for your serial board), but some will require modification to accommodate the different paging system. The 32k BASIC images work unmodified. Stephen C Cousins' excellent [Small Computer Monitor](https://smallcomputercentral.wordpress.com/small-computer-monitor/) can be used as well - choose the R3 configuration. CP/M loading *won't work* without code changes.

But most importantly, this board is compatible with [RomWBW](https://github.com/wwarthen/RomWBW), providing several CP/M versions, ROM applications, and maximized free application memory. It will need to be built from source with configuration changes, although this is fairly straightforward (RomWBW includes its entire build toolchain in the source tree). The build process is covered in [this page](https://github.com/wwarthen/RomWBW/blob/master/Source/ReadMe.txt) from the RomWBW documentation (including Linux and Mac instructions). An example image can be found in the [firmware](/firmware) directory.

### RomWBW Config changes
With the [source tree](https://github.com/wwarthen/RomWBW) unpacked, go into `Source/HBIOS/Config` and make a copy of `RCZ80_std.asm` (call it `RCZ80_pd.asm`). Edit this new file, and add the following lines:
```
MEMMGR		.SET	MM_SBC		; MEMORY MANAGER: MM_[SBC|Z2|N8|Z180]
MPCL_RAM	.EQU	$78		; SBC MEM MGR RAM PAGE SELECT REG (WRITE ONLY)
MPCL_ROM	.EQU	$7C		; SBC MEM MGR ROM PAGE SELECT REG (WRITE ONLY)
;
```
With our new file, we can go ahead and build our ROM image. Under Windows, this is done with the following commands in `Source`:

* `BuildShared`
  * Wait for this to complete, ignoring the harmless program origin warnings
* `BuildROM`
  * Enter `RCZ80` at the *Platform* prompt
  * Enter `pd` at the *Configuration* prompt

Once the build is complete, we are done! Back at the root of the source tree, go into `Binary` and retrieve our new `RCZ80_pd.rom`. This can now be burned to the flash chip using an EPROM programmer, such as a [TL866](https://www.ebay.com/sch/i.html?_nkw=tl866ii+plus).

## Part selection
Bill Of Materials and part references are below. I recommend using gold-plated header for the bus connection - I use these ones from [Pololu](https://www.pololu.com/product/967) or [Sparkfun](https://www.sparkfun.com/products/553). A nice detail of these, is that they sit at the same height as double-row header in a backplane.

The specified parts are just the ones I used, and can be substituted as needed - Mouser links provided for convenience and reference.

| Reference | Value | Qty | Mouser link |
| --------- | ----- | --- | ----------- |
| C1-C6 | 100nF ceramic | 6 | [KEMET C315C104M5U5TA](https://www.mouser.com/ProductDetail/C315C104M5U5TA7303) |
| J1 | 1x40 right-angle header | 1 | [Wurth 61304011021](https://www.mouser.com/ProductDetail/61304011021) |
| U1 | SST39SF040 NOR flash | 1 | [Microchip SST39SF040-70-4C-PHE](https://www.mouser.com/ProductDetail/SST39SF040-70-4C-PHE) |
| U2 | AS6C4008 SRAM | 1 | [Alliance AS6C4008-55PCN](https://www.mouser.com/ProductDetail/AS6C4008-55PCN) |
| | sockets | 2 | [Amphenol DILB32P-223TLF](https://www.mouser.com/ProductDetail/DILB32P-223TLF) |
| U3 | 74HCT138 | 1 | [TI SN74HCT138N](https://www.mouser.com/ProductDetail/SN74HCT138N) |
| U4 | 74HCT174 | 1 | [TI CD74HCT174E](https://www.mouser.com/ProductDetail/CD74HCT174E) |
| U5 | 74HCT32 | 1 | [TI SN74HCT32N](https://www.mouser.com/ProductDetail/SN74HCT32N) |
| U6 | 74HCT253 | 1 | [TI CD74HCT253E](https://www.mouser.com/ProductDetail/CD74HCT253E) |
