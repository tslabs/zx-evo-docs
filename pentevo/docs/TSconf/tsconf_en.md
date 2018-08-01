# TS-Configuration

## Features

- High compatibility with original Pentagon-128 clone

- Advanced video features:
  - Pixel resolutions 360x288, 320x240, 320x200, 256x192
  - Up to 720x288 Hi-res pixel resolution
  - Hardware scrolled graphic planes
  - 256 and 16 indexed colors per pixel
  - Programmable color RAM with RGB555 color space and 256 cells
  - 512 and 256 bytes per line addressing
  - Text mode with loadable font and hardware vertical scroll
  - Up to 256 graphic screens

- Hardware engine for Tiles and Sprites graphics
  - Up to 85 sprites per line
  - Sprites sized from 8x8 to 64x64 pixels
  - Up to 3 sprite planes
  - Up to 2 tile planes with 8x8 pixels tiles
  - Up to 16 palettes for sprites per line
  - Up to 4 palettes for tiles per line for each tile plane

- Z80 Memory addressing enhancements:
  - Programmable RAM page for any 16kB window

- Z80 acceleration features
  - Selectable CPU clock 14MHz, 7MHz and 3,5MHz
  - 512 bytes of zero-wait RAM for 14MHz
  - On-the-fly programmable maskable interrupt position
  - Separate IM2 vectors for different interrupt sources

- Advanced hardware features
  - DRAM-to-Device, Device-to-DRAM and DRAM-to-DRAM DMA Controller

## Conventions

### Register access

When writing unused bits in a register must always be written with 0, if not specified otherwise. This is to maintain compatibility with future versions of configuration.

When reading unused bits in a register must be ignored.

### Formating

The following formatting is used in this document.

#C000 - hexadecimal numbers.

**VConfig** - registers names.

***NOGFX*** - register bits names.

*window* - special term.

`HALT` - CPU mnemonics.

### Terms

R/W - register or register bit access. R - read, W - write.

**REG**[n] - bit n from register **REG**.

**REG**[n..m] - bits n to m from register **REG**.

## Achitecture

## Programming model

Hardware in TS-Configuration is controlled via dedicated pull of ports with common address #nnAF. They can be written and/or read. Their access modes are specified in descriptions.

Some of legacy ports are also provided like Z-Controller, Gluclock and Pentagon-128 ports. See correspondent sections for their description.

It is also possible to access #nnAF registers for write by writing memory in pre-configured addresses using **FMAddr** register.

Dedicated memory files like color RAM and sprite descriptors are also accessed using **FMAddr**.

## Registers

### Register access

Access to TS-Configuration registers is performed via set of CPU ports with address #nnAF, where lower A0..A7 address part is #AF and higher A8..A15 is a register number.



### FMAPS

## ��������� � ������

### ���������� ��������� ������

TS ������������ ��������� ���������� �� 4096�� ��� � 512�� ���.

Since Z80 CPU has 16-bit address bus it can only access 64kB of memory. CPU addressable span is split into four regions - *windows*. Each *window* is 16kB and has its dedicated 8-bit register to select memory *page* to be mapped into the correspondent CPU addresses. 256 *pages* 16kB each give 4096kB of addressable memory.

Since ROM is only 512kB three MSBs of correspondent *page* register are not used when ROM is selected for mapping.

See table below for *windows* addresses and *page* registers.

CPU *window*|CPU address|*Page* register|R/W|Reset value
----------|---------------|------------------------|---|----
0         |#0000..#3FFF    |Page0                   | W |#00
1         |#4000..#7FFF    |Page1                   | W |#05
2         |#8000..#BFFF    |Page2                   |R/W|#02
3         |#C000..#FFFF    |Page3                   |R/W|#00

Only RAM can be mapped into *windows* 1..3.

RAM or ROM can be mapped into *window* 0. See description below.

#### ������� MemConfig

| Bit         | 7         | 6         | 5    | 4    | 3      | 2       | 1     | 0      |
| ----------- | --------- | --------- | ---- | ---- | ------ | ------- | ----- | ------ |
| Name        | LCK128[1] | LCK128[0] | -    | -    | W0_RAM | !W0_MAP | W0_WE | ROM128 |
| R/W         | W         | W         | -    | -    | W      | W       | W     | W      |
| Reset value | 0         | 0         | x    | x    | 0      | 1       | 0     | 0      |

* ���� 7:6 - ����� ������ ����������� ����� #7FFD,
* ���� 5:4 - ���������������,
* ��� 3 - ����� ROM/RAM,
* ��� 2 - ���������� ��������,
* ��� 1 - ���������� ������ � ROM/RAM,
* ��� 0 - ����� ������� ��������.

� ������� ���� *W0_WE* �������� **MemConfig** ����� ��������� (�������� 0) ��� ��������� (�������� 1) ������ � ���� 0. ��� ������� RAM - ��� ������ ������ ������ ������. ��� ROM ������ ����� ������������ �� ���� ������.

#### ������� ������� � ���� 0

� ������� ���� ����� ���������� �������� RAM/ROM � ���� �������:

- ����� �������� �������
- ������� �����

����� ���������� � ������� ���� *!W0_MAP* �������� **MemConfig**: 0 - ����� �������� �������, 1 - ������� �����.

� ������� ������ ����� ������������ �������� ������� �� �������� **Page0**[7:0] ��� RAM � **Page0**[4:0] ��� ROM.

����� �������� ������� ������������ ��� ����������� 64�� �������� ���, ������� ����� ������������� ��� � ROM, ��� � � RAM ����������. ����� �������� ����� ��� ���������� � ������� ������� DOS � ���� *ROM128* �������� **MemConfig** (��� ����� #7FFD). ������ DOS � ��� *ROM128* ������������� � �������� 1 � 0 ���� ������ �������� ��������������, � ��� ��������� ���� ������� �� �������� **Page0**, ����� ������� �������� ��� ������ ������������� � ��������� ������� 4.

������ �������� ��� � ����������� ������� ��� ������ ������� ����� ��� ������������ � ��������� �������:

| DOS  | ROM128 | �������� ��� |
| ---- | ------ | ------------ |
| 0    | 0      | Service      |
| 0    | 1      | DOS          |
| 1    | 0      | 128          |
| 1    | 1      | 48           |

#### Paging via port #7FFD

Port #7FFD provides compatibility with Pentagon-128k (with 512k and 1024k extensions).

Paging mode *LCK128*[1:0] �������� **MemConfig**:

LCK128[1:0]|����� ����� #7FFD| ����������� ��� � ������� Page3
-----------|-----------------|--------------------------------------
00         | 512��        | Page3[4:0] = #7FFD[7:6], #7FFD[2:0]
01         | 128��         | Page3[2:0] = #7FFD[2:0]
10         | ����            | See description below 
11         | 1024��        | Page3[5:0] = #7FFD[5], #7FFD[7:6], #7FFD[2:0]

��� ������ �������� � ���� #7FFD � ������� 128��, 512�� � 1024��, ����, ���������� �� ������������ �������, ������������ � ������� ������� �������� **Page3**.

����� *����* ��������� � ������� ������� ��������� ����� �������� � 512�� RAM (`out (c),a`), � ����� �������� ��������� � 128�� RAM (`out (#FD),a`). ����� ��������� ������������ �� ������ �������..

The function of bits 5..7 of port #7FFD depends on Lock mode set in **MemConfig** register.

| Bits   | 7     | 6     | 5     | 4    | 3    | 2     | 1     | 0     |
| ------ | ----- | ----- | ----- | ---- | ---- | ----- | ----- | ----- |
| Name*1 | -     | -     | LOCK  | ROM  | SCR  | PG[2] | PG[1] | PG[0] |
| Name*2 | PG[4] | PG[3] | LOCK  | ROM  | SCR  | PG[2] | PG[1] | PG[0] |
| Name*3 | PG[4] | PG[3] | PG[5] | ROM  | SCR  | PG[2] | PG[1] | PG[0] |
| R/W    | W     | W     | W     | W    | W    | W     | W     | W     |
| Init   | 0     | 0     | 0     | 0    | 0    | 0     | 0     | 0     |

*1 - for 128k mode
*2 - for 512k mode
*3 - for 1024k mode

* ***PG*** - Selects page mapped at address #C000. Only first 1024kB of RAM available for this paging.
* ***LOCK*** - Disables paging and locks the selected page at #C000. For example, if value #23 is written, the page number 3 will be mapped to #C000 and cannot be changed via port #7FFD until system reset. Paging via **Page0..3** is still available.
* ***ROM*** - Selects Basic-128 ROM page. See description of **MemConfig**.
* ***SCR*** - Selects active screen. 0 writes 5 to ***VPage***, while 1 writes 7. Has no line-latch effect (see [line-latch]).

### CPU modes control

CPU clock frequency can be selected from 14 / 7 / 3.5MHz.

In 14MHz mode each access to RAM is delayed to be synchronized with DRAM Controller which works slower than CPU. To minimize the effect of such throttling Cache Controller is used. A dedicated 512 bytes Cache Memory caches RAM read accesses and provides data for the CPU without delay on second and next accesses to the cached addresses. See [Cache description] for more detailes.

Cache has no effect in 3.5 and 7MHz modes.

#### ������� SysConfig

Bit        |7|6|5|4|3|2|1|0
-----------|-|-|-|-|-|-|-|-|
Name       |-|-|-|-|-|CACHE|ZCLK[1]|ZCLK[0]
R/W        |-|-|-|-|-|W|W|W
Reset value|x|x|x|x|x|0|0|0

* ���� 7:3 - ���������������.
* ��� 2 - ���������� ����������� �������� � RAM,
* ���� 1:0 - ���������� �������� ����������

##### ������� ����������

����� �������������� 3 ������ ������ ����������: 3.5MHz, 7.0MHz, 14.0MHz. ��� ������� ���� �������� **SysConfig** ��������� ������� ���� �� ���� ������� �������� ��������� �������:

ZCLK[1:0]| MHz
---------|----
00       |3.5
01       |7.0
10       |14.0
11       |���������������

##### ���

� TS ������������ ����� �������� ����������� ���� �������� �� ������/������ ����� ����������� � RAM. ������� � ROM �� ����������.

������������ ���������� ������ (64��) ������� ������� �� 4 ���� �� 16 �������� � ��� ������� ������ ���� � �������� **CacheConfig** ������������ ��������������� ���, ������� �������� ��� ��������� ���������� � ���� ����.

#### ������� CacheConfig
	���� 7  6  5  4    3       2       1       0
	     -  -  -  - EN_C000 EN_8000 EN_4000 EN_0000
	R/W                W       W       W       W
	Init x  x  x  x    0       0       0       0

* ���� 7:4 - ���������������.
* ��� 3 - ���������� ����������� ���� RAM #C000-#FFFF,
* ��� 2 - ���������� ����������� ���� RAM #8000-#BFFF,
* ��� 1 - ���������� ����������� ���� RAM #4000-#7FFF,
* ��� 0 - ���������� ����������� ���� RAM #0000-#3FFF,

��� ������ �������� � ��� *CACHE* �������� **SysConfig** ��� �� �������� ���������� � ���� 0-3 �������� **CacheConfig**. ��� ��������� �������� � ��������� ����������� ��������� ��� ���� ������� RAM.

**��������!** ������� �� DMA � RAM �� ����������. �� ���� ������� ����� DMA ����������, ���������� � ������ RAM ��� ������� �������� �����������, ���������� ��������� ����������� ����.

��� ����������� ���� ��������� �������� 512 ���� � ����� ���������������� ������. ��� ���� ����������, ���� �� ��������� ������� ���������� ���.
������ ����������� ����:

```assembly
LD HL, #FE00
LD DE, #FE00
LD BC, #0200
LDIR
```

## ���������� ����������

��� ���� ������� ���������� (`IM 0`, `IM 1`, `IM 2`) TS ������������ ������������ ��������� ���������� ������������ ���������� INT:

* �������� (*FRAME*)
* �������� (*LINE*)
* ��������� DMA ���������� (*DMA*)

� ������ ������� ���������� ������� ������������, ������� �������������� ���������� � ������� �����������. ��� ���������� ISR ������������ `EI : RET` ����� ���������� ��������� ���������� �� ������� ����������, ������� ������������ � ��������� �������� ����� ���������� `RET`.

#### Frame interrupt

�������� *FRAME* �����������, ����� �������� ��������� ������ ��������� � ���������� **HSINT**, **VSINT**. �������� *LINE* ����������� � ������ ������, ����� �������������� ������� ������ ����� 0. �������� *DMA* ����������� ����� ��������� DMA ����������.

#### Line interrupt
#### DMA interrupt
#### Wait Ports interrupt

� ������ `IM 2` ������ �������� ���������� ��������� ������ INT � ���������� ����������� ����� �� ���� ������.

��������|���������|�����
--------|---------|-----
FRAME   |    0    | #FF
LINE    |    1    | #FD
DMA     |    2    | #FB

������� **INTMask** �������� ���� ��������������� ��������� ������������ ����������: 0 - ��������, 1 - ��������. �� RESET ���� ������������ �������� #01 - �������� *FRAME*, ��� ��������� ���������. ���� �� ISR � ������� ����������� �������� 0 � ��������������� ��� ����� ��������� ����������, ���������� � ������ ������ ���������, �� ���������� ��� ����� � ���������� ���������� �� �����. ������ 1 �� ������ �� ��������� ���������� ����������.

### Registers

#### ������� INTMask

	����  7   6   5   4   3   2    1    0
	      -   -   -   -   -  DMA LINE FRAME
	R/W                       W    W    W
	Init  x   x   x   x   x   0    0    1

* ���� 7:3 - ���������������,
* ��� 2 - ���������� ��������� *DMA*,
* ��� 1 - ���������� ��������� *LINE*,
* ��� 0 - ���������� ��������� *FRAME*,

## �������

### Modes

#### ZX
#### 16c
#### 256c
#### text

### raster sizes
### screen page
### CRAM
#### VDAC mode
#### PWM mode
#### DMA
### scrolls
### registers

## TSU
### Features
### tiles
Overview

#### bitmap
#### tile map
#### scrolls
#### registers

### sprites
Overview

#### SFILE
#### registers

## ���������� DMA

���������� DMA ��������� ���������� ������ ����� ������ ������������ ���������� ��� RAM, IDE, SPI, CRAM � SFILE ����� ���������. �� ����� �������� ������ ��������� ����� ���� ����� ���-�� ������. � ���� ������ ������� ����� ���� �������� ���� ���� DMA ����������.

��� ���������� DMA ������������� ��������� ��������:

�������  |����������
---------|-----------------------------
DMASAddr*| ����� ��������� ������ � RAM
DMADAddr*| ����� ��������� ������ � RAM
DMALen   | ����� �����
DMANum   | ���������� ������
DMACtrl  | ������� ���������� DMA
DMAStatus| ��������� ����������� DMA

DMA ���������� ���������� �������� ������ �������. �������� ������ � ������ �������� �������� WORD, �.�. 16 ���. � ��������� DMA ����������� ����� ���������� ��� ������ ������ ����� (������� **DMALen**), ��� � ���������� ������ (������� **DMANum**) ��� ���������. ������������� � �������� **DMACtrl** ����� ������ ������������ ��� ������� ����� ����������.

�������� � ��������� **DMALen** � **DMANum** ������ ���� �� ������� ������, �.�. ��� ��������� � ��������� 1 - 256.

### ������� DMACtrl

	����  7   6   5      4     3    2  1  0
	     W/R  - S_ALGN D_ALGN A_SZ DDEV[2:0]
	R/W   W       W      W     W    W  W  W
	Init  x   x   x      x     x    x  x  x

* ��� 7 - ����������� �������� ������: 0 - � ������, 1 - �� ������,
* ��� 6 - ���������������,
* ��� 5 - ��������� ������������ ������ ��������� (0 - ���������, 1 - ��������),
* ��� 4 - ��������� ������������ ������ ��������� (0 - ���������, 1 - ��������),
* ��� 3 - ������ ������������: 0 - 256 ����, 1 - 512 ����,
* ���� 2:0 - ����� ���������� ��� ������ �������.

���� �������� ������������ ������ ��������� ��� ���������, �� ����� ��������� ������� ����� � ����������� �������� ����� �������� ������������ 256 ��� 512 ���� � ����������� �� ���� *A_SZ*. ���� ������������ ��� ������ ���������, �� �������� �������� �� ���������� � ������������� ����, ��� ���� � ��� �� ������ ��������� ��������� ���������� �����.

������ ��������� � ��������� ������ (*DMASAddr* � *DMADAddr*) �������� 22 ������� � ��� ������ � ��� �������� ������������ �� 3 ��������: **DMASAddrL**, **DMASAddrH** � **DMASAddrX** - ��� ������ ��������� ������, **DMADAddrL**, **DMADAddrH**, **DMADAddrX** - ��� ������ ��������� ������.

��� �������� ��������� �������� **DMASAddrL** � **DMASAddrH** ������ �������� ������ �������� ������, � ������� **DMASAddrX** ������ ����� ��������. ��� ������ ��������� ��� ����������.

������� ������ ���������:

��� W/R|DDEV[2:0]|��������|��������|��������
:-----:|:-------:|:------:|:------:|----------------
   0   |   000   |   -    |   -    | ���������������
   1   |   000   |   -    |   -    | ���������������
   0   |   001   |  RAM   |  RAM   | ����������� RAM
   1   |   001   |  BLT   |  RAM   | �������� RAM
   0   |   010   |  SPI   |  RAM   | ����������� �� SPI � RAM
   1   |   010   |  RAM   |  SPI   | ����������� �� RAM � SPI
   0   |   011   |  IDE   |  RAM   | ����������� �� IDE � RAM
   1   |   011   |  RAM   |  IDE   | ����������� �� RAM � IDE
   0   |   100   |  FILL  |  RAM   | ���������� RAM
   1   |   100   |  RAM   |  CRAM  | ����������� �� RAM � CRAM
   0   |   101   |   -    |   -    | ���������������
   1   |   101   |  RAM   |  SFILE | ����������� �� RAM � SFILE
   0   |   110   |   -    |   -    | ���������������
   1   |   110   |   -    |   -    | ���������������
   0   |   111   |   -    |   -    | ���������������
   1   |   111   |   -    |   -    | ���������������

**�������� RAM**

������ �������� �� ��������� � ���������. ���� �������� � ��������� �� ����� 0, �� ������ �� ��������� ������������ ����.

**���������� RAM**

�������� ���� WORD �� ��������� � ��� �������� ������������ ��� ���������� ���������.

### ������� DMAStatus

	����    7    6  5  4  3  2  1  0
	     DMA_ACT -  -  -  -  -  -  -
	R/W     R
	Init    x    x  x  x  x  x  x  x

* ��� 7 - ��������� DMA ���������� (0 - �� �������, 1 - �������),
* ���� 6:0 - ���������������.

��������� DMA ���������� ����� ���������� �� ���� *DMA_ACT* �������� **DMAStatus**. �� ��������� ���������� ������������ ��������������� ����������, ���� ��� ���� ���������. ��������� �� ���� ����� ������ � ������� [���������� ����������][interrupts].

**��������!** ������, ������������ � ������� DMA �� ����������, ������� �������� � ���� ���������� ����� ���� ������ ��� ������� ��������� �����������, ���� ����� ���������� ����������� ����������� ����. ��������� � ������ ���� ������� � ������� [����������� �������� � RAM][cpu#cache].

## Sound

### Registers

## SPI

### Registers

## IDE

### Registers

## RTC

### Registers

## VDOS

### Registers

## Version detect

## Credits

Special thanks to: Earl (Dmitry Limonov) for initial version of this datasheet in Russian.
