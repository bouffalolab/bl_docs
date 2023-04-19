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

- Sleep control (PDS): 4 levels (PDS 1/2/3/7), large-scale power saving, moderate response speed

- Deep sleep control (HBN): 4 levels (HBN 0/1/2/3), global power saving, and long response time

Functional Description
==========

Power Domain
-----------
There are 7 power domains in the BL808 chip, with main functions as follows:

- PD_AON_RTC

   - RC32K/XTAL32K control register is reserved

   - RTC Counter clock source selection function

   - RTC can be used for wake-up or LED flashing

- PD_AON

   - HBN state machine controls power supply/isolation/reset/clock

   - Maintain internal voltage output selection

   - AON_PIN (GPIO9/10/11/12/13/40/41) pin wake-up control

   - HBN_OUT0_PIR (Acomp0/Acomp1/bor/pir) wake-up mask, enable register, and interrupt status register

   - BOR (Brown Out Reset) function

- PD_AON_HBNCORE

   - Partial power control register

   - 4 KB HBN_RAM, which is used to save program data before the PDS/HBN mode is enabled, so that data will not disappear in these modes

   - PIR digital control: PIR is a Passive Infra-Red (PIR) sensor, a peripheral in HBN area, which can be used as HBN wake-up source

   - AON_PIN control and IO retention function in HBN/PDS mode

   - Acomp0 and Acomp1 configuration, GPADC clock source selection and configuration

- PD_CORE

   - HBN state machine controls power supply/isolation/reset/clock

   - Reserved 64 KB RAM

   - WIFI/BLE timer control

   - 160KB WRAM/EM

- PD_CORE_MISC_DIG

   - M0 CPU and its peripherals

   - Chip's global register

- PD_USB

   - USB digital controller
- PD_MM

   - D0 CPU and its peripherals, PSRAM
   - Codec
   - VRAM 32~96KB
   - BLAI

Each power domain is controlled by 9 different power modes as shown below:

.. table:: Power mode

    +--------+------------+------------+--------------+----------------+------------+------------------+------------+------------+
    |        |            | Power Domain                                                                                         |
    +        +            +------------+--------------+----------------+------------+------------------+------------+------------+
    | NO.    | Scenario   | PD_AON_RTC |   PD_AON     | PD_AON_HBNCORE | PD_CORE    | PD_CORE_MISC_DIG | PD_USB     | PD_MM      |
    +========+============+============+==============+================+============+==================+============+============+
    | 1      | Normal     |    ON      |     ON       |     ON         |     ON     |     ON           | ON         |     ON     |
    +--------+------------+------------+--------------+----------------+------------+------------------+------------+------------+
    | 2      | PDS1       |    ON      |     ON       |     ON         |    ON      |     ON           | ON         |     OFF    |
    +--------+------------+------------+--------------+----------------+------------+------------------+------------+------------+
    | 3      | PDS2       |    ON      |     ON       |     ON         |    ON      |     ON           | OFF        |     ON     |
    +--------+------------+------------+--------------+----------------+------------+------------------+------------+------------+
    | 4      | PDS3       |    ON      |     ON       |     ON         |    ON      |     ON           | OFF        |     OFF    |
    +--------+------------+------------+--------------+----------------+------------+------------------+------------+------------+
    | 5      | PDS7       |    ON      |     ON       |     ON         |    ON      |     OFF          | OFF        |     OFF    |
    +--------+------------+------------+--------------+----------------+------------+------------------+------------+------------+
    | 6      | HBN0       |    ON      |     ON       |     ON         |    OFF     |     OFF          | OFF        |     OFF    |
    +--------+------------+------------+--------------+----------------+------------+------------------+------------+------------+
    | 7      | HBN1       |    ON      |     ON       |    OFF         |    OFF     |     OFF          | OFF        |     OFF    |
    +--------+------------+------------+--------------+----------------+------------+------------------+------------+------------+
    | 8      | HBN2       |    ON      |     OFF      |     OFF        |    OFF     |     OFF          | OFF        |     OFF    |
    +--------+------------+------------+--------------+----------------+------------+------------------+------------+------------+
    | 9      | HBN3       |    OFF     |     OFF      |     OFF        |    OFF     |     OFF          | OFF        |     OFF    |
    +--------+------------+------------+--------------+----------------+------------+------------------+------------+------------+

.. note::
   
   - To enter the HBN2 mode, the chip needs to enter the HBN1 mode first, and then turn VDDIO2 off
   - In the HBN2 mode, only RTC_Timer is kept, but no wake-up is provided. To quit the HBN2 mode, you only need to resupply power to VDDIO2.
   - VDD33_RTC/VDDIO2(AON) share pin encapsulation (QFN88 Type2, QFN68), without HBN2 mode
   - The pu_chip must be powered off before entering the HBN3 mode. In this mode, most devices will be powered off, but you can set whether to power RTC off through rtc_pu_chip_sel.

If rtc_pu_chip_sel is 1, the RTC power supply is always maintained. If that is 0, it is controlled by pu_chip. The value of rtc_pu_chip_sel is determined at encapsulation.

Wake-up Source
------------
The chip supports multiple wake-up sources, which can wake up the chip from different power modes.

PDS 1/2/3 can be wake up in the following ways:

- HBN wake-up source

- All GPIO wake-up

- Infrared receiver

- BLE wake-up event

- WIFI wake-up event

- PDS timer

The wake-up sources of other power modes are shown as follows:

.. table:: Wakeup source

    +--------------+-----------------------------------------------------------+
    | Power Mode   | Wake-up Source                                            |
    +==============+===========================================================+
    |PDS7          |PDS timer/PSD_PIN/RTC/AON_PIN/BOR/Pir/Acomp0/Acomp1        |
    +--------------+-----------------------------------------------------------+
    |HBN0          |RTC/AON_PIN/BOR/Pir/Acomp0/Acomp1                          |
    +--------------+-----------------------------------------------------------+
    |HBN1          |RTC/AON_PIN                                                |
    +--------------+-----------------------------------------------------------+
    |HBN2          |Resupply power to VDDIO2                                   |
    +--------------+-----------------------------------------------------------+
    |HBN3          |Resupply power to pu_chip                                  |
    +--------------+-----------------------------------------------------------+


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
As most circuits will be powered off in this mode, the corresponding register values and memory data will disappear. Therefore, there is a 4 KB HBN_RAM reserved in HBN that will not be powered off in the HBN0 mode. The data or state that the software needs to save can be copied to this memory before the chip enters the sleep mode. When recovering from the sleep mode, the chip can access data directly from RAM, which can usually be used as a record of state or a quick data recovery.

IO Retention
------------

IO retention includes AON_IO retention and PDS_IO retention. In the PDS 0/1/2/3 mode, for the chip's MISC domain is still powered, GPIO can be controlled by the glb register. After the glb register is powered off, AON_CTRL and PDS can control the IE/PD/PU of AON_IO and PDS_IO.

**AON_IO**

AON_IO refers to GPIO9/10/11/12/13/14/15/40/41. GPIO40/41 is used as the input and output of XTAL32K by default (it can only be used as AON_IO if it is changed to other pinmux purposes). When the IE/OE of GPIO40/41 is all set to 0, GPIO40/41 multiplexes the analog function XTAL32K_INXTAL32K_OUT. Contrarily, when that is not all set to 0, GPIO40/41 is used as a normal IO function. For example, when GPIO40 is used as a general IO function, reg_en_aon_ctrl_gpio\[7\] and reg_aon_pad_oe\[7\] are set to 0, and reg_aon_pad_ie_smt\[7\] is set to 1.

1. Hardware's IO retention: HBN can control the IE/PD/PU/OE/O of AON_IO, thus achieving IO retention.
   When reg_en_aon_ctrl_gpio is 1, reg_aon_gpio_pu controls the pull-up enable of AON_IO and reg_aon_gpio_pd controls the pull-down enable. The reg_aon_gpio_oe controls OE, and aon_led_out\[0\], \[1\], and \[2\] control AON PAD O, respectively. Whereas, IE/SMT can be controlled by reg_aon_gpio_ie_smt no matter the value of reg_en_aon_ctrl_gpio is 0 or 1. For example, when reg_en_aon_ctrl_gpio is 0, even if reg_aon_pad_pu is 1, it cannot enable pull-up. But setting reg_aon_gpio_ie_smt to 1 can enable the IE function.

2. Software IO retention: After reg_aon_gpio_iso_mode is set to 1, AON PAD can retain OEO, but PUPD cannot be retained when entering the HBN mode. After HBN wake-up, the AON PAD state will still be retained, and you need to clear reg_aon_gpio_iso_mode before exiting IO retention state. For example, GPIO40 keeps high level in the HBN mode, so you need to configure it as a normal IO function first, then configure it to output high level through the glb register, and finally set reg_aon_gpio_iso_mode to 1 before entering the HBN mode.


**PDS_IO**

PDS_IO refers to other 32 GPIOs except AON_IO, and they are divided into 3 groups by the physical location of PAD:

- Left: GPIO0–8

- Right: GPIO16–23

- Top: GPIO24–39

1. Hardware IO retention: The IE/PD/PU of PDS_IO can be controlled by the register pds_gpio_i\_set, and the same group of GPIOs must hold the same level. For example, if GPIO0 is configured to pull-up, GPIO8 is also configured to pull-up.

2. Software IO retention: When cr_pds_gpio_iso_mode is 1, if cr_pds_gpio_kee_en\[0\], \[1\], and \[2\] is 1 in the PDS7 mode, GPIO0–8, GPIO16–23, and GPIO24–38 enter the GPIO retention state respectively. After PDS7 is woke up, the PDS_IO state will still be retained, so you need to clear cr_pds_gpio_iso_mode to 0 before existing IO retention. The advantage of this IO retention is that the same group of GPIOs can hold different levels.

