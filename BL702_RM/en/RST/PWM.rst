===========
PWM
===========

PWM introduction
=================
Pulse width modulation (PWM) is an analog control method. The bias of the base of the transistor or the gate of the MOS tube is modulated according to the change of the corresponding load to realize the change of the conduction time of the transistor or the MOS tube. So as to realize the change of the output of the switch stable power supply. This method can keep the output voltage of the power supply constant when the working conditions change, and it is a very effective technology to control the analog circuit with the digital signal of the microprocessor. It is widely used in many fields from measurement and communication to power control and conversion.

PWM main features
===================

- Support 5-channel PWM signal generation

- Three clock sources can be selected (bus clock <bclk>, crystal oscillator clock <xtal>, slow clock <32k>), with 16-bit clock divider

- Double threshold setting, increase pulse flexibility

PWM function description
==========================
Clock and divider
-------------------
There are three options for each PWM counter clock source, the sources are as follows:

A. bclk - Chip bus clock

B. XTAL - External crystal clock

C. f32k - System RTC clock

Each counter has its own 16-bit frequency divider. The PWM counter will use the divided clock as the counting cycle unit, and perform one action every time a counting cycle passes .

Pulse generation principle
--------------------------------
There is a counter in the PWM. When the counter is in the middle of two settable thresholds, the PWM output is 1, otherwise when the counter is outside the two set thresholds, the PWM output is 0. As shown below:

.. figure:: ../../picture/PwmWave.svg
   :align: center

   PWM waveform

The PWM cycle is determined by two parts, one is the clock frequency division coefficient, and the other is the clock duration.

The clock division coefficient is set by the register PWMn_CLK_DIV[15:0] (n is 0~4), which is used to divide the PWM source clock.

The clock duration is set by the register PWMn_PERIOD[15:0] (n is 0~4), which is used to set the number of divided clock cycles for a PWM cycle. That is, the period of PWM=PWM source clock/PWMn_CLK_DIV[15:0]/PWMn_PERIOD[15:0].

The duty cycle of PWM is determined by the clock duration and two thresholds. The first threshold is set by the register PWMn_THRE1[15:0] (n is 0~4), the second threshold is set by the register PWMn_THRE2[15:0] (n is 0~4), the PWM waveform will be in the first Pull up at one threshold, and pull down at the second threshold. That is, the duty cycle of PWM=(PWMn_THRE2[15:0]-PWMn_THRE1[15:0])/PWMn_PERIOD[15:0].

Example:
If the PWM clock source is selected as bclk, which is 72MHz, to generate a 1kHz, 20% duty cycle PWM wave, set as follows:

PWMn_CLK_DIV[15:0]=2

PWMn_PERIOD[15:0]=72000000/2/1000=36000

PWMn_THRE1[15:0]=0

PWMn_THRE2[15:0]=0+36000*20%=7200

PWM interrupt
----------------
For each PWM channel, you can set the cycle count value. When the cycle number of the PWM output reaches this count value, a PWM interrupt will be generated.

.. table:: Duty Cycle Parameters 

    +---------------+---------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    | frequency/MHz |                       Supported duty cycle (n is an integer, and 2 <= n <= 65535^2)                        |
    +===============+=========+========+========+========+========+========+========+========+========+========+========+========+
    | 36            |     0%  |    50% |   100% |        |        |        |        |        |        |        |        |        |
    +---------------+---------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    | 24            |     0%  | 33.33% | 66.67% |   100% |        |        |        |        |        |        |        |        |
    +---------------+---------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    | 18            |     0%  |    25% |    50% |   75%  |  100%  |        |        |        |        |        |        |        |
    +---------------+---------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    | 14.4          |     0%  |    20% |    40% |   60%  |  80%   |  100%  |        |        |        |        |        |        |
    +---------------+---------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    | 12            |     0%  | 16.67% | 33.33% |    50% | 66.67% | 83.33% | 100%   |        |        |        |        |        |
    +---------------+---------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    | 10.29         |     0%  | 14.29% | 28.57% | 42.86% | 57.14% | 71.43% | 85.71% |  100%  |        |        |        |        |
    +---------------+---------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    | 9             |     0%  | 12.50% |    25% | 37.50% |   50%  | 62.50% | 75%    | 87.50% |  100%  |        |        |        |
    +---------------+---------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    | 8             |     0%  | 11.11% | 22.22% | 33.33% | 44.44% | 55.56% | 66.67% | 77.78% | 88.89% |  100%  |        |        |
    +---------------+---------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    | 7.2           |     0%  |    10% |    20% |   30%  |  40%   |    50% |  60%   |  70%   |    80% |    90% |  100%  |        |
    +---------------+---------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    | ·             |         |        |        |        |        |        |        |        |        |        |        |        |
    +---------------+---------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    | ·             |         |        |        |        |        |        |        |        |        |        |        |        |
    +---------------+---------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    | ·             |         |        |        |        |        |        |        |        |        |        |        |        |
    +---------------+---------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    | 72/n          |     0/n |    1/n |   2/n  |  3/n   |   4/n  |  5/n   |   6/n  |  7/n   |  8/n   |   9/n  |  ···   |  n/n   |
    +---------------+---------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+

.. only:: html

   .. include:: pwm_register.rst

.. raw:: latex

   \input{../../en/content/pwm}