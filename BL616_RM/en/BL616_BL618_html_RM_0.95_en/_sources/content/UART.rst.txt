===========
UART
===========

Overview
=====
The Universal Asynchronous Receiver/Transmitter (UART) provides a flexible way to exchange full-duplex data with external devices.
BL616/BL618 is provided with 2 UARTs, which can be used together with DMA to achieve efficient data communication.

Features
=========
- Full-duplex asynchronous communication

- Optional data bit length: 5/6/7/8-bit

- Optional stop bit length: 0.5/1/1.5/2-bit

- Supports odd/even/none check bit

- Error-detectable start bit

- Abundant interrupt control modes

- Hardware flow control (RTS/CTS)

- Convenient baud rate programming

- Configurable MSB/LSB transfer priority

- Automatic baud rate detection of ordinary/fixed characters

- 32-byte TX/RX FIFO

- Supports DMA transfer mode

- Supports baud rate of 10 Mbps and below

- Supports the LIN bus protocol

- Supports the RS485 mode

- Optional clock sources: 160M/BCLK/XCLk

- Support filter function

Functional Description
===========
Data Formats
-------------
The normal UART communication data consists of start bits, data bits, parity check bits, and stop bits. The UART of xx supports configurable data bits, parity check bits, and stop bits, which are set in registers utx_config and urx_config. The waveform of a frame of data is shown as follows:

.. figure:: ../../picture/UARTData.svg
   :align: center

   UART Data Format

The start bit of the data frame occupies 1 bit, and the stop bit can be 0.5/1/1.5/2-bit wide by configuring the cr_utx_bit_cnt_p bit in the register utx_config. The start bit is at a low level and the stop bit is at a high level.
The data bit width can be set to 5/6/7/8-bit by the cr_utx_bit_cnt_d bit in the register utx_config.

When the cr_utx_prt_en bit in the register utx_config and the cr_urx_prt_en bit in the register urx_config are set, the data frame adds a parity check bit after the data. The cr_utx_prt_sel bit in the register utx_config and the cr_urx_prt_sel bit in the register urx_config are used to select odd or even parity check. When the receiver detects the check bit error of the input data, it will generate the check error interrupt. However, the received data will still be stored into the FIFO.

Calculation method of odd parity check: If there is an odd number of "1" in the current data bit, the odd parity check bit is set to 0. Otherwise, it is set to 1.

Calculation method of even parity check: If there is an odd number of "1" in the current data bit, the even parity check bit is set to 1. Otherwise, it is set to 0.

Basic Architecture
-------------
.. figure:: ../../picture/UARTFramework.svg
   :align: center

   UART Architecture

The UART has 3 clock sources: XCLK, 160MHz CLK and BCLK. The frequency divider in the clock is used to divide the frequency of the clock source and then generate the clock signal to drive the UART module.

The UART controller is divided into two functional blocks: transmitter and receiver.

Transmitter
-------------
The transmitter contains a 32-byte TX FIFO to store the data to be sent. When the transmission enable bit is set, the data stored in FIFO will be output from the TX pin. Software can transfer data into TX FIFO through DMA or APB bus. Software can check the status of transmitter by querying the remaining free space count value of TX FIFO through tx_fifo_cnt in the register uart_fifo_config_1.

FreeRun mode of transmitter:

- If the FreeRun mode is disabled, transmission will be terminated and an interrupt will be generated when the sent bytes reach the specified length. Before next transmission, you need to re-disable and enable the TxE bit.

- If the FreeRun mode is enabled, the transmitter will send when there is data in the TX FIFO, and will not stop working because the sent bytes reach the specified length.

Receiver
-------------
The receiver contains a 32-byte RX FIFO to store the received data. Software can check the status of receiver by querying the available data count value of RX FIFO through rx_fifo_cnt in the register uart_fifo_config_1. 
The low 8 bits of the register URX_RTO_TIMER are used to set a receiving timeout threshold, which will trigger an interrupt when the receiver fails to receive data beyond the threshold. 
The cr_urx_deg_en and cr_urx_deg_cnt in the register urx_config are used to enable the deburring function and set the threshold, which control the filtering part before sampling by UART.
UART will filter out the burrs whose width is lower than the threshold in the waveform and then send it to sampling.

Baud Rate Setting
-------------
.. math:: Baudrate = \frac{UART\_clk}{uart\_prd + 1}

The user can set the baud rate of RX and TX separately. Take TX as an example: the value of uart_prd is the value of the lower 16 bits cr_utx_bit_prd of the register UART_BIT_PRD. Since the maximum value of the 16-bit bit width coefficient is 65535, the minimum baud rate supported by UART is : UART_clk/65536.

Before sampling the data, UART will filter the data to remove the burrs in the waveform. Then, the data will be sampled at the intermediate value of the 16-bit width factor, so that the sampling time can be adjusted based on baud rates, to ensure that the intermediate value is always sampled, providing much higher flexibility and accuracy. The sampling process is shown as follows:

.. figure:: ../../picture/UARTSample.svg
   :align: center

   UART Sampling Waveform

Filtering
-------------
.. figure:: ../../picture/UARTDeg.svg
   :align: center

   UART Filter Waveform

When this function is enabled by configuring cr_urx_deg_en and the threshold is set by configuring cr_urx_deg_cnt in the register urx_config, UART will filter out the data that cannot meet the width threshold. 
As shown in the figure below, when the data width is 4, setting cr_urx_deg_cnt to 4 can meet this condition. "Input" is the initial data and "output" is the filtered data.

Filtering logic process:

- "Tgl" is the exclusive OR result of input and output.

- "Deg_cnt" counts from 0, and the counting condition is that "tgl" is at the high level and "reached" is at the low level.

- If the count value of deg_cnt reaches the value set by cr_urx_deg_cnt, reached is high level.

- When "reached" is at a high level, "input" is output to "output".

- Note: user-defined condition for deg_cnt: "tgl" is at a high level and "reached" is at a low level. In other cases, "deg_cnt" will be cleared to 0.



Automatic Baud Rate Detection
-----------------
The UART module supports automatic baud rate detection, which is divided into two modes, a generic mode and a fixed character (square wave) mode.
The cr_urx_abr_en in the urx_config register enables auto baud rate detection, and when it is turned on, both detection modes are enabled.

**Generic mode**

For any character data received, UART will count the number of clocks in the start bit width, which will then be written into the low 16 bits sts_urx_abr_prd_start in the register STS_URX_ABR_PRD and used to calculate the baud rate. So the correct baud rate can be obtained when the first received data bit is 1, such as '0x01' under LSBFIRST.

**Fixed character (square wave) mode**

In this mode, after the UART module counts the number of clocks in the bit width, it will continue to count the number of clocks in the subsequent data bits and compare it with the start bit. The check is passed, otherwise the count value is discarded. The allowable error can be set by setting the cr_urx_abr_pw_tol bit in the register urx_abr_pw_tol, and the unit is the clock source of the UART.

Therefore, only when the fixed character '0x55'/'0xD5' under LSB-FIRST or '0xAA'/'0xAB' under MSB-FIRST is received, the UART module will write the clock count value in the starting bit width into the high 16-bit sts_urx_abr_prd_0x55 of the register STS_URX_ABR_PRD. As shown below:

.. figure:: ../../picture/UARTAbr.svg
   :align: center

   Waveform of UART in fixed character mode

As shown above, assuming the maximum allowable error set is 4, for a received data with unknown baud rate, the UART uses UART_CLK to count the bit width of the starting bit as 1000, the bit width of the second bit as 1001, which is not more than 4 UART_CLK up or down from the previous bit width, then the UART will continue to count the third bit. The third bit is 1005, the difference with the starting bit is more than 4, the detection is not passed and the data is discarded. UART compares the first 6 bit widths of the data bits with the starting bit in turn.

Formula for calculating the detected baud rate:

.. math:: Baudrate = \frac{UART\_clk}{Count + 1}

Hardware Flow Control
-------------
UART supports hardware flow control in CTS/RTS mode to prevent data in FIFO from being lost due to too late processing. Hardware flow control connection is shown as follows:

.. figure:: ../../picture/UARTCTSRTS.svg
   :align: center

   UART hardware flow control

Require To Send (RTS) is an output signal, which indicates whether the chip is ready to receive data from the other side. This is valid at a low level denoting that the chip can receive data.

Clear To Send (CTS) is an input signal, which determines whether the chip can send data to the other side. This is valid at a low level denoting that the chip can send data to the other side.

When the hardware flow control function is enabled, the low level of chip's RTS indicates requesting the other side to send data, and the high level of that indicates informing the other side to stop sending data.
When the chip detects that CTS goes high, TX will stop sending data, and continue sending until CTS goes low. If CTS goes high or low at any time during communication, it does not affect the continuity of data sent by TX, and the other side also can receive continuous data.

Two ways for hardware flow control of the transmitter:

- Hardware control (the cr_urx_rts_sw_mode in the register uart_sw_mode is 0): RTS goes high when cr_urx_en in the register urx_config is not turned on or the RX FIFO is almost full (one byte left).

- Software control(the cr_urx_rts_sw_mode in the register uart_sw_mode is 1): The level of RTS can be changed by configuring cr_urx_rts_sw_val in the register uart_sw_mode.

DMA Transfer
-------------
UART supports DMA transfer. Using DMA transfer, the TX and RX FIFO thresholds need to be set respectively by tx_fifo_th and rx_fifo_th in register uart_fifo_config_1.
When this mode is enabled, if tx_fifo_cnt in uart_fifo_config_1 is greater than tx_fifo_th, a DMA TX request will be triggered.
After the DMA is configured, when the DMA receives the request, it will move the data from the memory to the TX FIFO according to the settings.
If the rx_fifo_cnt in uart_fifo_config_1 is greater than rx_fifo_th, the DMA RX request will be triggered.
After the DMA is configured, when the DMA receives the request, it will transfer the data of the RX FIFO to the memory according to the settings.

In order to ensure the correctness of the data transferred by the chip DMA TX Channel, the following conditions need to be met in the Channel configuration: (transferWidth * burstSize) ≤ (tx_fifo_th + 1).

In order to ensure the integrity of the data transferred by the chip DMA RX Channel, the following conditions need to be met in the Channel configuration: (transferWidth * burstSize) = (rx_fifo_th + 1).

Support for LIN Bus
-------------
The protocol for the Local Interconnect Network (LIN) is based on the Volcano-Lite technology developed by the Volvo spin-out company—Volcano Communications Technology (VCT).
LIN is a complementary protocol to CAN and SAE J1850, suitable for applications that have low requirement for time or require no precise fault tolerance (as LIN is not as reliable as CAN).
LIN aims to be easy to use as a low-cost alternative to CAN. The vehicle parts where LIN can be used include window regulator, rearview mirror, wiper, and rain sensor.

UART supports the LIN bus mode.The LIN bus is under the master-slave mode, and data is always initiated by the master node. The frame (header) sent by the master node contains synchronization interval field, synchronization byte field, and identifier field.
A typical LIN data transfer is shown as follows.

.. figure:: ../../picture/UARTLinFrame.svg
   :align: center

   A typical LIN frame

1. LIN Break Field

.. figure:: ../../picture/UARTLinBreak.svg
   :align: center

   Break Field of LIN

The synchronization interval field indicates the start of the message, with at least 13 dominant bits (including the start bit). The synchronization interval ends with an "interval separator", which contains at least one recessive bit.

The length of the Break in the LIN frame can be set by cr_utx_bit_cnt_b in utx_config.

2. LIN Sync Field

A synchronization byte field is sent to determine the time between two falling edges, to determine the transmission rate used by the master node. The bit pattern is 0x55 (01010101, maximum number of falling edges).

.. figure:: ../../picture/UARTLinSync.svg
   :align: center

   Sync Field of LIN

3. LIN ID Field

The identifier field contains a 6-bit identifier and two parity check bits. The 6-bit identifier contains information about the sender and receiver, and the number of bytes required in the response. The parity check bit is calculated as follows: The check bit P0 is the result of logical OR operation among ID0, ID1, ID2, and ID4. The check bit P1 is the result of inversion after logical OR operation among ID1, ID3, ID4, and ID5.

.. figure:: ../../picture/UARTLinId.svg
   :align: center

   ID Field of LIN

The slave node waits for the synchronization interval field, and then starts to synchronize between master and slave nodes through the synchronization byte field. Depending on the identifier sent by the master node, the slave node will receive, send or do not respond. The slave node that should send data sends the number of bytes requested by the master node, and then ends the transmission with a checksum field.

UART supports the LIN transfer mode. To enable this mode, you need to configure the cr_utx_bit_cnt_b by setting cr_utx_lin_en in the register utx_config so that the synchronization interval field consists of at least a 13-bit dominant level.

RS485 mode
-------------
UART supports the RS485 mode. After the cr_utx_rs485_en in the register UTX_RS485_CFG is set, UART can work in the RS485 mode.
Then, UART can be connected to the RS485 bus through an external RS485 transceiver.
In this mode, the RTS pin in the module performs the Dir function of transceiver.
When UART has data to send, it will automatically control the RTS pin at a high level, so that the transceiver can send data to the bus.
Contrarily, when UART has no data to send, it will automatically control the RTS at a low level, to keep the transceiver in the RX state.

UART supports the RS485 transfer mode. To enable this mode, you need to set cr_utx_rs485_pol and cr_utx_rs485_en in the register UTX_RS485_CFG.

UART Interrupt
-------------
UART supports the following interrupt control modes:

- TX end of transfer interrupt

- RX end interrupt

- TX FIFO request interrupt

- RX FIFO request interrupt

- RX timeout interrupt

- RX parity check error interrupt

- TX FIFO overflow interrupt

- RX FIFO overflow interrupt

- RX BCR interrupt

- LIN synchronization error interrupt

- Auto baud rate detection (universal mode) interrupt

- Auto baud rate detection (fixed characters mode) interrupt

TX/RX end of transfer interrupt
-----------------
You can set a transfer length for TX and RX respectively by configuring the high 16 bits of the registers utx_config and urx_config. When the number of transferred bytes reaches this value, the corresponding TX/RX end of transfer interrupt will be triggered.
While this interrupt is generated, TX stops working. To continue to use TX, you must re-initialize this module. Then, RX resumes to work.
If the preset transfer length of TX is less than the data volume actually sent by TX, the other side can only receive the data equal to the transfer length, and the remaining data will be stored in TX FIFO. After this module is re-initialized, the data in TX FIFO can be sent out.

For TX, if the data continuously filled into the TX FIFO is greater than the set transmission length value, only the data of the set length value will be transmitted on the TX pin, and the excess data will be kept in the TX FIFO, and the TX will be re-enabled. After the function, the remaining data in the TX FIFO will be sent out.

For example: set the TX transmission length value to 64, enable the TX function, first fill 63 bytes into the TX FIFO, these 63 bytes will be transmitted on the pins, but no TX transmission completion interrupt is generated, and then the TX FIFO is sent to the TX FIFO. Fill in 1 byte, at this time, the transmission length reaches the transmission length value set by TX, a transmission completion interrupt will be generated, and the TX function will stop.
Continue to fill 1 byte into the TX FIFO, you will find that there is no data transmission on the pin, the byte is still retained in the TX FIFO, and the TX function is turned off and re-enabled, and the byte is sent out on the pin.

For RX, if the data length sent by the other party exceeds the set transmission length, RX can continue to receive data after the RX transmission completion interrupt is generated.

For example: set the RX transmission length value to 16, the other party sends 32 bytes of data, RX will generate an RX transmission completion interrupt when it receives 16 bytes of data, and continue to receive the remaining 16 bytes of data, all saved in the RX FIFO.

TX/RX FIFO request interrupt
------------------
A TX FIFO request interrupt will be generated when tx_fifo_cnt in uart_fifo_config_1 is greater than tx_fifo_th. When the condition is not met, the interrupt flag will be cleared automatically.

A RX FIFO request interrupt will be generated when rx_fifo_cnt in uart_fifo_config_1 is greater than rx_fifo_th. When the condition is not met, the interrupt flag will be cleared automatically.

TX/RX supports multiple rounds of transmission/receiving, instead of reaching the value set by tx_fifo_th/rx_fifo_th at a time.

E.g:

1. Set tx_fifo_th/rx_fifo_th in register uart_fifo_config_1 to 16.
2. Set cr_utx_frm_en in register utx_config to enable free run mode.
3. Set cr_utx_frdy_mask/cr_urx_frdy_mask in register uart_int_mask to 0, and enable FIFO interrupt of TX/RX.
4. Set cr_utx_en/cr_urx_en in register utx_config/urx_config to enable TX/RX.
5. TX FIFO interrupt: TX will always enter the FIFO interrupt, when the chip sends 128 bytes, set cr_utx_frdy_mask to 1 to shield the interrupt. If you want to enter the TX FIFO interrupt again, set cr_utx_frdy_mask to 0.
6. RX FIFO interrupt: the other party first sends 15 bytes, no interrupt is generated, at this time, the value of rx_fifo_cnt is 15, and an interrupt is generated when 1 byte is sent again to reach the value set by rx_fifo_th. After the transmission is interrupted, the other party sends the data again, and the chip can receive the data.

RX timeout interrupt
---------------------
The RX timeout interrupt generation condition is: after receiving data last time, the receiver will start timing, and the interrupt will be triggered when the timing value exceeds the timeout threshold and the next data has not been received. The time-out threshold value is in the unit of communication bit.

When the other party sends data to the chip, a timeout interrupt will be generated after the set timeout period is reached.

RX parity check error interrupt
-----------------
The RX parity check error interrupt will be generated when a parity check error occurs. But it does not affect the RX, which still can correctly receive and analyze the data sent by the other side. When receiving data, RX takes the first 8 bits as data bits and ignores parity check bits, ensuring data consistency.

For example, you can enable parity check by setting cr_utx_prt_en/cr_urx_prt_en in the register utx_config/urx_config, and select the parity check type by setting cr_utx_prt_sel/cr_urx_prt_sel in the register utx_config/urx_config. When the other side sends data to the chip through odd/even parity check, the parity check of RX is disabled, but RX can receive correct data.

TX/RX FIFO overflow interrupt
------------------
If the TX/RX FIFO overflows or underflows, it will trigger the corresponding overflow interrupt. When the tx_fifo_clr and rx_fifo_clr bits in the FIFO clear bit register UART_FIFO_CONFIG_0 are set to 1, the corresponding FIFO will be cleared and the overflow interrupt flag will be cleared automatically.
You can query the interrupt status through the register UART_INT_STS, and clear the interrupt by writing 1 to the corresponding bit in the register UART_INT_CLEAR.

RX BCR interrupt
----------
A BCR interrupt will be generated when the data received by RX reaches the value set by cr_urx_bcr_value in the register urx_bcr_int_cfg.

The difference from RX END interrupt is that END interrupt is suitable for receiving data of known length, while BCR interrupt can be used to receive interrupts of unknown length. The trigger position of the END interrupt is controlled by cr_urx_len, and the counter will be cleared to 0 when an interrupt is triggered. The trigger position of BCR interrupt is controlled by cr_urx_bcr_value. When the interrupt is triggered, the counter will accumulate instead of being cleared to 0, but it can be cleared by software (cr_urx_bcr_clr). When the BCR interrupt is used together with the chained DMA, if you do not know how much data has been transferred by DMA, you can check it through "count".

LIN synchronization error interrupt
----------------
When cr_utx_lin_en in utx_config is enabled, the LIN mode is enabled. Then, if the synchronization field of the LIN bus is not detected when data is received in this mode, the LIN synchronization error interrupt will be generated.

Auto baud rate detection (universal/fixed characters mode) interrupt
---------------------------------
In the auto baud rate detection mode, when a baud rate is detected, the auto baud rate detection (universal/fixed characters mode) interrupt will be generated as configured.

.. only:: html

   .. include:: uart_register.rst

.. raw:: latex

   \input{../../en/content/uart}