=========
LowPower
=========

Overview
=====
Low power consumption is an important indicator of IoT applications. The chip's CPU supports the working mode, idle power saving mode, and sleep mode. You can select a proper mode according to the current application scenario to reduce the chip's power consumption and prolong the battery life.

.. figure:: ../../picture/LPPwrSt.png
   :align: center

   Low power modes

Features
=========

- Clock control: clock control of peripherals in GLB, small-scale power saving, and fast response speed

- Sleep control (PDS): 5 levels (PDS 1/2/3/7/15), large-scale power saving, moderate response speed

- Deep sleep control (HBN): 4 levels (HBN 0/1/2/3), global power saving, and long response time

Functional Description
==========

Power Domain
-----------
There are 8 power domains in the xxx chip, with main functions as follows:

- PD\_AON
  
  - HBN state machine, which can control the power/isolation unit/reset/clock of PD\_AON\_HBNRTC/PD\_AON\_HBNCORE/PD\_CORE
  
  - AON\_PIN (GPIO16/17/18/19) pin wakeup function
  
  - Reserve 4 readable and writable registers to save data in PDS/HBN0/HBN1/HBN2 modes
  
  - BOR (Brown Out Reset) function
  
  - LDO11\_AON/RT/SOC output voltage selection unit
  
  - Root, F32K, and Uart clock source selection
  
  - HBN\_OUT0\_PIR (Acomp0/Acomp1/bor/pir) wakeup mask, enable register, and interrupt status register

- PD\_AON\_HBNRTC
  
  - Select the control unit of RTC Counter clock source
  
  - RTC can be used for wakeup or LED flashing
  
  - RC32K and Xtal32X control functions

- PD\_AON\_HBNCORE
  
  - Partial power control register
  
  - Reserve HBN\_RAM, which can be used to store programs and data, so that data will not disappear in PDS/HBN0 modes
  
  - In HBN mode, it has the function of controlling AON\_PIN and keeping other IO
  
  - PIR digital control: PIR is a Passive Infra-Red (PIR) sensor, a peripheral in HBN area, which can be used as HBN wakeup source
  
  - Reserve LDO11SOC, LDO15RF, DCDC0, DCDC1 and DCDC2 control registers
  
  - Reserve XTAL, TSEN, Acomp0/1 and GPADC control registers
  
  - Reserve 4 readable and writable registers to save data in PDS/HBN0 modes

- PD\_CORE
  
  - The PDS state machine controls the power, isolation unit, reset, clock, and memory of PD\_CORE\_MISC/PD\_USB/PD\_CPU/PD\_WB
  
  - Reserve PDS\_RAM, so that data will not disappear in PDS mode
  
  - WIFI/BLE timer control
  
  - 160KB WRAM Retention/Sleep
  
  - Control the PDS interrupt and wakeup function
  
  - Reserve the control of GPIO0~15/20~36
  
  - Reserve the control of RC32M
  
  - PDS\_Timer timer

- PD\_CORE\_MISC
  
  - PD\_CORE\_MISC\_DIG and PD\_CORE\_MISC\_ANA are collectively referred to PD\_CORE\_MISC
  
  - Peripheral (including I2C/SDIO/UART/FLASH/SPI/ADC/DAC/GPIO/PWM)
  
  - Chip GLB register

- PD\_USB
  
  - USB controller

- PD\_CPU
  
  - NP\_CPU and cache unit
  
  - ROM, TCM

- PD\_WB
  
  - WIFI PHY/MAC
  
  - BLE PHY/MAC
  
  - RF Controller

Each power domain is controlled by 8 different power modes as shown below:

.. table:: Power mode

    +--------+------------+-----------+-----------------+----------------+------------+---------------+------------+------------+------------+
    |        |            | Power Domain                                                                                                     |
    +        +            +-----------+-----------------+----------------+------------+---------------+------------+------------+------------+
    | NO.    | Scenario   |  PD_AON   |  PD_AON_HBNRTC  | PD_AON_HBNCORE |  PD_CORE   | PD_CORE_MISC  |   PD_USB   |   PD_CPU   |    PD_WB   |
    +========+============+===========+=================+================+============+===============+============+============+============+
    | 1      | Normal     |    ON     |        ON       |       ON       |     ON     |       ON      |     ON     |     ON     |     ON     |
    +--------+------------+-----------+-----------------+----------------+------------+---------------+------------+------------+------------+
    | 2      | PDS1       |    ON     |        ON       |       ON       |     ON     |       ON      |     ON     |     ON     |     OFF    |
    +--------+------------+-----------+-----------------+----------------+------------+---------------+------------+------------+------------+
    | 3      | PDS2       |    ON     |        ON       |       ON       |     ON     |       ON      |     ON     |     OFF    |     ON     |
    +--------+------------+-----------+-----------------+----------------+------------+---------------+------------+------------+------------+
    | 4      | PDS3       |    ON     |        ON       |       ON       |     ON     |       ON      |     ON     |     OFF    |     OFF    |
    +--------+------------+-----------+-----------------+----------------+------------+---------------+------------+------------+------------+
    | 5      | PDS7       |    ON     |        ON       |       ON       |     ON     |       ON      |     OFF    |     OFF    |     OFF    |
    +--------+------------+-----------+-----------------+----------------+------------+---------------+------------+------------+------------+
    | 6      | PDS15      |    ON     |        ON       |       ON       |     ON     |       OFF     |     OFF    |     OFF    |     OFF    |
    +--------+------------+-----------+-----------------+----------------+------------+---------------+------------+------------+------------+
    | 7      | HBN0       |    ON     |        ON       |       ON       |    OFF     |       OFF     |     OFF    |     OFF    |     OFF    |
    +--------+------------+-----------+-----------------+----------------+------------+---------------+------------+------------+------------+
    | 8      | HBN1       |    ON     |        ON       |       OFF      |    OFF     |       OFF     |     OFF    |     OFF    |     OFF    |
    +--------+------------+-----------+-----------------+----------------+------------+---------------+------------+------------+------------+
    | 9      | HBN2       |    ON     |        OFF      |       OFF      |    OFF     |       OFF     |     OFF    |     OFF    |     OFF    |
    +--------+------------+-----------+-----------------+----------------+------------+---------------+------------+------------+------------+
    | 10     | HBN3       |    OFF    |        OFF      |       OFF      |    OFF     |       OFF     |     OFF    |     OFF    |     OFF    |
    +--------+------------+-----------+-----------------+----------------+------------+---------------+------------+------------+------------+

Wake-up Source
------------
The chip supports multiple wakeup sources, which can wake up the chip from different power modes.

The wake-up sources for different power modes are shown in the following table:

.. table:: Wakeup source 

   +--------------+------------------------------------------------------------------------+
   | Power mode   | Wakeup source                                                          |
   +==============+========================================================================+
   |PDS0          |AON_PIN/BOR/RTC/Pir/Acomp0/Acomp1/PDS_Timer/GPIO/IRRX/DM/USB/WIFI       |
   +--------------+------------------------------------------------------------------------+
   |PDS1          |AON_PIN/BOR/RTC/Pir/Acomp0/Acomp1/PDS_Timer/GPIO/IRRX/DM/USB            |
   +--------------+------------------------------------------------------------------------+
   |PDS2          |AON_PIN/BOR/RTC/Pir/Acomp0/Acomp1/PDS_Timer/GPIO/IRRX/DM/USB/WIFI       |
   +--------------+------------------------------------------------------------------------+
   |PDS3          |AON_PIN/BOR/RTC/Pir/Acomp0/Acomp1/PDS_Timer/GPIO/IRRX/DM/USB            |
   +--------------+------------------------------------------------------------------------+
   |PDS7          |AON_PIN/BOR/RTC/Pir/Acomp0/Acomp1/PDS_Timer/GPIO/IRRX/DM                |
   +--------------+------------------------------------------------------------------------+
   |PDS15         |AON_PIN/BOR/RTC/Pir/Acomp0/Acomp1/PDS_Timer                             |
   +--------------+------------------------------------------------------------------------+
   |HBN0          |AON_PIN/BOR/RTC/Pir/Acomp0/Acomp1                                       |
   +--------------+------------------------------------------------------------------------+
   |HBN1          |AON_PIN/RTC                                                             |
   +--------------+------------------------------------------------------------------------+
   |HBN2          |AON_PIN                                                                 |
   +--------------+------------------------------------------------------------------------+
   |HBN3          | Reapply power to VDDIO2                                                |
   +--------------+------------------------------------------------------------------------+

Power Modes
------------
**Operating mode**

The chip provides independent clock control between CPU and peripherals. The GLB and clock sections detail the clock control of each module. The software can perform clock control over CPUs or peripherals that are not in use according to the current application scenario.
The clock control provides real-time response, so there is no need to worry about the response time in this working mode.

**Power-down sleep mode**

The power-down mode consumes less power than the working mode. In the PDS mode, the clocks other than RTC will be controlled and switched to the internal low-speed clocks, and the external crystal oscillator and PLL will be turned off to save more power, so there will be a time delay when entering and exiting this low-power mode. After the power-down sleep mode is enabled, the data in the OCRAM area can automatically switch to the "retention" state and remain, and automatically exit this state after wake-up.

1. Enter Idle Power Saving Mode

The software can make this module enter the power-down mode through PDS configuration, waiting for processing. After entering the wait for interrupt (WFI) mode, PDS will trigger the clock control module to perform the gate clock operation, and notify the analog circuit to turn off the PLL and external crystal oscillator

2. Exit Idle Power Saving Mode

There are two ways to exit the idle power saving mode. One is that a specific interrupt or event stops the idle state. The other is that the time in PDS_TIM set by the software is met. Both will trigger PDS to exit the power-down mode. Considering that it takes about 1 ms to turn on the crystal oscillator, PDS allows software to turn on the crystal oscillator in advance, to wake up PDS faster.
When PDS is ready to wake up, it will notify CPU to exit the WFI mode through interrupt.

**HBN**

In the sleep mode, most of the chip logic is powered off (Vcore) while the AON power is kept ON, and the internal circuit will not wake up until an external event is received.
This mode can achieve ultimate power saving, but it takes the longest response time compared with the first two modes, so it suits the case where the chip does not need to work for a long time, to prolong the battery life.
As most circuits will be powered off in this mode, the corresponding register values and memory data will disappear. Therefore, there is a 4 KB HBN_RAM reserved in HBN that will not be powered off in sleep state. The data or state that the software needs to save can be copied to this memory before the chip enters the sleep mode. When recovering from the sleep mode, the chip can access data directly from RAM, which can usually be used as a record of state or a quick data recovery.

IO Retention
------------

IO retention includes AON_IO retention and PDS_IO retention. In the PDS 1/2/3/7 mode, for the chip's MISC domain is still powered, GPIO can be controlled by the glb register. After the glb register is powered off, AON_CTRL and PDS can control the IE/PD/PU of AON_IO and PDS_IO.


**AON_IO**

AON_IO refers to GPIO16/17/18/19 . GPIO16/17 can be used as XTAL32K input and output.

When reg_en_aon_ctrl_gpio is 1, reg_en_aon_ctrl_gpio[3:0] controls whether GPIO16/17/18/19 are controlled by AON_HW. When reg_en_aon_ctrl_gpio is 1, the pull-up enable of AON_IO is controlled by reg_aon_gpio_pu, the pull-down enable is controlled by reg_aon_gpio_pd, IE/SMT is controlled by reg_aon_gpio_ie_smt, and OE is controlled by reg_aon_gpio_oe.

1. Hardware IO retention
HBN can control the IE/PD/PU/OE/O of AON_IO to achieve IO retention.
When reg_en_aon_ctrl_gpio is 1, the pull-up enable of AON_IO is controlled by reg_aon_pad_pu, the pull-down enable is controlled by reg_aon_pad_pd, OE is controlled by reg_aon_pad_oe, and IE/SMT is controlled by reg_aon_pad_ie_smt.
For example, when reg_en_aon_ctrl_gpio is 0 and reg_aon_pad_pu is 1, the pull-up function cannot be implemented; when reg_aon_gpio_ie_smt is 1 and reg_aon_pad_pu is 1, the pull-up function can be implemented.

2. Software IO keep
After setting reg_aon_gpio_iso_mode to 1, when entering HBN mode, AON PAD can keep OE\O, but PU\PD cannot keep it; after HBN wakes up, AON PAD state will still keep, you need to set reg_aon_gpio_iso_mode to 0 before leaving IO On hold.
For example, GPIO16 keeps a high level in HBN mode, it needs to be configured as a normal IO function first, then use the glb register to configure it to output a high level, and finally configure reg_aon_gpio_iso_mode to 1, then enter the HBN mode.

**PDS_IO**

PDS_IO refers to other GPIOs except AON_IO, a total of 30 GPIOs, divided into 2 groups:

- GPIO0~15
- GPIO20~33
Note that GPIO16~19 can be controlled by PDS to achieve software IO retention, but cannot achieve hardware IO retention.

1. Hardware IO retention
The IE/PD/PU of PDS_IO can be controlled by the pds_gpio_i_set register, the same group of GPIOs must maintain the same level.
For example, if GPIO0 is configured as a pull-up, then GPIO8 is also configured as a pull-up.

2. Software IO keep
When cr_pds_gpio_iso_mode is 1, after entering PDS7 mode, if cr_pds_gpio_kee_en[0], [1], [2] are 1, GPIO0~15, GPIO20~33 (excluding GPIO21/22/28/29), GPIO16~19 respectively enter the GPIO hold state. After the PDS wakes up, the PDS_IO state will still be maintained. You need to set cr_pds_gpio_iso_mode to 0 before leaving the IO hold state.
The advantage of this IO retention method is that the same group of GPIOs can be kept at different levels.

.. only:: html

   .. include:: pds_register.rst

.. raw:: latex

   \input{../../en/content/pds}

.. only:: html

   .. include:: HBN_register.rst

.. raw:: latex

   \input{../../en/content/HBN}