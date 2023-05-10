============
Product use
============

Moisture Sensitivity Level(MSL)
=====================================

The moisture sensitivity level of the chip is: MSL3. After the vacuum package is opened, it needs to be used up within 168 hours (7 days) at ≤30°C/60%RH, otherwise it needs to be baked and put online.

For baking temperature and time, please refer to IPC/JEDECJ-STD-033B01.

.. table::  Reference Conditions for Drying Mounted or Unmounted SMD Packages
            (User Bake: Floor life begins counting at time = 0 after bake)

    +--------------+--------+-----------+------------+-----------+------------+-----------+------------+
    | Package Body | Level  |    Bake @ 125°C        |    Bake @ 90°C         |    Bake @ 40°C         |
    +              +        +                        +                        +                        +
    |              |        |                        | ≤5% RH                 | ≤5% RH                 |
    +              +        +-----------+------------+-----------+------------+-----------+------------+
    |              |        | Exceeding | Exceeding  | Exceeding | Exceeding  | Exceeding | Exceeding  |
    +              +        +           +            +           +            +           +            +
    |              |        | Floor Life| Floor Life | Floor Life| Floor Life | Floor Life| Floor Life |
    +              +        +           +            +           +            +           +            +
    |              |        | by >72 h  | by ≤72 h   | by >72 h  | by ≤72 h   | by >72 h  | by ≤72 h   |
    +==============+========+===========+============+===========+============+===========+============+
    | Thickness    | 2      | 5 hours   | 3 hours    | 17 hours  | 11 hours   | 8 days    | 5 days     |
    +              +--------+-----------+------------+-----------+------------+-----------+------------+
    | ≤1.4 mm      | 2a     | 7 hours   | 5 hours    | 23 hours  | 13 hours   | 9 days    | 7 days     |
    +              +--------+-----------+------------+-----------+------------+-----------+------------+
    |              | 3      | 9 hours   | 7 hours    | 33 hours  | 23 hours   | 13 days   | 9 days     |
    +              +--------+-----------+------------+-----------+------------+-----------+------------+
    |              | 4      | 11 hours  | 7 hours    | 37 hours  | 23 hours   | 15 days   | 9 days     |
    +              +--------+-----------+------------+-----------+------------+-----------+------------+
    |              | 5      | 12 hours  | 7 hours    | 41 hours  | 24 hours   | 17 days   | 10 days    |
    +              +--------+-----------+------------+-----------+------------+-----------+------------+
    |              | 5a     | 16 hours  | 10 hours   | 54 hours  | 24 hours   | 22 days   | 10 days    |
    +--------------+--------+-----------+------------+-----------+------------+-----------+------------+

Electro-Static discharge（ESD）
==================================
- Human Body Model(HBM): 2000V
- Charged-Device Model(CDM): 500V

Reflow Profile
==============================
For details, please refer to IPC/JEDEC J-STD-020E.

.. figure:: ../../picture/ClassificationProfile.svg
   :align: center

   Classification Profile (Not to scale)

.. table:: Classification Reflow Profiles

    +--------------------------------------------------------------------+----------------------------------+-----------------------------------+
    |  Profile Feature                                                   | Sn-Pb Eutectic Assembly          | Pb-Free Assembly                  | 
    +====================================================================+==================================+===================================+
    | Preheat/Soak                                                       |                                  |                                   | 
    +                                                                    +                                  +                                   +
    | Temperature Min (T\ :sub:`smin`\)                                  | 100 °C                           | 150 °C                            |
    +                                                                    +                                  +                                   +
    | Temperature Max (T\ :sub:`smax`\)                                  | 150 °C                           | 200 °C                            |
    +                                                                    +                                  +                                   +
    | Time (t\ :sub:`s`\) from (T\ :sub:`smin`\ to T\ :sub:`smax`\)      | 60-120 seconds                   | 60-120 seconds                    |
    +--------------------------------------------------------------------+----------------------------------+-----------------------------------+
    | Ramp-up rate (T\ :sub:`L`\ to T\ :sub:`p`\)                        | 3 °C/second max.                 |  3 °C/second max.                 | 
    +--------------------------------------------------------------------+----------------------------------+-----------------------------------+
    | Liquidous temperature (T\ :sub:`L`\)                               | 183 °C                           | 217 °C                            |
    +                                                                    +                                  +                                   +
    | Time (t\ :sub:`L`\) maintained above T\ :sub:`L`\                  | 60-150 seconds                   | 60-150 seconds                    |
    +--------------------------------------------------------------------+----------------------------------+-----------------------------------+
    | Peak package body temperature (T\ :sub:`p`\)                       | 240 °C+0/-5 °C                   | 250 °C+0/-5 °C                    |
    +--------------------------------------------------------------------+----------------------------------+-----------------------------------+
    | Time (t\ :sub:`p`\)* within 5 °C of the specified                  | 10-30 seconds                    | 20-40 seconds                     |
    +                                                                    +                                  +                                   +
    | classification temperature (T\ :sub:`c`\)                          |                                  |                                   |
    +--------------------------------------------------------------------+----------------------------------+-----------------------------------+
    | Ramp-down rate (T\ :sub:`p`\ to T\ :sub:`L`\)                      | 6 °C/second max                  | 6 °C/second max                   |
    +--------------------------------------------------------------------+----------------------------------+-----------------------------------+
    | Time 25 °C to peak temperature                                     | 6 minutes max                    | 8 minutes max                     |
    +--------------------------------------------------------------------+----------------------------------+-----------------------------------+
    | \- Tolerance for peak profile temperature (Tp) is defined as a supplier minimum and a user maximum.                                       |
    +--------------------------------------------------------------------+----------------------------------+-----------------------------------+





