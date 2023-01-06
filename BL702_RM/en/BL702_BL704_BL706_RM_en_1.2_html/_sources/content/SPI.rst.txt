===========
SPI
===========

SPI introduction
================
Serial Peripheral Interface Bus(SPI) is a synchronous serial communication 
interface specification for short-range communication. Devices use 
full-duplex mode for communication. There is a master and one or more slaves. 
Requires at least 4 wires, in fact 3 wires are also available
(the one-way transmission), including SDI (data input), 
SDO (data output), SCLK (clock), CS (chip select).

SPI main features
===================
- Can be used as SPI master or SPI slave 
- The transmit and receive channels each have a FIFO with a depth of 4 frames 
- Both master and slave devices support 4 clock formats(CPOL,CPHA)
- Both master and slave devices support 1/2/3/4 byte transmission mode
- Flexible clock configuration, support up to 40M clock
- Configurable MSB/LSB priority transmission
- Acceptance filtering function
- Timeout mechanism under the slave
- Support DMA transfer mode

SPI function description
==========================
Clock control
--------------
According to different clock phases and polarity settings, the SPI clock has 
four modes, which can be set by bit4 (CPOL) and bit5 (CPHA) of the SPI_CONFIG 
register. CPOL is used to determine the level of the SCK clock signal when 
idle, CPOL = 0 means the idle level is low, and CPOL = 1 means the idle level 
is high. CPHA is used to determine the sampling time. CPHA = 0 samples on the 
first clock edge of each cycle, and CPHA = 1 samples on the second clock edge 
of each cycle.

By setting registers SPI_PRD_0 and SPI_PRD_1, you can also adjust the start 
and end level duration of the clock, the time of phase 0/1, and the interval 
between each frame of data. The specific settings in the four modes are shown 
below:

.. figure:: ../../picture/SPIClockPrd.svg
   :align: center

   SPI clock

The meaning of each number is as follows:
1 is the length of the start condition, 2 is the length of the stop condition, 
3 is the length of phase 0, 4 is the length of phase 1, and 5 is the interval 
between each frame of data.

Master continuous transmission mode
-------------------------------------
When this mode is enabled, the CS signal will not be released when the current data is transmitted and there is still data available in the FIFO.

Acceptance filtering function
-------------------------------
By setting the start and end bits that need to be filtered out, the SPI discards the corresponding data segment in the received data. As shown below:

.. figure:: ../../picture/SPIIgnore.svg
   :align: center

   SPI Ignore waveform

In the figure above, the start bit of the filter is set to 0, the end bit is set to 7, the dummy byte is received, and the end bit is set to 15, the dummy byte is discarded.

Receive error correction
----------------------------
By enabling this function and setting the threshold, the SPI will discard data that does not reach the threshold width.

Slave mode timeout mechanism
--------------------------------
By setting a timeout threshold, an interrupt will be triggered when the SPI does not receive a clock signal after exceeding this time value in slave mode.

I/O transfer mode
------------------
The chip communications processor can perform FIFO fill and empty operations in response to interrupts from the FIFO. Each FIFO has a programmable FIFO trigger threshold to trigger interrupts. When the value in the RX FIFO exceeds the RX FIFO trigger threshold in the SPI controller 1, an interrupt will be generated and a signal will be sent to the chip communication processor to clear the RX FIFO. When the value in the TX FIFO is less than or equal to the TX FIFO trigger threshold in the SPI control register 1 plus 1, an interrupt will be generated and a signal will be sent to the chip communication processor to refill the TX FIFO.

Query the SPI status register to determine the sampled value in the FIFO and the status of the FIFO. Software is responsible for ensuring the correct RX FIFO trigger threshold and TX FIFO trigger threshold to prevent receive FIFO overrun and transmit FIFO underrun.

DMA transfer mode
-------------------
SPI supports DMA transfer mode. The use of this mode requires the TX and RX FIFO thresholds to be set separately. When this mode is enabled, the UART will check the TX / RX FIFO. Once the TX / RX FIFO available count value is greater than its set threshold, a DMA request will be initiated , DMA will move data to TX FIFO or out of RX FIFO according to the setting.

SPI interrupt
-------------
SPI has a variety of interrupt control, including the following interrupt modes:

- SPI transfer end interrupt
- TX FIFO request interrupt
- RX FIFO request interrupt
- Slave mode transfer timeout interrupt
- Slave mode TX overload interrupt
- TX / RX FIFO overflow interrupt

In master mode, the SPI transfer end interrupt is triggered at the end of each frame of data transfer; in slave mode, the SPI transfer end interrupt is triggered when the CS signal is released. The TX / RX FIFO request interrupt will be triggered when its available FIFO count is greater than its set threshold. When the condition is not met, the interrupt flag will be automatically cleared. Slave mode transmission timeout interrupt is triggered when the threshold is exceeded in slave mode and no clock signal is received. If the TX / RX FIFO overflows or underflows, the TX / RX FIFO overflow interrupt will be triggered. When the FIFO clear bit TFC / RFC is set to 1, the corresponding FIFO will be cleared and the overflow interrupt flag will be automatically cleared.

Query the interrupt status through register SPI_INT_STS and write 1 to the corresponding bit to clear the interrupt.

.. only:: html

   .. include:: spi_register.rst

.. raw:: latex

   \input{../../en/content/spi}