===========================
System and memory overview
===========================

Introduction
==============
The on-chip processor uses RISC-V 32-bit with floating point.
With high-speed processing memory system (see the L1C chapter for details), to achieve high-quality computing efficiency.
External to the processor is a multilayer 32-bit AHB architecture with low power consumption, low latency, and high flexibility.
The memory section contains high-speed tightly coupled memory as well as cache and system shared memory.
Off-chip memory supports Flash expansion.

Main features
=================
• RISC-V 32-bit with floating point
• Multi-layer 32-bit AHB bus architecture
• 132KB RAM
• 192KB ROM
• Off-chip memory Flash

Function description
========================

The BL702/704/706 bus connection and address access are summarized as follows:

The bus master includes CPU, Ethernet, DMA, encryption engine, debugging interface.
The bus slave includes memory, peripherals, and Zigbee/BLE.
Except for Ethernet and encryption engine which can only access memory, all other bus masters can access all bus slaves.

.. table:: Bus connection

    +--------------------+------------+----------+--------+-------------------+-----------------+
    |  Slave / Master    |  CPU       | Ethernet | DMA    |Encryption engine  | Debug interface |
    +====================+============+==========+========+===================+=================+
    | memory             | V          | V        | V      |      V            | V               |
    +--------------------+------------+----------+--------+-------------------+-----------------+
    | Peripheral         | V          | \-       | V      |      \-           | V               |
    +--------------------+------------+----------+--------+-------------------+-----------------+
    |Zigbee/BLE          | V          | \-       | V      |      \-           | V               |
    +--------------------+------------+----------+--------+-------------------+-----------------+

The address access mainly distinguishes "memory" or "peripheral" by [27:24], and the [31:28] bits can be ignored.
The memory space is consecutive addresses 0x2010000~0x202FFFF (128KB SRAM)，the read-only memory address is 0x1000000，and the deep sleep memory address is 0x0010000。
The off-chip space address is 0x3000000 (maximum support 8MB Flash).
The peripheral space is 0x0000000 ~ 0x000F000.

.. table:: Memory Map 

    +--------+------------+-------+--------------------------------------------------+
    |Module  |Base Address| Size  |    Description                                   |
    +========+============+=======+==================================================+
    | RETRAM | 0x40010000 | 4KB   | Deep sleep memory (Retention RAM)                |
    +--------+------------+-------+--------------------------------------------------+
    | HBN    | 0x4000F000 | 4KB   | Deep sleep control (Hibernate)                   |
    +--------+------------+-------+--------------------------------------------------+
    | PDS    | 0x4000E000 | 4KB   | Sleep control (Power Down Sleep)                 |
    +--------+------------+-------+--------------------------------------------------+
    | USB    | 0x4000D800 | 1KB   | USB control                                      |
    +--------+------------+-------+--------------------------------------------------+
    | EMAC   | 0x4000D000 | 2KB   | Ethernet MAC control                             |
    +--------+------------+-------+--------------------------------------------------+
    | DMA    | 0x4000C000 | 4KB   | DMA control                                      |
    +--------+------------+-------+--------------------------------------------------+
    | QSPI   | 0x4000B000 | 4KB   | Flash/pSRAM QSPI control                         |
    +--------+------------+-------+--------------------------------------------------+
    | I2S    | 0x4000AA00 | 256B  | I2S control                                      |
    +--------+------------+-------+--------------------------------------------------+
    | KYS    | 0x4000A900 | 256B  | Key-Scan control                                 |
    +--------+------------+-------+--------------------------------------------------+
    | QDEC2  | 0x4000A880 | 64B   | Quadrature decoder control                       |
    +--------+------------+-------+--------------------------------------------------+
    | QDEC1  | 0x4000A840 | 64B   | Quadrature decoder control                       |
    +--------+------------+-------+--------------------------------------------------+
    | QDEC0  | 0x4000A800 | 64B   | Quadrature decoder control                       |
    +--------+------------+-------+--------------------------------------------------+
    | IRR    | 0x4000A600 | 256B  | IR Remote control                                |
    +--------+------------+-------+--------------------------------------------------+
    | TIMER  | 0x4000A500 | 256B  | Timer control                                    |
    +--------+------------+-------+--------------------------------------------------+
    | PWM    | 0x4000A400 | 256B  | Pulse Width Modulation *5 control                |
    +--------+------------+-------+--------------------------------------------------+
    | I2C    | 0x4000A300 | 256B  | I2C control                                      |
    +--------+------------+-------+--------------------------------------------------+
    | SPI    | 0x4000A200 | 256B  | SPI master/slave control                         |
    +--------+------------+-------+--------------------------------------------------+
    | UART1  | 0x4000A100 | 256B  | UART control (support LIN-bus)                   |
    +--------+------------+-------+--------------------------------------------------+
    | UART0  | 0x4000A000 | 256B  | UART control (support LIN-bus)                   |
    +--------+------------+-------+--------------------------------------------------+
    | L1C    | 0x40009000 | 4KB   | Cache control                                    |
    +--------+------------+-------+--------------------------------------------------+
    | eFuse  | 0x40007000 | 4KB   | eFuse memory control                             |
    +--------+------------+-------+--------------------------------------------------+
    | SEC    | 0x40004000 | 4KB   | Security engine                                  |
    +--------+------------+-------+--------------------------------------------------+
    | GPIP   | 0x40002000 | 4KB   | General purpose DAC/ADC/ACOMP interface control  |
    +--------+------------+-------+--------------------------------------------------+
    | MIX    | 0x40001000 | 4KB   | Mixed signal register                            |
    +--------+------------+-------+--------------------------------------------------+
    | GLB    | 0x40000000 | 4KB   | Global control register                          |
    +--------+------------+-------+--------------------------------------------------+
    | pSRAM  | 0x24000000 | 8MB   | pSRAM memory                                     |
    +--------+------------+-------+--------------------------------------------------+
    | XIP    | 0x23000000 | 8MB   | XIP Flash memory                                 |
    +--------+------------+-------+--------------------------------------------------+
    | OCRAM  | 0x22020000 | 64KB  | On-chip memory                                   |
    +--------+------------+-------+--------------------------------------------------+
    | DTCM   | 0x22014000 | 48KB  | Data cache memory                                |
    +--------+------------+-------+--------------------------------------------------+
    | ITCM   | 0x22010000 | 16KB  | Instruction cache memory                         |
    +--------+------------+-------+--------------------------------------------------+
    | ROM    | 0x21000000 | 192KB | Read-only memory                                 |
    +--------+------------+-------+--------------------------------------------------+

There are 64 interrupt sources. The level or edge trigger is configured by the CPU and can be masked. Details as follows:

.. table:: Interrupt source

    +-----------+----------------+
    |  Num      |  Signal source |
    +===========+================+
    | 54~63     | wireless       |
    +-----------+----------------+
    | 53        | brown-out      |
    +-----------+----------------+
    | 51~52     | hbn_irq        |
    +-----------+----------------+
    | 50        | pds_int        |
    +-----------+----------------+
    | 47~49     | wireless       |  
    +-----------+----------------+
    | 44        | gpio_irq       |
    +-----------+----------------+
    | 40~42     | qdec_int       |
    +-----------+----------------+
    | 39        | kys_int        |
    +-----------+----------------+
    | 35~38     | timer_irq      |
    +-----------+----------------+
    | 34        | pwm_int        |
    +-----------+----------------+
    | 32        | i2c_int        |
    +-----------+----------------+
    | 30        | uart1_irq      |
    +-----------+----------------+
    | 29        | uart0_irq      |
    +-----------+----------------+
    | 27        | spi_int        |
    +-----------+----------------+
    | 26        | efuse_int      |
    +-----------+----------------+
    | 25        | adc_int        |
    +-----------+----------------+
    | 23        | flash_int      |
    +-----------+----------------+
    | 22        | emac_int       |
    +-----------+----------------+
    | 21        | usb_int        |
    +-----------+----------------+
    | 19~20     | ir_remote_int  |
    +-----------+----------------+
    | 18        | i2s_int        |
    +-----------+----------------+
    | 15        | dma_int        |
    +-----------+----------------+
    | 9~14      | sec_eng_int    |
    +-----------+----------------+
    | 8         | err_int        |
    +-----------+----------------+
    | 5~6       | rf_int         |
    +-----------+----------------+
    | 0~4       | err_int        |
    +-----------+----------------+