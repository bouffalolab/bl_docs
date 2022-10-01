===========
GLB
===========

Overview
=====

Functional Description
===========
Clock Management
-------------
It is mainly used to set the clocks of processors, buses, and peripherals, set the clock source and frequency division of the above modules, and achieve the clock gating of these modules, to save power for the system. For more details, see system clock sections.

Reset Management
-------------
It provides the separate reset function of each peripheral module and the chip reset function.

Chip reset:

- CPU reset: Only CPU is reset. The program will roll back, and peripherals will not be reset
- System reset: All peripherals and CPUs will be reset, but the registers in the AON field will not be reset
- Power-on reset: The whole system including the registers in the AON field will be reset

The application can select a proper reset mode as required.

Bus Management
-------------
It provides bus arbitration and bus error settings, so that users can set whether to interrupt and provide the error bus address information when a bus error occurs, facilitating program debugging by users.

Memory Management
-------------

It provides power management of each memory module in the low-power mode of the chip system, including two setting modes:

- Retention mode: The data in the memory can be saved, but it cannot be read or written before exiting the low-power mode
- Sleep mode: This mode is only used to reduce system power consumption, because it will cause memory data to lose

