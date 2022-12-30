===========
AudioDAC
===========

Overview
=====
A PWM modulation module is built in the chip to output analog signals to drive the speaker, so as to realize audio playback.

Features
===========
- One 16-bit DAC PWM is integrated, supporting one analog PWM differential output
  * Sampling rate: 8k–48k
  * Signal to noise ratio (AW): 95 dB gain
  * Harmonic distortion + noise: 70dB @ 0dB gain
- Independent digital volume control that supports soft volume adjustment/mute
- 32-bit FIFO with a depth of 32
- Supports DMA transfer mode
- Interpolation filter 
- Stereo data mixer selector
- Linkage with a traditional DAC

Clock Tree
====================
The user starts the Audio PLL to select the corresponding frequency value. After one frequency division, the user enters the module and selects the division factor through the au\_pwm\_mode in audac\_0. The value of this register and the frequency division ratio are shown as follows.

.. figure:: ../../picture/clockTreePWM.svg
   :align: center

   Block Diagram of Clock

Functional Description
===========

The block diagram of AudioDAC is shown as follows.

.. figure:: ../../picture/AudioDAC_Arc.svg
   :align: center

   Block Diagram of Module

AudioDAC performs DSM modulation on the data in TX FIFO after it passes the interpolation filter. Finally, it outputs a PWM signal whose duty ratio is related to TX FIFO data. Next, this signal passes through the RC low-pass filter and then can drive the speaker to play audio data.

AudioDAC Interrupt
-------------
AudioDAC supports the following interrupt control modes:

- TX FIFO request interrupt

- TX FIFO underrun interrupt

- TX FIFO overrun interrupt

- Volume adjustment complete interrupt

A TX FIFO request interrupt is generated when TX\_DRQ\_CNT in TX\_FIFO\_CTRL is greater than TX\_TRG\_LEVEL. When the condition is not met, the interrupt flag is cleared automatically.

When there is no data in TX FIFO, but the user enables TX FIFO state machine through TX\_CH\_EN in TX\_FIFO\_CTRL, the TX FIFO underrun interrupt is generated.

When the user fills in data that exceeds the maximum depth of TX FIFO, it leads to TX FIFO overflow and cause a TX FIFO overrun interrupt.

Audio DAC volume control supports the fade-in/fade-out function. When volume adjustment is completed, the "volume adjustment complete interrupt" is generated, indicating that fade-in/fade-out adjustment is completed.

FIFO Format Control
--------------
AUPWM\_TX\_FIFO\_CTRL can control the format of the audio data to be stored in FIFO.

The FIFO controller supports the following four data storage formats, which are determined by FIFO\_CTRL\[25:24].

 - Mode 0:

    DATA[15:0] = {FIFO[31:16]}

 - Mode 1:

    DATA[15:0] = {FIFO[23:8]}}

 - Mode 2:

    DATA[15:0] = {FIFO[19:4]}

 - Mode 3:

    DATA[15:0] = {FIFO[15:0]}


Distribution of MSB

- Mode 0:
  
  The MSB of data is 31 bits

- Mode 1:
  
  The MSB of data is 23 bits

- Mode 2:
  
  The MSB of data is 19 bits

- Mode 3:
  
  The MSB of data is 15 bits

When the audio file to be played is 16-bit wide, select Mode3. The maximum resolution of DAC is 16-bit. The significance of other modes is that if the width of the audio file to be played is 32/24/20-bit, the user needs to make a trade-off, and cut out some information of low bits, to ensure that the next-stage circuit gets the 16-bit data. Here, the data of low bits is discarded by default.

Startup of FIFO and DMA Transfer
------------------------
The data in TX FIFO of AWPWM can be transferred by DMA.

The user can obtain the current amount of valid data in FIFO in real time through the register PDM\_TX\_FIFO\_STATUS.

The FIFO count threshold (8/16/32) for initiating DMA request is selected by configuring FIFO\_CTRL\[15:14], or can be determined by FIFO\_CTRL\[22:16].

When the count value is greater than the set threshold, and the FIFO of the channel corresponding to PDM\_TX\_FIFO\_CTRL\[12:8] is enabled, a DMA transfer is initiated.

When TX FIFO is started, if there is no valid data in TX FIFO, the tx underrun error will be triggered. Therefore, the software configuration sequence must be followed.

Audio Channel Selector
----------------------------
Audio DAC only supports the playback of one channel of audio data. To play stereo audio, you need to use the audio channel selector module. The typical application of this module is to transfer the audio files from memory to TX FIFO through DMA to play stereo audio using Audio DAC. At this time, the user only needs to transfer the stereo data to TX FIFO in turn using DMA, and can select Play Left Channel, Play Right Channel, Sum of Two Channels' Data or Average of Two Channels' Data through the register dac\_mix\_sel in audac\_1. This function is used together with the tx\_ch\_en in audac\_fifo\_ctrl, as shown below.

.. figure:: ../../picture/audacMixer.svg
   :align: center

   Mixer

The above figure shows how to fill the data transferred using DMA in turn in the L and R caches through tx\_ch\_en. When the playback source is mono, you can select tx\_ch\_en=2'01 or tx\_ch\_en=2'10, and the data will be filled in R/L in turn. The data will not be updated to the other L/R. At this time, you can select the corresponding channel through a mixer selector to go to the next-stage modulation circuit.

When the playback source is stereo, select tx\_ch\_en=2'11. At this time, the data transferred using DMA will be filled in L and R caches in turn, ensuring that both L and R will be updated. The storage format of stereo audio data source is also in the form of "L, R, L, R" arranged in turn, so the corresponding audio data will be correctly updated to L and R. At this time, you can choose which channel you need through a mixer selector, or sum or average the two channels' data to go to the next-stage modulation circuit.

Pay attention to the selection of tx\_ch\_en and mixer. If the mono data is filled in the left channel through tc\_ch\_en, but you select the right channel for the mixer, null data will be played, resulting in an error.

Volume Control
--------------------------------
The user can configure the volume through the register dac\_s0\_volume in audac\_s0. The gain range is 95.5dB18dB, and the register value\*0.5 is the true gain value. Writing "1" in the register dac\_s0\_volume\_update to update the operation after configuration.

Users can select the volume adjustment mode through dac\_s0\_mute\_softmode or dac\_s0\_ctrl\_mode. 0 indicates immediate adjustment without gradient processing; 1 means gradient adjustment in cross zero manner; 2 means gradient adjustment in a ramp manner. The registers dac\_s0\_mute\_rmpup\_rate and dac\_s0\_ctrl\_zcd\_rate adjust the slope of ramp or cross zero.

ZeroDetect
-----------------------------------
AudioDAC provides the ZeroDetect function. When this function is enabled, namely audac\_zd\_0\[16]=1, the data in TX FIFO is always 0 and this closes the digital channel, with output being "0". The purpose is to reduce the noise caused by playing 0 data and make the audio system more stable.

Configuration Process
================================

1. Depending on the sampling rate of the audio to be played, select the corresponding sampling rate through audac\_0\[31:28].

2. Depending on the number of audio channels for playback, decide the configuration of the mixer register.

3. Configure the DMA to transfer data to AudioDAC TX FIFO.

4. Enable the state machine by the tx\_ch\_en in audac\_fifo\_ctrl, and start playing.

5. Adjust the volume during playback (optional)

.. only:: html

   .. include:: audac_register.rst

.. raw:: latex

   \input{../../en/content/audac}