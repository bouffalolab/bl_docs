===========
IR
===========

Overview
=====
Infrared Remote Control (IR) is a wireless and non-contact control technique that features strong anti-interference, reliable information transmission, low power consumption, and low cost. The IR radiating circuit uses the IR LED to emit the modulated infrared waves. The receiving circuit consists of infrared receiving diodes, triodes or silicon photocells, which convert the infrared light emitted by the infrared transmitter into corresponding electrical signals, and then send them to the post amplifier.

Features
=========
- Receives data through the fixed NEC and RC5 protocols

- Receives data in any format by pulse width counting

- With powerful infrared waveform editing capability: It can emit waveforms conforming to various protocols

- Up to 15-gear power settings to adapt to different power consumption requirements

- Receives up to 64-bit data

- Sends up to 128-bit data in the non-free mode and continuously sends the data of any length in the free mode

- 4*4 byte TX FIFO, 64*2 byte RX FIFO

- Adjustable width of the TX FIFO in the free mode (8/16/24/32-bit)

- Programmable carrier frequency and duty ratio

- Maximum operating frequency of 32 MHz

- Supports the DMA sending mode

- End of sending and receiving interrupts

Functional Description
===========
Fixed Protocol Based Receiving
-------------
IR receiving supports the fixed NEC and RC5 protocols.

- NEC Protocol

Logical '1' and logical '0' waveforms of the NEC protocol are shown as follows:

.. figure:: ../../picture/IRNecLogical.svg
   :align: center

   NEC Logic Waveform

Logical '1' is 2.25 ms and the pulse time is 560 us. Logical '0' is 1.12 ms and the pulse time is 560 us.
The format of the NEC protocol is shown as follows:

.. figure:: ../../picture/IRNec.svg
   :align: center

   NEC Protocol Waveform

The head pulse is a 9 ms high-level pulse and a 4.5 ms low-level pulse, followed by an 8-bit address code and its radix-minus-one complement (diminished radix complement), then an 8-bit command code and its radix complement, and the tail pulse is a 560 us high-level pulse and a 560 us low-level pulse.

- RC5 Protocol

Logical '1' and logical '0' waveforms of the RC5 protocol are shown as follows:

.. figure:: ../../picture/IRRc5Logical.svg
   :align: center

   RC5 Logic Waveform

The logical '1' is 1.778 ms, first the 889 us low level and then the 889 us high level. Logical '0' is opposite to the logical 1 waveform.
The format of the RC5 protocol is shown as follows:

.. figure:: ../../picture/IRRc5.svg
   :align: center

   RC5 Protocol Waveform

The first two bits are the Start bits, both logical '1'. The third bit is the toggle bit, which toggles each time a key is released and pressed again. The next 5 bits are address bits and the next 6 bits are command bits.

It is worth noting that to improve the receiving sensitivity, the common infrared integrated receiver outputs a low level after receiving a high level, so you need to enable the receiving toggle function when using the IR receiving function.

Pulse width receiving
-------------
For data in any format other than the NEC and RC5 protocols, IR will count the duration of each high or low level in turn by its clock, and then store the data into the RX FIFO with a depth of 64 and a width of 2 bytes.

Normal sending mode
-------------
You can configure the head pulse, tail pulse, logical '0' pulse, and logical '1' pulse depending on protocols. During setting, you need to calculate the common pulse width unit—the greatest common divisor—of various pulses with different widths in the used protocol, fill it in the low 12 bits in the register IRTX_PULSE_WIDTH, and then fill the corresponding multiple of each pulse in the register IRTX_PW.

Then, you can fill the logic value to be sent into the TX FIFO with a depth of 4 and a width of 4 bytes.

Pulse width sending
-------------
IR provides a way of sending pulse width for the protocols that are unsuitable for the normal sending mode. You need to calculate the common pulse width unit—the greatest common divisor—of various pulses with different widths in the used protocol, and fill it in the low 12 bits in the register IRTX_PULSE_WIDTH. Then, for the multiples (8-bit per multiple) corresponding to each level width from the first high level to the last level, every four multiples are combined into a 32-bit number for padding in the TX FIFO.

Carrier modulation
-------------
Setting the high 16 bits of the register IRTX_PULSE_WIDTH can generate carriers with different frequencies and duty ratios. The <TXMPH1W> bit in the register is set to the width of carrier phase 1, while the <TXMPH0W> bit is set to the width of carrier phase 0.

Free mode
-------------
In the free mode, there is no limit to the total length of data to be sent. When there is data in TX FIFO, it will be sent, so it can be used with DMA. The width of TX FIFO in free mode can be configured to 8/16/24/32-bit.

DMA mode
-------------
IR TX supports the DMA mode, which requires to enable the free mode and set the threshold of TX FIFO by configuring <TFITH> in the register IRTX_FIFO_CONFIG_1.

When this mode is enabled, if <TFICNT> in the IRTX_FIFO_CONFIG_1 is greater than <TFITH>, the DMA TX request will be triggered. After DMA is configured, when receiving this request, DMA will transfer data from memory to TX FIFO as configured.

IR Interrupt
-------------
There are separate end of sending and receiving interrupts for IR, and when a sending action ends, a TX interrupt will be generated. When a piece of data is received, it will wait for the level duration to reach the preset end threshold before generating a receiving interrupt.

The register IRTX_INT_STS can query the TX interrupt status and clear the interrupt, and the register IRRX_INT_STS can query the receiving interrupt status and clear the interrupt.

.. only:: html

   .. include:: ir_register.rst

.. raw:: latex

   \input{../../en/content/ir}