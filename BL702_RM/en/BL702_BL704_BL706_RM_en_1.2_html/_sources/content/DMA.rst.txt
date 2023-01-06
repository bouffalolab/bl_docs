==========
DMA
==========

DMA Introduction
=================
DMA (Direct Memory Access) is a memory access technology that can independently read and write system memory directly without processor intervention.
Under the same degree of processor load, DMA is a fast data transfer method.
The DMA controller has 8 channels,  which manage the data transfer between peripheral devices and memory to improve bus efficiency.

There are four main types of transfers: memory to memory, memory to peripheral, peripheral to peripheral and peripheral to memory. And support LLI link list function.
Use the software to configure the transmission data size, data source address, and destination address.

DMA main features
==================
- 8 independently configurable channels (requests) on DMA
- Independent control of source and destination access width (single-byte, double-byte, four-byte)
- Each channel acts as a read-write cache independently
- Each channel can be triggered by independent peripheral hardware or software
- Support peripherals including UART、I2C、SPI、ADC、I2S、DAC。
- 4 kinds of process control

  * DMA flow control, source memory, target memory 
  * DMA flow control, source memory, target peripheral
  * DMA flow control, source peripheral, target memory
  * DMA flow control, source peripheral, target peripheral

- Support LLI linked list function to improve DMA efficiency

DMA functional description
=========================
Working principle
-----------------------
When a device attempts to transfer data (usually a large amount of data) directly to another device via the 
bus, it will first send a DMA request signal to the CPU. The peripheral device makes a bus request to the 
CPU to take over the bus control right through the DMA. After the CPU receives the signal, after the current 
bus cycle ends, it will respond to the DMA signal according to the priority of the DMA signal and the order 
of the DMA request.

When the CPU responds to a DMA request to a device interface, it will give up bus control.

Therefore, under the management of the DMA controller, the peripherals and the memory directly exchange 
data without CPU intervention. After the data transfer is complete, the device sends a DMA end signal to 
the CPU, returning the bus control.

.. figure:: ../../picture/DmaArch.svg
   :align: center
   :scale: 70%

   DMA architecture

The DMA includes a set of AHB Master interfaces and a set of AHB Slave interfaces. 
The AHB Master interface actively accesses memory or peripherals through the system 
bus according to the current configuration requirements, as a port for data movement. 
The AHB Slave interface is used to configure the DMA interface and only supports 32-bit access.

DMA channel configuration
----------------------------
DMA supports 8 channels in total, each channel does not interfere with each other and can run 
at the same time. The following is the configuration process of DMA channel x:

1. Set 32-bit source address in DMA_C0SrcAddr register

2. Set the 32-bit target address in the DMA_C0DstAddr register

3. Configure SI (source) and DI (destination) in the DMA_C0Control register to set whether to enable the automatic address accumulation mode. When set to 1, enable the automatic address accumulation mode

4. Set the transmission data width by configuring the STW (source) and DTW (destination) bits in the DMA_C0Control register. The width options are single-byte, double-self-knot, and four-byte

5. Burst type, which can be set by configuring the SBS (source) and DBS (destination) bits in the DMA_C0Control register. The configuration options are Single, INCR4, INCR8, INCR16

6. Special attention should be paid to the configured combination. A single burst cannot exceed 16 bytes

7. Set the data transmission length range: 0-4095

Peripheral support
-------------------------
The SrcPeripheral (source) and DstPeripheral (destination) are configured to determine the 
peripherals that the current DMA cooperates with. The relationship is 0-5 : UART / 6-9 : I2C / 10-13 : SPI / 18-21 : I2S / 22 : ADC / 23 : DAC

**UART uses DMA to transfer data**

UART sends data packets, using DMA method can greatly reduce CPU processing time, so that its CPU 
resources are not wasted a lot.
Especially when the UART sends and receives a large number of data packets (such as high-frequency 
sending and receiving instructions) has obvious advantages.

Taking UART0 transmission as an example, the configuration process is as follows:

1. Set the value of the register DMA_C0Config [SRCPH] bit to 1, that is, set the Source peripheral to UART_TX

2. Set the value of the DMA_C0Config [DSTPH] bit to 0, that is, set the Destination peripheral to UART_RX

**I2C uses DMA to transfer data**

The configuration is as follows:

1. Set the value of the register DMA_C0Config [SRCPH] bit to 7, that is, set the Source peripheral to I2C_TX

2. Set the value of the DMA_C0Config [DSTPH] bit to 6, that is, set the Destination peripheral to I2C_RX

**SPI uses DMA to transfer data**

The configuration is as follows:

1. Set the value of the DMA_C0Config [SRCPH] bit to 11, that is, set the Source peripheral to SPI_TX

2. Set the value of the DMA_C0Config [DSTPH] bit to 10, that is, set the Destination peripheral to SPI_RX

**ADC uses DMA to transfer data**

The configuration is as follows:

1. Set the value of the DMA_C0Config [SRCPH] bit to 22, that is, set the Source peripheral to GPADC

**DAC uses DMA to transfer data**

The configuration is as follows:

1. Set the value of the DMA_C0Config [SRCPH] bit to 23, that is, set the Source peripheral to GPDAC

Linked List Mode
-----------------------
DMA supports linked list operation mode. When performing a DMA read or write operation, 
you can fill the data in the next linked list. After completing the data transfer of the 
current linked list, read the DMA_C0LLI register to obtain the start address of the next 
linked list, and directly transfer the data in the next linked list. 

Ensure continuous and uninterrupted work during DMA transfer, and improve the efficiency of CPU and DMA.

.. figure:: ../../picture/DMALLI.svg
   :align: center
   :scale: 70%

   LLI architecture

DMA interrupt
---------------

- DMA_INT_TCOMPLETED

   * Data transmission completed interrupt. When a data transmission is completed, this interrupt will be entered.

- DMA_INT_ERR

   * Data transmission error interrupt, when an error occurs during data transmission, this interrupt will be entered

Transmission mode
=====================
Memory to memory
-------------------
After this mode is started, the DMA will move the data from the source address to the 
destination address according to the set transfer size. After the transfer, the DMA controller 
will automatically return to the idle state and wait for the next transfer.

The specific configuration process is as follows:

1. Set the value of the register DMA_C0SrcAddr to the memory address of the source

2. Set the value of the register DMA_C0DstAddr to the target memory address

3. Select the transmission mode and set the value of the DMA_C0Config [FLOWCTRL] bit to 0, that is, select the memory-to-memory mode

4. Set the value of the corresponding bit in the DMA_C0Control register: set the DI and SI bits to 1 to enable the automatic address accumulation mode, the DTW and STW bits set the transmission width of the source and destination, and the DBS and SBS bits set the burst type of the source and destination

5. Select the appropriate channel, enable DMA, and complete the data transfer

Memory to peripheral
-------------------------
In this working mode, the DMA will move data from the source to the internal cache 
according to the set transfer size (TransferSize). When the cache space is insufficient, 
the DMA will automatically suspend it. When there is sufficient cache space, continue to 
transfer until it reaches Set the moving quantity.

On the other hand, when the target peripheral request triggers, it will burst the target 
configuration to the target address until it reaches the set number of moves and 
automatically returns to the idle state, waiting for the next startup.

The specific configuration process is as follows:

1. Set the value of the register DMA_C0SrcAddr to the memory address of the source

2. Set the value of the register DMA_C0DstAddr to the target peripheral address

3. Select the transfer mode and set the value of the DMA_C0Config [FLOWCTRL] bit to 1 to select the memory-to-peripheral mode

4. Set the value of the corresponding bit in the DMA_C0Control register: the SI bit is set to 1 to enable the address auto-accumulation mode, and the DI bit is set to 0 to disable the address auto-accumulation mode, the DTW and STW bits set the transmission width of the source and destination, and the DBS and SBS bits set the burst type of the source and destination

5. Select the appropriate channel, enable DMA, and complete the data transfer

Peripheral to memory
-----------------------
In this working mode, when the source peripheral request is triggered, 
the source configuration is burst to the buffer until the set number of 
moves reaches the stop. On the other hand, when the internal cache is enough 
for the target burst number once, the DMA will automatically move the cached 
content to the target address until it reaches the set number of moves and 
automatically returns to the idle state, waiting for the next startup

The specific configuration process is as follows:

1. Set the value of the register DMA_C0SrcAddr to the source peripheral address

2. Set the value of the register DMA_C0DstAddr to the target memory address

3. Select the transfer mode and set the value of the DMA_C0Config [FLOWCTRL] bit to 2 to select the Peripheral-to-memory mode

4. Set the value of the corresponding bit in the DMA_C0Control register: the DI bit is set to 1 to enable the address auto-accumulation mode, and the SI bit is set to 0 to disable the address auto-accumulation mode , the DTW and STW bits set the transmission width of the source and destination respectively, and the DBS and SBS bits set the burst type of the source and destination respectively

5. Select the appropriate channel, enable DMA, and complete the data transfer

Peripheral to peripheral
--------------------------
In this working mode, when the source peripheral requests a trigger, the source configuration burst will be stored in the buffer, and it will stop until the set number of moves is reached. On the other hand, when the internal cache is enough for the target burst number, DMA will automatically move the cached content to the target address until the set number of transfers is reached and automatically return to the idle state, waiting for the next start

The specific configuration process is as follows:

1. Set the value of the register DMA_C0SrcAddr to the peripheral address of the source

2. Set the value of the register DMA_C0DstAddr to the target peripheral address

3. Select the transfer mode, set the value of the register DMA_C0Config[FLOWCTRL] bit to 3, that is, select the Peripheral-to-Peripheral mode

4. Set the value of the corresponding bit in the DMA_C0Control register: DI and SI bits are set to 0, the address automatic accumulation mode is disabled, the STW and DTW bits respectively set the source and target transfer widths, and the SBS and DBS bits respectively set the source and target bursts type.

5. Select the appropriate channel, enable DMA, and complete the data transfer 

.. only:: html

   .. include:: dma_register.rst

.. raw:: latex

   \input{../../en/content/dma}