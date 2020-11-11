===========
SPI
===========

Introduction
===============
Serial Peripheral Interface Bus(SPI) is a synchronous serial communication 
interface specification for short-range communication. Devices use 
full-duplex mode for communication. There is a master and one or more slaves. 
Requires at least 4 wires, in fact 3 wires are also available
(the one-way transmission), including SDI (data input), 
SDO (data output), SCLK (clock), CS (chip select).

Main features
==================
- Can be used as SPI master or SPI slave
- The transmit and receive channels each have a FIFO with a depth of 4 words
- Both master and slave devices support 4 clock formats(CPOL,CPHA)
- Both master and slave devices support 1/2/3/4 byte transmission mode
- Flexible clock configuration, support up to 40M clock
- Configurable MSB/LSB priority transmission
- Acceptance filtering function
- Timeout mechanism under the slave
- Support DMA transfer mode

Function description
=======================
Clock control
----------------
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

.. figure:: /content/Spi/picture/SPI_clock_prd.png
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
--------------------------------------
By setting the start and end bits that need to be filtered out, the SPI discards the corresponding data segment in the received data. As shown below:

.. figure:: /content/Spi/picture/SPI_ignore.png
   :align: center

   SPI ignore

In the figure above, the start bit of the filter is set to 0, the end bit is set to 7, the dummy byte is received, and the end bit is set to 15, the dummy byte is discarded.

Receive error correction
---------------------------
By enabling this function and setting the threshold, the SPI will discard data that does not reach the threshold width.

Slave mode timeout mechanism
--------------------------------
By setting a timeout threshold, an interrupt will be triggered when the SPI does not receive a clock signal after exceeding this time value in slave mode.

I/O transfer mode
---------------------
The chip communications processor can perform FIFO fill and empty operations in response to interrupts from the FIFO. Each FIFO has a programmable FIFO trigger threshold to trigger interrupts. When the value in the RX FIFO exceeds the RX FIFO trigger threshold in the SPI controller 1, an interrupt will be generated and a signal will be sent to the chip communication processor to clear the RX FIFO. When the value in the TX FIFO is less than or equal to the TX FIFO trigger threshold in the SPI control register 1 plus 1, an interrupt will be generated and a signal will be sent to the chip communication processor to refill the TX FIFO.

Query the SPI status register to determine the sampled value in the FIFO and the status of the FIFO. Software is responsible for ensuring the correct RX FIFO trigger threshold and TX FIFO trigger threshold to prevent receive FIFO overrun and transmit FIFO underrun.

DMA transfer mode
-------------------
SPI supports DMA transfer mode. The use of this mode requires the TX and RX FIFO thresholds to be set separately. When this mode is enabled, the UART will check the TX / RX FIFO. Once the TX / RX FIFO available count value is greater than its set threshold, a DMA request will be initiated , DMA will move data to TX FIFO or out of RX FIFO according to the setting.

SPI interrupt
----------------
SPI has a variety of interrupt control, including the following interrupt modes:

- SPI transfer end interrupt
- TX FIFO request interrupt
- RX FIFO request interrupt
- Slave mode transfer timeout interrupt
- Slave mode TX overload interrupt
- TX / RX FIFO overflow interrupt

In master mode, the SPI transfer end interrupt is triggered at the end of each frame of data transfer; in slave mode, the SPI transfer end interrupt is triggered when the CS signal is released. The TX / RX FIFO request interrupt will be triggered when its available FIFO count is greater than its set threshold. When the condition is not met, the interrupt flag will be automatically cleared. Slave mode transmission timeout interrupt is triggered when the threshold is exceeded in slave mode and no clock signal is received. If the TX / RX FIFO overflows or underflows, the TX / RX FIFO overflow interrupt will be triggered. When the FIFO clear bit TFC / RFC is set to 1, the corresponding FIFO will be cleared and the overflow interrupt flag will be automatically cleared.

Query the interrupt status through register SPI_INT_STS and write 1 to the corresponding bit to clear the interrupt.


Register description
==========================

+----------------------+----------------------------------+
| Name                 | Description                      |
+----------------------+----------------------------------+
| `spi_config`_        | SPI configuration register       |
+----------------------+----------------------------------+
| `spi_int_sts`_       | SPI interrupt status             |
+----------------------+----------------------------------+
| `spi_bus_busy`_      | SPI bus busy                     |
+----------------------+----------------------------------+
| `spi_prd_0`_         | SPI length control register      |
+----------------------+----------------------------------+
| `spi_prd_1`_         | SPI length of interval           |
+----------------------+----------------------------------+
| `spi_rxd_ignr`_      | SPI ingnore function             |
+----------------------+----------------------------------+
| `spi_sto_value`_     | SPI time-out value               |
+----------------------+----------------------------------+
| `spi_fifo_config_0`_ | SPI FIFO configuration register0 |
+----------------------+----------------------------------+
| `spi_fifo_config_1`_ | SPI FIFO configuration register1 |
+----------------------+----------------------------------+
| `spi_fifo_wdata`_    | SPI FIFO write data              |
+----------------------+----------------------------------+
| `spi_fifo_rdata`_    | SPI FIFO read data               |
+----------------------+----------------------------------+

spi_config
------------
 
**Address：**  0x4000a200
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                                                                                                                          |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| DEGCNT                                        | DEGEN     | RSVD      | MCEN      | IGNREN    | BYTEINV   | BITINV    | SCLKPH    | SCLKPOL   | FSIZE                 | SEN       | MEN       |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                                                                                                                                                                                                                   |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 31:16    | RSVD     |        |             |                                                                                                                                                                                                                               |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 15:12    | DEGCNT   | R/W    | 4'D0        | De-glitch function cycle count                                                                                                                                                                                                |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 11       | DEGEN    | R/W    | 1'B0        | Enable signal of all input de-glitch function                                                                                                                                                                                 |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 10       | RSVD     |        |             |                                                                                                                                                                                                                               |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 9        | MCEN     | R/W    | 1'B0        | Enable signal of master continuous transfer mode                                                                                                                                                                              |
+          +          +        +             +                                                                                                                                                                                                                               +
|          |          |        |             | 1'b0: Disabled, SS_n will de-assert between each data frame                                                                                                                                                                   |
+          +          +        +             +                                                                                                                                                                                                                               +
|          |          |        |             | 1'b1: Enabled, SS_n will stay asserted between each consecutive data frame if the next data is valid in the FIFO                                                                                                              |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 8        | IGNREN   | R/W    | 1'B0        | Enable signal of RX data ignore function                                                                                                                                                                                      |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 7        | BYTEINV  | R/W    | 1'B0        | Byte-inverse signal for each FIFO entry data                                                                                                                                                                                  |
+          +          +        +             +                                                                                                                                                                                                                               +
|          |          |        |             | 0: Byte[0] is sent out first                                                                                                                                                                                                  |
+          +          +        +             +                                                                                                                                                                                                                               +
|          |          |        |             | 1: Byte[3] is sent out first                                                                                                                                                                                                  |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 6        | BITINV   | R/W    | 1'B0        | Bit-inverse signal for each data byte                                                                                                                                                                                         |
+          +          +        +             +                                                                                                                                                                                                                               +
|          |          |        |             | 0: Each byte is sent out MSB-first                                                                                                                                                                                            |
+          +          +        +             +                                                                                                                                                                                                                               +
|          |          |        |             | 1: Each byte is sent out LSB-first                                                                                                                                                                                            |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 5        | SCLKPH   | R/W    | 1'B0        | SCLK clock phase inverse signal                                                                                                                                                                                               |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 4        | SCLKPOL  | R/W    | 1'B0        | SCLK polarity                                                                                                                                                                                                                 |
+          +          +        +             +                                                                                                                                                                                                                               +
|          |          |        |             | 0: SCLK output LOW at IDLE state                                                                                                                                                                                              |
+          +          +        +             +                                                                                                                                                                                                                               +
|          |          |        |             | 1: SCLK output HIGH at IDLE state                                                                                                                                                                                             |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 3:2      | FSIZE    | R/W    | 2'D0        | SPI frame size (also the valid width for each FIFO entry)                                                                                                                                                                     |
+          +          +        +             +                                                                                                                                                                                                                               +
|          |          |        |             | 2'd0: 8-bit                                                                                                                                                                                                                   |
+          +          +        +             +                                                                                                                                                                                                                               +
|          |          |        |             | 2'd1: 16-bit                                                                                                                                                                                                                  |
+          +          +        +             +                                                                                                                                                                                                                               +
|          |          |        |             | 2'd2: 24-bit                                                                                                                                                                                                                  |
+          +          +        +             +                                                                                                                                                                                                                               +
|          |          |        |             | 2'd3: 32-bit                                                                                                                                                                                                                  |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 1        | SEN      | R/W    | 1'B0        | Enable signal of SPI Slave function, Master and Slave should not be both enabled at the same time                                                                                                                             |
+          +          +        +             +                                                                                                                                                                                                                               +
|          |          |        |             | (This bit becomes don't-care if cr_spi_m_en is enabled)                                                                                                                                                                       |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 0        | MEN      | R/W    | 1'B0        | Enable signal of SPI Master function                                                                                                                                                                                          |
+          +          +        +             +                                                                                                                                                                                                                               +
|          |          |        |             | Asserting this bit will trigger the transaction, and should be de-asserted after finish                                                                                                                                       |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

spi_int_sts
-------------
 
**Address：**  0x4000a204
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                  | FEREN     | TXUEN     | STOEN     | RXFEN     | TXFEN     | ENDEN     | RSVD                              | TXUCLR    | STOCLR    | RSVD                  | ENDCLR    |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                  | FERMASK   | TXUMASK   | STOMASK   | RXFMASK   | TXFMASK   | ENDMASK   | RSVD                  | FERINT    | TXUINT    | STOINT    | RXFINT    | TXFINT    | ENDINT    |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                                                                                                                                                                |
+----------+----------+--------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 31:30    | RSVD     |        |             |                                                                                                                                                                            |
+----------+----------+--------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 29       | FEREN    | R/W    | 1'B1        | Interrupt enable of spi_fer_int                                                                                                                                            |
+----------+----------+--------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 28       | TXUEN    | R/W    | 1'B1        | Interrupt enable of spi_txu_int                                                                                                                                            |
+----------+----------+--------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 27       | STOEN    | R/W    | 1'B1        | Interrupt enable of spi_sto_int                                                                                                                                            |
+----------+----------+--------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 26       | RXFEN    | R/W    | 1'B1        | Interrupt enable of spi_rxv_int                                                                                                                                            |
+----------+----------+--------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 25       | TXFEN    | R/W    | 1'B1        | Interrupt enable of spi_txe_int                                                                                                                                            |
+----------+----------+--------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 24       | ENDEN    | R/W    | 1'B1        | Interrupt enable of spi_end_int                                                                                                                                            |
+----------+----------+--------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 23:21    | RSVD     |        |             |                                                                                                                                                                            |
+----------+----------+--------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 20       | TXUCLR   | W1C    | 1'B0        | Interrupt clear of spi_txu_int                                                                                                                                             |
+----------+----------+--------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 19       | STOCLR   | W1C    | 1'B0        | Interrupt clear of spi_sto_int                                                                                                                                             |
+----------+----------+--------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 18:17    | RSVD     |        |             |                                                                                                                                                                            |
+----------+----------+--------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 16       | ENDCLR   | W1C    | 1'B0        | Interrupt clear of spi_end_int                                                                                                                                             |
+----------+----------+--------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 15:14    | RSVD     |        |             |                                                                                                                                                                            |
+----------+----------+--------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 13       | FERMASK  | R/W    | 1'B1        | Interrupt mask of spi_fer_int                                                                                                                                              |
+----------+----------+--------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 12       | TXUMASK  | R/W    | 1'B1        | Interrupt mask of spi_txu_int                                                                                                                                              |
+----------+----------+--------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 11       | STOMASK  | R/W    | 1'B1        | Interrupt mask of spi_sto_int                                                                                                                                              |
+----------+----------+--------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 10       | RXFMASK  | R/W    | 1'B1        | Interrupt mask of spi_rxv_int                                                                                                                                              |
+----------+----------+--------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 9        | TXFMASK  | R/W    | 1'B1        | Interrupt mask of spi_txe_int                                                                                                                                              |
+----------+----------+--------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 8        | ENDMASK  | R/W    | 1'B1        | Interrupt mask of spi_end_int                                                                                                                                              |
+----------+----------+--------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 7:6      | RSVD     |        |             |                                                                                                                                                                            |
+----------+----------+--------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 5        | FERINT   | R      | 1'B0        | SPI TX/RX FIFO error interrupt, auto-cleared when FIFO overflow/underflow error flag is cleared                                                                            |
+----------+----------+--------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 4        | TXUINT   | R      | 1'B0        | SPI slave mode TX underrun error flag, triggered when TXD is not ready during transfer in slave mode                                                                       |
+----------+----------+--------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 3        | STOINT   | R      | 1'B0        | SPI slave mode transfer time-out interrupt, triggered when SPI bus is idle for a given value                                                                               |
+----------+----------+--------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 2        | RXFINT   | R      | 1'B0        | SPI RX FIFO ready (rx_fifo_cnt > rx_fifo_th) interrupt, auto-cleared when data is popped                                                                                   |
+----------+----------+--------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 1        | TXFINT   | R      | 1'B0        | SPI TX FIFO ready (tx_fifo_cnt > tx_fifo_th) interrupt, auto-cleared when data is pushed                                                                                   |
+----------+----------+--------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 0        | ENDINT   | R      | 1'B0        | SPI transfer end interrupt, shared by both master and slave mode                                                                                                           |
+          +          +        +             +                                                                                                                                                                            +
|          |          |        |             | Master mode: Triggered when the final frame is transferred                                                                                                                 |
+          +          +        +             +                                                                                                                                                                            +
|          |          |        |             | Slave mode: Triggered when CS_n is de-asserted                                                                                                                             |
+----------+----------+--------+-------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

spi_bus_busy
--------------
 
**Address：**  0x4000a208
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                                                                                                                          |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                                                                                                              | BUSBUSY   |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+---------------------------+
| Bit      | Name     |Type    | Reset       | Description               |
+----------+----------+--------+-------------+---------------------------+
| 31:1     | RSVD     |        |             |                           |
+----------+----------+--------+-------------+---------------------------+
| 0        | BUSBUSY  | R      | 1'B0        | Indicator of SPI bus busy |
+----------+----------+--------+-------------+---------------------------+

spi_prd_0
-----------
 
**Address：**  0x4000a210
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| PRDPH1                                                                                        | PRDPH0                                                                                        |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| PRDP                                                                                          | PRDS                                                                                          |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+----------------------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                                              |
+----------+----------+--------+-------------+----------------------------------------------------------+
| 31:24    | PRDPH1   | R/W    | 8'D15       | Length of DATA phase 1 (please refer to "Timing" tab)    |
+----------+----------+--------+-------------+----------------------------------------------------------+
| 23:16    | PRDPH0   | R/W    | 8'D15       | Length of DATA phase 0 (please refer to "Timing" tab)    |
+----------+----------+--------+-------------+----------------------------------------------------------+
| 15:8     | PRDP     | R/W    | 8'D15       | Length of STOP condition (please refer to "Timing" tab)  |
+----------+----------+--------+-------------+----------------------------------------------------------+
| 7:0      | PRDS     | R/W    | 8'D15       | Length of START condition (please refer to "Timing" tab) |
+----------+----------+--------+-------------+----------------------------------------------------------+

spi_prd_1
-----------
 
**Address：**  0x4000a214
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                                                                                                                          |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                          | PRDI                                                                                          |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+-----------------------------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                                                     |
+----------+----------+--------+-------------+-----------------------------------------------------------------+
| 31:8     | RSVD     |        |             |                                                                 |
+----------+----------+--------+-------------+-----------------------------------------------------------------+
| 7:0      | PRDI     | R/W    | 8'D15       | Length of INTERVAL between frame (please refer to "Timing" tab) |
+----------+----------+--------+-------------+-----------------------------------------------------------------+

spi_rxd_ignr
--------------
 
**Address：**  0x4000a218
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                                                              | RXDIGS                                                    |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                                                              | RXDIGP                                                    |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+-------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                               |
+----------+----------+--------+-------------+-------------------------------------------+
| 31:21    | RSVD     |        |             |                                           |
+----------+----------+--------+-------------+-------------------------------------------+
| 20:16    | RXDIGS   | R/W    | 5'D0        | Starting point of RX data ignore function |
+----------+----------+--------+-------------+-------------------------------------------+
| 15:5     | RSVD     |        |             |                                           |
+----------+----------+--------+-------------+-------------------------------------------+
| 4:0      | RXDIGP   | R/W    | 5'D0        | Stopping point of RX data ignore function |
+----------+----------+--------+-------------+-------------------------------------------+

spi_sto_value
---------------
 
**Address：**  0x4000a21c
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                                                                                                                          |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                          | STOV                                                                                                                                          |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+-------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                               |
+----------+----------+--------+-------------+-------------------------------------------+
| 31:12    | RSVD     |        |             |                                           |
+----------+----------+--------+-------------+-------------------------------------------+
| 11:0     | STOV     | R/W    | 12'HFFF     | Time-out value for spi_sto_int triggering |
+----------+----------+--------+-------------+-------------------------------------------+

spi_fifo_config_0
-------------------
 
**Address：**  0x4000a280
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                                                                                                                          |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                          | RFUF      | RFOF      | TFUF      | TFOF      | RFC       | TFC       | DMAREN    | DMATEN    |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+----------------------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                                              |
+----------+----------+--------+-------------+----------------------------------------------------------+
| 31:8     | RSVD     |        |             |                                                          |
+----------+----------+--------+-------------+----------------------------------------------------------+
| 7        | RFUF     | R      | 1'B0        | Underflow flag of RX FIFO, can be cleared by rx_fifo_clr |
+----------+----------+--------+-------------+----------------------------------------------------------+
| 6        | RFOF     | R      | 1'B0        | Overflow flag of RX FIFO, can be cleared by rx_fifo_clr  |
+----------+----------+--------+-------------+----------------------------------------------------------+
| 5        | TFUF     | R      | 1'B0        | Underflow flag of TX FIFO, can be cleared by tx_fifo_clr |
+----------+----------+--------+-------------+----------------------------------------------------------+
| 4        | TFOF     | R      | 1'B0        | Overflow flag of TX FIFO, can be cleared by tx_fifo_clr  |
+----------+----------+--------+-------------+----------------------------------------------------------+
| 3        | RFC      | W1C    | 1'B0        | Clear signal of RX FIFO                                  |
+----------+----------+--------+-------------+----------------------------------------------------------+
| 2        | TFC      | W1C    | 1'B0        | Clear signal of TX FIFO                                  |
+----------+----------+--------+-------------+----------------------------------------------------------+
| 1        | DMAREN   | R/W    | 1'B0        | Enable signal of dma_rx_req/ack interface                |
+----------+----------+--------+-------------+----------------------------------------------------------+
| 0        | DMATEN   | R/W    | 1'B0        | Enable signal of dma_tx_req/ack interface                |
+----------+----------+--------+-------------+----------------------------------------------------------+

spi_fifo_config_1
-------------------
 
**Address：**  0x4000a284
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                  | RFTH                  | RSVD                                                                  | TFTH                  |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                      | RFCNT                             | RSVD                                                      | TFCNT                             |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                                                                               |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------+
| 31:26    | RSVD     |        |             |                                                                                           |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------+
| 25:24    | RFTH     | R/W    | 2'D0        | RX FIFO threshold, dma_rx_req will not be asserted if tx_fifo_cnt is less than this value |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------+
| 23:18    | RSVD     |        |             |                                                                                           |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------+
| 17:16    | TFTH     | R/W    | 2'D0        | TX FIFO threshold, dma_tx_req will not be asserted if tx_fifo_cnt is less than this value |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------+
| 15:11    | RSVD     |        |             |                                                                                           |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------+
| 10:8     | RFCNT    | R      | 3'D0        | RX FIFO available count                                                                   |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------+
| 7:3      | RSVD     |        |             |                                                                                           |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------+
| 2:0      | TFCNT    | R      | 3'D4        | TX FIFO available count                                                                   |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------+

spi_fifo_wdata
----------------
 
**Address：**  0x4000a288
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| FWDATA                                                                                                                                                                                        |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| FWDATA                                                                                                                                                                                        |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+---------------------+
| Bit      | Name     |Type    | Reset       | Description         |
+----------+----------+--------+-------------+---------------------+
| 31:0     | FWDATA   | W      | X           | SPI FIFO write data |
+----------+----------+--------+-------------+---------------------+

spi_fifo_rdata
----------------
 
**Address：**  0x4000a28c
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| FRDATA                                                                                                                                                                                        |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| FRDATA                                                                                                                                                                                        |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+--------------------+
| Bit      | Name     |Type    | Reset       | Description        |
+----------+----------+--------+-------------+--------------------+
| 31:0     | FRDATA   | R      | 32'H0       | SPI FIFO read data |
+----------+----------+--------+-------------+--------------------+

