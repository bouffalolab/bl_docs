===========
PDM
===========

Overview
=====
A PDM audio processing module is built in the chip, supporting recording via the PDM interface microphone.

Features
===========
- Three 20-bit ADCs are integrated to support three digital PDM inputs.
  
  - Sampling rate: 8k–96k
  
  - Signal to noise ratio (AW): 97 dB @ 0 dB gain
  
  - Harmonic distortion + noise: -87dB @ 0dB gain
  
  - Analog pre-amplifier gain: 0dB, 6–42 dB, 3dB per gear

- Adjustable high-pass filter and independent digital volume control for ADC channel

- Supports digital PDM interface, with input GPIO multiplexed

- Independent digital volume control

- TX/RX FIFO of 32-bit width

- Supports DMA transfer mode

Functional Description
===========

The block diagram of PDM module is shown as follows.

.. .. figure:: ../../picture/PDMBasicStruct.svg
..   :align: center

..   PDM基本框图

PDM has three PDM digital interface demodulators. It passes through the decimation and HPF filters and then goes to the volume control module. Users can control mute/unmute, volume, and audio fade-in/fade-out effect through the volume control module. The recorded data is stored in the FIFO with a depth of 32, and the storage format in FIFO is controlled by the register RX\_FIFO\_CTRL\[25:24]. PDM\_RX\_FIFO\_CTRL\[5] configures the resolution of sampling. PDM only supports PDM audio digital interface input.

PDM Interrupt
-------------
PDM supports the following interrupt control modes:

- RX FIFO request interrupt

- RX FIFO underrun interrupt

- RX FIFO overrun interrupt

A RX FIFO request interrupt is generated when RX\_DRQ\_CNT in RX\_FIFO\_CTRL is greater than RX\_TRG\_LEVEL. When the condition is not met, the interrupt flag is cleared automatically.

When there is no data in RX FIFO, but the user enables RX FIFO modulation through RX\_CH\_EN in RX\_FIFO\_CTRL, the RX FIFO underrun interrupt is generated.

When the user fills in data that exceeds the maximum depth of RX FIFO, it leads to RX FIFO overflow and cause a RX FIFO overrun interrupt.

FIFO Format Control
--------------
PDM\_RX\_FIFO\_CTRL can control the format of the audio data to be stored in FIFO.

The user sets the resolution of audio by configuring PDM\_RX\_FIFO\_CTRL\[5].

When 16-bit resolution is selected, the FIFO controller supports the following four data storage formats, which are determined by FIFO\_CTRL\[25:24].

 - Mode 0:

    DATA[31:0] = {FIFO[19:14],16'h0}

 - Mode 1:

    DATA[31:0] = {8{FIFO[19]},FIFO[19:4],8'h0}

 - Mode 2:

    DATA[31:0] = {12{FIFO[19]},FIFO[19:4],4'h0}

 - Mode 3:

    DATA[31:0] = {16{FIFO[19]},FIFO[19:4]}


During recording or playing, the conversion result of 32-bit data or the digital audio source to be demodulated is on the left, denoted as DATA\[31:0]. The format of data to be stored in FIFO is shown on the right. When the actual resolution of the recording is 20-bit, if the 16-bit resolution is selected, the data with 20-bit resolution must be properly cut. Thus, the high 16 bits of the data with 20-bit resolution are selected as the final result FIFO\[19:14] and stored in the position of the high 16 bits, while the low 16 bits are filled with 0. Modes 1, 2, and 3 are expressed in the same way as Mode 0. The 8{FIFO\[19]} symbol indicates that the high 8 bits are padded with the value of bit\[19].

Therefore, when a 16-bit resolution is selected, PDM provides four modes to store the digital results of conversion/output.

In the case of 20-bit resolution, the storage formats in the four modes are as follows:

 - Mode 0:

    DATA[31:0] = {FIFO[19:0],12'h0}

 - Mode 1:

    DATA[31:0] = {8{FIFO[19]},FIFO[19:0],4'h0}

 - Mode 2:

    DATA[31:0] = {12{FIFO[19]},FIFO[19:0]}

 - Mode 3:

    DATA[31:0] = {16{FIFO[19]},FIFO[19:4]}

Distribution of MSB

 - Mode 0:

    The MSB of data is 31 bits

 - Mode 1:

    The MSB of data is 23 bits

 - Mode 2:

    The MSB of data is 19 bits

 - Mode 3:

    The MSB of data is 15 bits

Startup of FIFO and DMA Transfer
------------------------
The data in FIFO of the PDM module can be transferred by DMA.

The user can obtain the current amount of valid data in FIFO in real time through the register PDM\_RX\_FIFO\_STATUS.

The FIFO count threshold (8/16/32) for initiating DMA request is selected by configuring FIFO\_CTRL\[15:14], or can be determined by FIFO\_CTRL\[22:16].

When the count value is greater than the set threshold, and the FIFO of the channel corresponding to PDM\_RX\_FIFO\_CTRL\[12:8] is enabled, a DMA transfer is initiated.

When TX FIFO is started, if there is no valid data in TX FIFO, the tx underrun error will be triggered. Therefore, the software configuration sequence must be followed.

.. only:: html

   .. include:: pdm_register.rst

.. raw:: latex

   \input{../../en/content/pdm}