==================
Reset and clock
==================
Introduction
==============
The reset sources included in the chip: hardware reset, watchdog reset, software reset.
The chip contains multiple clock sources: XTAL, PLL, RC. It is allocated to each module through configuration such as frequency division.

Reset source
=============

The reset sources are as follows:

- Hardware reset: reset via pin

   - Pin power reset (PU_CHIP = 0-> 1): similar to power reset
   - Power-on reset: the chip recovers from power failure, and HBN logic resets the chip system

- Watchdog reset

   - When the watchdog alarm triggers the reset signal, the reset management unit will reset the chip system after necessary preparations, and the internal logic of the watchdog will record the status of the watchdog reset

- Software reset: partial reset by software setting register

   - Software initial reset (reg_ctrl_pwron_rst): trigger the rising edge of this register by software to reset the chip system
   - Software CPU reset (reg_ctrl_cpu_reset): Trigger the rising edge of this register by software to reset the CPU part of the system
   - Software system reset (reg_ctrl_sys_reset): Trigger the rising edge of this register by software, retain necessary logic processing such as power management unit, and reset the chip part of the system
   - Software module reset: Set software reset according to the needs of specific modules


.. figure:: ../../picture/ResetTop.svg
   :align: center

   Reset source

Clock source
==============

The clock source includes:

- XTAL: External crystal oscillator clock, mainly supporting frequency 32MHz
- XTAL32K: external crystal oscillator clock, frequency 32KHz
- RC32M: RC oscillator clock, frequency 32MHz, provides calibration
- PLL: PLL clock, internal system high-speed clock, the highest frequency supports 144MHz

The clock control unit distributes the clock from the oscillator to the core and peripheral devices.

The system clock source, dynamic divider, clock configuration, and sleep use 32KHz clock can be selected to achieve low-power clock management.

Peripheral clocks include: Flash, USB2.0, Ethernet, UART, I2C, I2S, SPI, PWM, IR-remote, QDEC, KeyScan, ADC, DAC.


.. figure:: ../../picture/ClockTree.svg
   :align: center

   Clock architecture
