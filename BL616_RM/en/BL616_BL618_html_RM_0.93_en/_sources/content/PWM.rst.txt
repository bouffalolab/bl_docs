===========
PWM
===========

Overview
=====
Pulse Width Modulation (PWM), an efficient technique that uses the digital signal of microprocessor to control the analog circuit, is widely used in fields like measurement, communication, and power control and conversion.

Features
=========

- Two PWM signals are generated, each containing 4-channel PWM signal outputs, with 2 pairs of complementary PWM per channel

- There are optional three clock sources (bus clock<bclk>, crystal oscillator clock<xtal_ck>, and slow clock<32k>), with 16-bit clock divider

- Each PWM can be independently set to different cycles

- Each PWM channel has double threshold setting, allowing different duty ratios and phases, with more flexible pulse

- Each PWM channel has independent dead time setting

- Different active levels can be set for each PWM output pin

- Each PWM has an independent linked switch to connect/disconnect with the internal counter, and you can set the default output level when disconnected

- The software and external brake signals can set the PWM output level to a preset state

- Up to 9 trigger sources for triggering ADC conversion

- Interrupts will be generated when these events occur: counter overflow, threshold comparison and matching, brake signal generation, and PWM cycles reaching the set value.

Functional Description
===========
Clock and Frequency Divider
-------------
Each PWM counter has three optional clock sources:

1. bclk–bus clock of chip

2. XTAL–external crystal oscillator clock

3. f32k–RTC clock of system

Each counter has its own 16-bit frequency divider, which can divide the selected clock through APB. The PWM counter will take the divided clock as the counting cycle unit, and add one every time a counting cycle ends.

Active Level
---------
Different application scenarios require different active level states, some being active at low level and others being active at high level. The active level of each PWM output can be set independently.

When the <pwm_chy_ppl> bit in the register pwm_mcx_config1 is set to 1, it means that the forward channel of channel y (y = 0, 1, 2, 3) of pwmx (x = 0) is set to be active at high level. That is, when the PWM module outputs logical '1', its corresponding pin outputs high level, and when it outputs logical '0', that outputs low level. When <pwm_chy_ppl> is set to 0, it means that the same is set to be active at low level. That is, when the PWM module outputs logical '1', its corresponding pin outputs low level, and when it outputs logical '0', that outputs high level. The setting bit of the complementary channel is \<pwm_chy_npl\>, and its setting rule is the same as that of the forward channel.

Principle of Pulse Generation
-------------
When the <pwm_chy_pen> bit in the register pwm_mcx_config1 is set to 0, it means that the forward channel of channel y (y = 0, 1, 2, 3) of pwmx (x = 0) is set to a determined logic state, and then the state is determined by the <pwm_chy_psi> bit. When <pwm_chy_psi> is 0, the corresponding forward channel pin outputs logical '0', and when the same is 1, that outputs logical '1'. The actual output level must be jointly determined with <pwm_chy_ppl>. The setting bit of the complementary channel is <pwm_chy_nen>, and its setting rule is the same as that of the forward channel.

When the <pwm_chy_pen> (<pwm_chy_nen>) bit in the register pwm_mcx_config1 is set to 1, the forward (complementary) channel output of channel y (y = 0, 1, 2, 3) of pwmx (x = 0) is determined by comparing the internal counter with the two thresholds. If the counter value is between the two thresholds, the forward channel outputs logical '1', while the complementary one outputs logical '0'. If the counter value is beyond the two thresholds, the forward channel outputs logical '0', while the complementary one outputs logical '1'. The waveform is shown as follows.

.. figure:: ../../picture/PWMWave.svg
   :align: center

   Waveform of PWM in different configurations

Brake
-----

When the bit <pwm_sw_break_en> of pwm_mcx_config0 (x=0) is 1, the brake signal is generated, and the level of the PWM output will be modified to the preset state. The preset state is set by the bits <pwm_chy_pbs> and <pwm_chy_nbs> (y=0,1,2,3) in the pwm_mcx_config1 (x=0) register. When this bit is set to 1, after the brake signal is generated, the PWM output logic 1, when this bit is set to 0, this PWM output logic 0 after the brake signal is generated. When <pwm_sw_break_en> is 0, it exits the braking state, and the PWM resumes the previous operating mode. The brake signal does not trigger an interrupt.

Dead Zone
------
Different dead time can be set for each PWM channel independently. When the value of dead time is not 0, the forward channel of PWM will delay the generation of jump level when it matches the threshold 1, and immediately change the level state when it matches the threshold 2. The complementary channel of PWM will change the level state immediately when it matches the threshold 1, and delay the generation of jump level when it matches the threshold 2. The length of dead zone is set by the register pwm_mcx_dead_time.

Cycle and Duty Ratio Calculation
-----------------
The PWM cycle is determined the clock division factor and the clock duration cycle. The clock division factor is set by the register PWMx_CLK_DIV\[15:0\](x: 0) and is used to divide the source clock of PWM. The clock duration cycle is set by the register PWMx_PERIOD\[15:0\](x: 0\~1) and is used to set how many divided clock cycles a PWM cycle consists of. That is, PWM cycle = PWM source clock/PWMx_CLK_DIV\[15:0\]/PWMx_PERIOD\[15:0\].

PWM Interrupt
-------------
For each PWM, the cycle count value can be set by the high 16 bits of the register pwm_mcx_period. When the number of PWM cycles reaches this value, the PWM interrupt will be generated.

When the PWM count value reaches the number of cycles or it matches the threshold, the PWM interrupt will be generated.

ADC Linkage
----------
When the counter matches the threshold or the count value reaches the number of cycles, it will generate the signal that internally triggers ADC startup conversion. It should be noted that only PWM0 can trigger such conversion, while PWM1 cannot do that. The specific trigger source is set by the <pwm_adc_trg_src> bit in the register pwm_mcx_config0. This feature is commonly used in timing sampling. The following is an example.

Application scenario: In BLDC application, there is such a requirement that the current flowing through the coil shall be detected while the motor speed is controlled by PWM. In a PWM cycle, after PWM controls the power device to turn on, the current becomes stable after a certain period of time. Then, it is necessary to sample the current value, which means that there is a strict phase difference between the time point of triggering ADC conversion and PWM. For example, if the channel 0 of PWM is used to drive one of the phases of the motor, and a 10 KHz square wave with a duty ratio of 20% needs to be generated, and ADC sampling needs to be performed at the middle time point of the high level of the square wave, then the PWM cycle is 100us. When the clock source is 1 MHz, the cycle count value is 100, and the two thresholds of channel 1 can be set to 0 and 20 respectively. At this time, the counter is between 0 and 20, and PWM outputs a high level, and otherwise it outputs a low level. When the threshold L of channel 2 is set to 10 and the pwm_adc_trg_src in the register pwm_mc0_config0 is set to 4, pwm_ch2l_int can trigger ADC conversion. When the counter counts to 10, ADC will start sampling conversion at the middle time point of the high level generated by channel 1. This ensures that each sampling can meet the exact time requirement without CPU intervention, thus improving the performance.

.. only:: html

   .. include:: pwm_register.rst

.. raw:: latex

   \input{../../en/content/pwm}