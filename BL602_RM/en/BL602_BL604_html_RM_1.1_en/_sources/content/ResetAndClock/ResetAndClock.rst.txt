===================
Reset and clock
===================

Introduction
==============
The reset sources included in the chip: hardware reset, watchdog reset, software reset.
The chip contains multiple clock sources: XTAL, PLL, RC. It is allocated to each module through configuration such as frequency division.

Reset source
===============

The reset sources are as follows:

- Hardware reset: reset via pins

  - Pin global reset (PAD_EXT_RST = 1-> 0): all logic will reset and return to the initial state(for QFN40)

  - Pin power reset (CHIP_EN = 0-> 1): similar to power management reset

  - Power management reset: The chip is restored from power failure, and the HBN logic resets the chip system

- Watchdog reset

  - When the watchdog alarm triggers a reset signal, the reset management unit will reset the chip system after necessary preparations, and the internal logic of the watchdog will record the status of the watchdog reset

- Software reset: local or partial reset according to software setting register

  - Software initial reset (reg_ctrl_pwron_rst): The rising edge of this register is triggered by software to reset the chip system

  - Software CPU reset (reg_ctrl_cpu_reset): The rising edge of this register is triggered by software to reset the CPU part of the system

  - Retain necessary logic processing such as power management unit, perform chip system reset

  - Software module reset: Set software reset according to the requirements of specific modules

.. figure:: /content/ResetAndClock/picture/Reset_top.png
   :align: center

   Reset source

Clock source
===============

Clock source contains:

 - XTAL   ：External crystal clock, according to system requirements, the frequency can be selected from 24, 32, 38.4, 40MHz.

 - XTAL32K：External crystal clock, frequency 32KHz
 - RC32K  ：RC oscillator clock, 32KHz, provides calibration
 - RC32M  ：RC oscillator clock, frequency 32MHz, provides calibration
 - PLL    ：Phase-locked loop clock, internal system high-speed clock, the highest frequency supports 192MHz

The clock control unit distributes the clock from the oscillator to the core and peripheral devices.
By selecting the system clock source, dynamic frequency divider, clock configuration, sleep using 32KHz clock to achieve low power clock management.

Peripheral clock includes: Flash、UART、I2C、SPI、PWM、IR-remote、ADC、DAC.

.. figure:: /content/ResetAndClock/picture/clocktree.png
   :align: center

   Clock tree
