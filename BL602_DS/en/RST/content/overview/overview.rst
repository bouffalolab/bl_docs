=========
Overview
=========
BL602/BL604 is Wi-Fi + BLE combo chipset for ultra-low-cost and low-power application.

Wireless subsystem contains 2.4G radio, Wi-Fi 802.11b/g/n and BLE 5.0 baseband/MAC designs. Microcontroller subsystem contains a low-power 32-bit RISC CPU, high-speed cache and memories. Power Management Unit controls low-power modes. Moreover, variety of security features are supported.

Peripheral interfaces include SDIO, SPI, UART, I2C, IR remote, PWM, ADC, DAC, PIR, and GPIOs.

.. figure:: picture/Functionalblockdiagram.png
   :align: center

   Block Diagram

Wireless
===========
- 2.4 GHz RF transceiver
- Wi-Fi 802.11b/g/n
- Bluetooth® Low Energy 5.0

  BLE 5.0 Channel Selection#2 is supported

  2M PHY / Coded PHY / ADV extension is not supported
- Wi-Fi 20MHz bandwidth
- Wi-Fi Security WPS / WEP / WPA / WPA2 Personal / WPA2 Enterprise / WPA3
- STA，SoftAP，STA+SoftAP and sniffer modes
- Multi-cloud connectivity
- Wi-Fi fast connection with BLE assistance
- Wi-Fi and BLE coexistence
- Integrated balun, PA/LNA

MCU Subsystem
===============
- 32-bit RISC CPU with FPU (floating point unit)
- Level-1 cache
- One RTC timer update to one year
- Two 32-bit general purpose timers
- Four DMA channels
- DFS (dynamic frequency scaling) from 1MHz to 192MHz
- JTAG development support
- XIP QSPI Flash with hardware encryption support

Memory
========
- 276KB RAM
- 128KB ROM
- 1Kb eFuse
- Embedded Flash (Optional)

Security
=========
- Secure boot
- Secure debug ports
- QSPI Flash On-The-Fly AES Decryption (OTFAD) - AES-128, CTR mode
- AES 128/192/256 bits
- SHA-1/224/256
- TRNG (True Random Number Generator)
- PKA (Public Key Accelerator)

Peripheral
=============
- One SDIO 2.0 slave
- One SPI master/slave
- Two UART
- One I2C master/slave
- Five PWM channels
- 10-bit general DAC
- 12-bit general ADC
- Two general analog comparators (ACOMP)
- PIR (Passive Infra-Red) detection
- IR remote
- 16 or 23 GPIOs

Power Management
===================
- Off
- Hibernate (flexible modes)
- Power Down Sleep (flexible modes)
- Active

Clock
=========
- Support XTAL 24/32/38.4/40MHz
- Internal RC 32kHz oscillator
- Internal RC 32MHz oscillator
- Internal System PLL
- XTAL 32kHz

