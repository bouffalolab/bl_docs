==========
DMA
==========

Introduction
=================
DMA (Direct Memory Access) is a memory access technology that can independently read and write system memory directly without processor intervention.
Under the same degree of processor load, DMA is a fast data transfer method.
The DMA controller has 4 channels,  which manage the data transfer between peripheral devices and memory to improve bus efficiency.

There are three main types of transfers: memory to memory, memory to peripheral, and peripheral to memory. And support LLI link list function.
Use the software to configure the transmission data size, data source address, and destination address.

DMA main features
======================
- 4 independently configurable channels (requests) on DMA
- Independent control of source and destination access width (single-byte, double-byte, four-byte)
- Each channel acts as a read-write cache independently
- Each channel can be triggered by independent peripheral hardware or software
- Support peripherals including UART, I2C, SPI, ADC
- 8 kinds of process control

  * DMA flow control, source memory, target memory 
  * DMA flow control, source memory, target peripheral
  * DMA flow control, source peripheral, target memory
  * DMA flow control, source peripheral, target peripheral
  * Target peripheral process control, source peripheral, target peripheral
  * Target peripheral process control, source memory, target peripheral
  * Source peripheral process control, source peripheral, target memory
  * Source peripheral process control, source peripheral, target peripheral

-	Support LLI linked list function to improve DMA efficiency

DMA functional description
=================================
DMA transactions
----------------------
When a device attempts to transfer data directly to another device via the 
bus, it will first send a DMA request signal to the CPU. The peripheral device makes a bus request to the 
CPU to take over the bus control right through the DMA. After the CPU receives the signal, after the current 
bus cycle ends, it will respond to the DMA signal according to the priority of the DMA signal and the order 
of the DMA request.

When the CPU responds to a DMA request to a device interface, it will give up bus control.

Therefore, under the management of the DMA controller, the peripherals and the memory directly exchange 
data without CPU intervention. After the data transfer is complete, the device sends a DMA end signal to 
the CPU, returning the bus control.

.. figure:: /content/Dma/picture/Dma_Arch.png
   :align: center

   DMA architecture

The DMA includes a set of AHB Master interfaces and a set of AHB Slave interfaces. 
The AHB Master interface actively accesses memory or peripherals through the system 
bus according to the current configuration requirements, as a port for data movement. 
The AHB Slave interface is used to configure the DMA interface and only supports 32-bit access.

DMA channel configuration
---------------------------------
DMA supports 4 channels in total, each channel does not interfere with each other and can run 
at the same time. The following is the configuration process of DMA channel x:

1. Set 32-bit source address in DMA_C0SrcAddr register

2. Set the 32-bit target address in the DMA_C0DstAddr register

3. Configure SI (source) and DI (destination) in the DMA_C0Control register to set whether to enable the automatic address accumulation mode. When set to 1, enable the automatic address accumulation mode

4. Set the transmission data width by configuring the STW (source) and DTW (destination) bits in the DMA_C0Control register. The width options are single-byte, double-self-knot, and four-byte.

5. Burst type, which can be set by configuring the SBS (source) and DBS (destination) bits in the DMA_C0Control register. The configuration options are Single, INCR4, INCR8, INCR16

6. Special attention should be paid to the configured combination. A single burst cannot exceed 16 bytes.

7. Set the data transmission length range: 0-4095

Peripheral support
----------------------
The SrcPeripheral (source) and DstPeripheral (destination) are configured to determine the 
peripherals that the current DMA cooperates with. The relationship is 
0-3: UART / 6-7: I2C / 10-11: SPI / 22-23: ADC / DAC

**UART uses DMA to transfer data**

UART sends data packets, using DMA method can greatly reduce CPU processing time, so that its CPU 
resources are not wasted a lot,
Especially when the UART sends and receives a large number of data packets (such as high-frequency 
sending and receiving instructions) has obvious advantages.

Taking UART0 as an example, the configuration process is as follows:

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

**ADC0/1 uses DMA to transfer data**

The configuration is as follows:

1. Set the value of the DMA_C0Config [SRCPH] bit to 22/23, that is, set the Source peripheral to GPADC0 / GPADC1


Linked List Mode
-----------------------
DMA supports linked list operation mode. When performing a DMA read or write operation, 
you can fill the data in the next linked list. After completing the data transfer of the 
current linked list, read the DMA_C0LLI register to obtain the start address of the next 
linked list, and directly transfer the data in the next linked list. 

Ensure continuous and uninterrupted work during DMA transfer, and improve the efficiency of CPU and DMA.

.. figure:: /content/Dma/picture/DMA_LLI.png
   :align: center

   LLI architecture

DMA interrupt
------------------

- DMA_INT_TCOMPLETED

   * Data transmission completed interrupt. When a data transmission is completed, this interrupt will be entered.
 
- DMA_INT_ERR
 
   * Data transmission error interrupt, when an error occurs during data transmission, this interrupt will be entered


Transmission mode
=======================
Memory to memory
------------------
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

4. Set the value of the corresponding bit in the DMA_C0Control register: set the DI and SI bits to 1 to enable the automatic address accumulation mode, the DTW and STW bits set the transmission width of the source and destination, and the DBS and SBS bits set the burst type of the source and destination

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

4. Set the value of the corresponding bit in the DMA_C0Control register: set the DI and SI bits to 1 to enable the automatic address accumulation mode, the DTW and STW bits set the transmission width of the source and destination respectively, and the DBS and SBS bits set the burst type of the source and destination respectively

5. Select the appropriate channel, enable DMA, and complete the data transfer


Register description
==========================

+--------------------------+---------------------------------------------------------+
| Name                     | Description                                             |
+--------------------------+---------------------------------------------------------+
| `DMA_IntStatus`_         | Interrupt status                                        |
+--------------------------+---------------------------------------------------------+
| `DMA_IntTCStatus`_       | Interrupt terminal count request status                 |
+--------------------------+---------------------------------------------------------+
| `DMA_IntTCClear`_        | Terminal count request clear                            |
+--------------------------+---------------------------------------------------------+
| `DMA_IntErrorStatus`_    | Interrupt error status                                  |
+--------------------------+---------------------------------------------------------+
| `DMA_IntErrClr`_         | Interrupt error clear                                   |
+--------------------------+---------------------------------------------------------+
| `DMA_RawIntTCStatus`_    | Status of the terminal count interrupt prior to masking |
+--------------------------+---------------------------------------------------------+
| `DMA_RawIntErrorStatus`_ | Status of the error interrupt prior to masking          |
+--------------------------+---------------------------------------------------------+
| `DMA_EnbldChns`_         | Channel enable status                                   |
+--------------------------+---------------------------------------------------------+
| `DMA_SoftBReq`_          | Software burst request                                  |
+--------------------------+---------------------------------------------------------+
| `DMA_SoftSReq`_          | Software single request                                 |
+--------------------------+---------------------------------------------------------+
| `DMA_SoftLBReq`_         | Software last burst request                             |
+--------------------------+---------------------------------------------------------+
| `DMA_SoftLSReq`_         | Software last single request                            |
+--------------------------+---------------------------------------------------------+
| `DMA_Config`_            | DMA general configuration                               |
+--------------------------+---------------------------------------------------------+
| `DMA_Sync`_              | DMA request asynchronous setting                        |
+--------------------------+---------------------------------------------------------+
| `DMA_C0SrcAddr`_         | Channel DMA source address                              |
+--------------------------+---------------------------------------------------------+
| `DMA_C0DstAddr`_         | Channel DMA Destination address                         |
+--------------------------+---------------------------------------------------------+
| `DMA_C0LLI`_             | Channel DMA link list                                   |
+--------------------------+---------------------------------------------------------+
| `DMA_C0Control`_         | Channel DMA bus control                                 |
+--------------------------+---------------------------------------------------------+
| `DMA_C0Config`_          | Channel DMA configuration                               |
+--------------------------+---------------------------------------------------------+
| `DMA_C1SrcAddr`_         | Channel DMA source address                              |
+--------------------------+---------------------------------------------------------+
| `DMA_C1DstAddr`_         | Channel DMA Destination address                         |
+--------------------------+---------------------------------------------------------+
| `DMA_C1LLI`_             | Channel DMA link list                                   |
+--------------------------+---------------------------------------------------------+
| `DMA_C1Control`_         | Channel DMA bus control                                 |
+--------------------------+---------------------------------------------------------+
| `DMA_C1Config`_          | Channel DMA configuration                               |
+--------------------------+---------------------------------------------------------+
| `DMA_C2SrcAddr`_         | Channel DMA source address                              |
+--------------------------+---------------------------------------------------------+
| `DMA_C2DstAddr`_         | Channel DMA Destination address                         |
+--------------------------+---------------------------------------------------------+
| `DMA_C2LLI`_             | Channel DMA link list                                   |
+--------------------------+---------------------------------------------------------+
| `DMA_C2Control`_         | Channel DMA bus control                                 |
+--------------------------+---------------------------------------------------------+
| `DMA_C2Config`_          | Channel DMA configuration                               |
+--------------------------+---------------------------------------------------------+
| `DMA_C3SrcAddr`_         | Channel DMA source address                              |
+--------------------------+---------------------------------------------------------+
| `DMA_C3DstAddr`_         | Channel DMA Destination address                         |
+--------------------------+---------------------------------------------------------+
| `DMA_C3LLI`_             | Channel DMA link list                                   |
+--------------------------+---------------------------------------------------------+
| `DMA_C3Control`_         | Channel DMA bus control                                 |
+--------------------------+---------------------------------------------------------+
| `DMA_C3Config`_          | Channel DMA configuration                               |
+--------------------------+---------------------------------------------------------+

DMA_IntStatus
---------------
 
**Address：**  0x4000c000
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                                                                                                                          |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                          | INTSTA                                                                                        |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+--------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                                |
+----------+----------+--------+-------------+--------------------------------------------+
| 31:8     | RSVD     |        |             |                                            |
+----------+----------+--------+-------------+--------------------------------------------+
| 7:0      | INTSTA   | R      | 0           | Status of the DMA interrupts after masking |
+----------+----------+--------+-------------+--------------------------------------------+

DMA_IntTCStatus
-----------------
 
**Address：**  0x4000c004
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                                                                                                                          |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                          | INTTCSTA                                                                                      |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+-----------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                             |
+----------+----------+--------+-------------+-----------------------------------------+
| 31:8     | RSVD     |        |             |                                         |
+----------+----------+--------+-------------+-----------------------------------------+
| 7:0      | INTTCSTA | R      | 0           | Interrupt terminal count request status |
+----------+----------+--------+-------------+-----------------------------------------+

DMA_IntTCClear
----------------
 
**Address：**  0x4000c008
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                                                                                                                          |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                          | TCRC                                                                                          |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+------------------------------+
| Bit      | Name     |Type    | Reset       | Description                  |
+----------+----------+--------+-------------+------------------------------+
| 31:8     | RSVD     |        |             |                              |
+----------+----------+--------+-------------+------------------------------+
| 7:0      | TCRC     | W      | 0           | Terminal count request clear |
+----------+----------+--------+-------------+------------------------------+

DMA_IntErrorStatus
--------------------
 
**Address：**  0x4000c00c
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                                                                                                                          |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                          | IES                                                                                           |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+------------------------+
| Bit      | Name     |Type    | Reset       | Description            |
+----------+----------+--------+-------------+------------------------+
| 31:8     | RSVD     |        |             |                        |
+----------+----------+--------+-------------+------------------------+
| 7:0      | IES      | R      | 0           | Interrupt error status |
+----------+----------+--------+-------------+------------------------+

DMA_IntErrClr
---------------
 
**Address：**  0x4000c010
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                                                                                                                          |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                          | IEC                                                                                           |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+-----------------------+
| Bit      | Name     |Type    | Reset       | Description           |
+----------+----------+--------+-------------+-----------------------+
| 31:8     | RSVD     |        |             |                       |
+----------+----------+--------+-------------+-----------------------+
| 7:0      | IEC      | W      | 0           | Interrupt error clear |
+----------+----------+--------+-------------+-----------------------+

DMA_RawIntTCStatus
--------------------
 
**Address：**  0x4000c014
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                                                                                                                          |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                          | SOTCIPTM                                                                                      |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+---------------------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                                             |
+----------+----------+--------+-------------+---------------------------------------------------------+
| 31:8     | RSVD     |        |             |                                                         |
+----------+----------+--------+-------------+---------------------------------------------------------+
| 7:0      | SOTCIPTM | R      | 0           | Status of the terminal count interrupt prior to masking |
+----------+----------+--------+-------------+---------------------------------------------------------+

DMA_RawIntErrorStatus
-----------------------
 
**Address：**  0x4000c018
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                                                                                                                          |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                          | SOTEIPTM                                                                                      |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+------------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                                    |
+----------+----------+--------+-------------+------------------------------------------------+
| 31:8     | RSVD     |        |             |                                                |
+----------+----------+--------+-------------+------------------------------------------------+
| 7:0      | SOTEIPTM | R      | 0           | Status of the error interrupt prior to masking |
+----------+----------+--------+-------------+------------------------------------------------+

DMA_EnbldChns
---------------
 
**Address：**  0x4000c01c
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                                                                                                                          |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                          | CES                                                                                           |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+-----------------------+
| Bit      | Name     |Type    | Reset       | Description           |
+----------+----------+--------+-------------+-----------------------+
| 31:8     | RSVD     |        |             |                       |
+----------+----------+--------+-------------+-----------------------+
| 7:0      | CES      | R      | 0           | Channel enable status |
+----------+----------+--------+-------------+-----------------------+

DMA_SoftBReq
--------------
 
**Address：**  0x4000c020
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| SBR                                                                                                                                                                                           |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| SBR                                                                                                                                                                                           |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+------------------------+
| Bit      | Name     |Type    | Reset       | Description            |
+----------+----------+--------+-------------+------------------------+
| 31:0     | SBR      | R/W    | 0           | Software burst request |
+----------+----------+--------+-------------+------------------------+

DMA_SoftSReq
--------------
 
**Address：**  0x4000c024
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| SSR                                                                                                                                                                                           |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| SSR                                                                                                                                                                                           |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+-------------------------+
| Bit      | Name     |Type    | Reset       | Description             |
+----------+----------+--------+-------------+-------------------------+
| 31:0     | SSR      | R/W    | 0           | Software single request |
+----------+----------+--------+-------------+-------------------------+

DMA_SoftLBReq
---------------
 
**Address：**  0x4000c028
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| SLBR                                                                                                                                                                                          |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| SLBR                                                                                                                                                                                          |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+-----------------------------+
| Bit      | Name     |Type    | Reset       | Description                 |
+----------+----------+--------+-------------+-----------------------------+
| 31:0     | SLBR     | R/W    | 0           | Software last burst request |
+----------+----------+--------+-------------+-----------------------------+

DMA_SoftLSReq
---------------
 
**Address：**  0x4000c02c
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| SLSR                                                                                                                                                                                          |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| SLSR                                                                                                                                                                                          |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+------------------------------+
| Bit      | Name     |Type    | Reset       | Description                  |
+----------+----------+--------+-------------+------------------------------+
| 31:0     | SLSR     | R/W    | 0           | Software last single request |
+----------+----------+--------+-------------+------------------------------+

DMA_Config
------------
 
**Address：**  0x4000c030
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                                                                                                                          |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                                                                                                  | AHBMEC    | SDMAEN    |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+------------------------------------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                                                            |
+----------+----------+--------+-------------+------------------------------------------------------------------------+
| 31:2     | RSVD     |        |             |                                                                        |
+----------+----------+--------+-------------+------------------------------------------------------------------------+
| 1        | AHBMEC   | R/W    | 0           | AHB Master endianness configuration: 0 = little-endian, 1 = big-endian |
+----------+----------+--------+-------------+------------------------------------------------------------------------+
| 0        | SDMAEN   | R/W    | 0           | SMDMA Enable.                                                          |
+----------+----------+--------+-------------+------------------------------------------------------------------------+

DMA_Sync
----------
 
**Address：**  0x4000c034
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| DSLFDRS                                                                                                                                                                                       |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| DSLFDRS                                                                                                                                                                                       |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+----------------------------------------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                                                                |
+----------+----------+--------+-------------+----------------------------------------------------------------------------+
| 31:0     | DSLFDRS  | R/W    | 0           | DMA synchronization logic for DMA request signals: 0 = enable, 1 = disable |
+----------+----------+--------+-------------+----------------------------------------------------------------------------+

DMA_C0SrcAddr
---------------
 
**Address：**  0x4000c100
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| DMASA                                                                                                                                                                                         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| DMASA                                                                                                                                                                                         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+--------------------+
| Bit      | Name     |Type    | Reset       | Description        |
+----------+----------+--------+-------------+--------------------+
| 31:0     | DMASA    | R/W    | 0           | DMA source address |
+----------+----------+--------+-------------+--------------------+

DMA_C0DstAddr
---------------
 
**Address：**  0x4000c104
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| DMADA                                                                                                                                                                                         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| DMADA                                                                                                                                                                                         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+-------------------------+
| Bit      | Name     |Type    | Reset       | Description             |
+----------+----------+--------+-------------+-------------------------+
| 31:0     | DMADA    | R/W    | 0           | DMA Destination address |
+----------+----------+--------+-------------+-------------------------+

DMA_C0LLI
-----------
 
**Address：**  0x4000c108
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| FLLI                                                                                                                                                                                          |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| FLLI                                                                                                                                                                                          |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+-----------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                                   |
+----------+----------+--------+-------------+-----------------------------------------------+
| 31:0     | FLLI     | R/W    | 0           | First linked list item. Bits [1:0] must be 0. |
+----------+----------+--------+-------------+-----------------------------------------------+

DMA_C0Control
---------------
 
**Address：**  0x4000c10c
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| TCIEN     | PROTECT                           | DI        | SI        | RSVD      | IMTMMODE  | DTW                               | STW                               | DBS                   |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| DBS       | SBS                               | TS                                                                                                                                            |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                                                                                                                   |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 31       | TCIEN    | R/W    | 0           | Terminal count interrupt enable bit. It controls whether the current LLI is expected to trigger the terminal count interrupt. |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 30:28    | PROTECT  | R/W    | 0           | Protection.                                                                                                                   |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 27       | DI       | R/W    | 1           | Destination increment. When set, the Destination address is incremented after each transfer.                                  |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 26       | SI       | R/W    | 1           | Source increment. When set, the source address is incremented after each transfer.                                            |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 25       | RSVD     |        |             |                                                                                                                               |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 24       | IMTMMODE | R/W    | 0           | In Memory-to-memory mode, Set this bit high when Src data size is larger than Dst.                                            |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 23:21    | DTW      | R/W    | 3'B010      | Destination transfer width: 8/16/32                                                                                           |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 20:18    | STW      | R/W    | 3'B010      | Source transfer width: 8/16/32                                                                                                |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 17:15    | DBS      | R/W    | 3'B001      | Destination burst size: 1/4/8/16                                                                                              |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 14:12    | SBS      | R/W    | 3'B001      | Source burst size: 1/4/8/16. Note CH FIFO Size is 16Bytes and SBSize*Swidth should <= 16B                                     |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 11:0     | TS       | R/W    | 0           | Transfer size: 0~4095. Number of data transfers left to complete when the SMDMA is the flow controller.                       |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+

DMA_C0Config
--------------
 
**Address：**  0x4000c110
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                  | LLICOUNT                                                                                                              | RSVD      | HALT      | ACTIVE    | LOCK      |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| TCIM      | IEM       | FLOWCTRL                          | DSTPH                                                     | SRCPH                                                     | CHEN      |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                                                                                                                                                                                                                                                                                                                                                                     |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 31:30    | RSVD     |        |             |                                                                                                                                                                                                                                                                                                                                                                                 |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 29:20    | LLICOUNT | R      | 0           | LLI counter. Increased 1 each LLI run. Cleared 0 when config Control.                                                                                                                                                                                                                                                                                                           |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 19       | RSVD     |        |             |                                                                                                                                                                                                                                                                                                                                                                                 |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 18       | HALT     | R/W    | 0           | Halt: 0 = enable DMA requests, 1 = ignore subsequent source DMA requests.                                                                                                                                                                                                                                                                                                       |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 17       | ACTIVE   | R      | 0           | Active: 0 = no data in FIFO of the channel, 1 = FIFO of the channel has data.                                                                                                                                                                                                                                                                                                   |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 16       | LOCK     | R/W    | 0           | Lock.                                                                                                                                                                                                                                                                                                                                                                           |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 15       | TCIM     | R/W    | 0           | Terminal count interrupt mask.                                                                                                                                                                                                                                                                                                                                                  |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 14       | IEM      | R/W    | 0           | Interrupt error mask.                                                                                                                                                                                                                                                                                                                                                           |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 13:11    | FLOWCTRL | R/W    | 0           | 000: Memory-to-memory (DMA)                                                                                                                                                                                                                                                                                                                                                     |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | 001: Memory-to-peripheral (DMA)                                                                                                                                                                                                                                                                                                                                                 |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | 010: Peripheral-to-memory (DMA)                                                                                                                                                                                                                                                                                                                                                 |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | 011: Source peripheral-to-Destination peripheral (DMA)                                                                                                                                                                                                                                                                                                                          |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | 100: Source peripheral-to-Destination peripheral (Destination peripheral)                                                                                                                                                                                                                                                                                                       |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | 101: Memory-to-peripheral (peripheral)                                                                                                                                                                                                                                                                                                                                          |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | 110: Peripheral-to-memory (peripheral)                                                                                                                                                                                                                                                                                                                                          |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | 111: Source peripheral-to-Destination peripheral (Source peripheral)                                                                                                                                                                                                                                                                                                            |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 10:6     | DSTPH    | R/W    | 0           | Destination peripheral.                                                                                                                                                                                                                                                                                                                                                         |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | [23:22] DAC/ADC                                                                                                                                                                                                                                                                                                                                                                 |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | [11:10] SPI TX/RX                                                                                                                                                                                                                                                                                                                                                               |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | [ 7: 6] I2C TX/RX                                                                                                                                                                                                                                                                                                                                                               |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | [ 3: 0] UART1 TX/RX ; UART0 TX/RX                                                                                                                                                                                                                                                                                                                                               |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 5:1      | SRCPH    | R/W    | 0           | Source peripheral.                                                                                                                                                                                                                                                                                                                                                              |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 0        | CHEN     | R/W    | 0           | Channel enable.                                                                                                                                                                                                                                                                                                                                                                 |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

DMA_C1SrcAddr
---------------
 
**Address：**  0x4000c200
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| SRCADDR                                                                                                                                                                                       |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| SRCADDR                                                                                                                                                                                       |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+--------------------+
| Bit      | Name     |Type    | Reset       | Description        |
+----------+----------+--------+-------------+--------------------+
| 31:0     | SRCADDR  | R/W    | 0           | DMA source address |
+----------+----------+--------+-------------+--------------------+

DMA_C1DstAddr
---------------
 
**Address：**  0x4000c204
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| DSTADDR                                                                                                                                                                                       |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| DSTADDR                                                                                                                                                                                       |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+-------------------------+
| Bit      | Name     |Type    | Reset       | Description             |
+----------+----------+--------+-------------+-------------------------+
| 31:0     | DSTADDR  | R/W    | 0           | DMA Destination address |
+----------+----------+--------+-------------+-------------------------+

DMA_C1LLI
-----------
 
**Address：**  0x4000c208
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| LLI                                                                                                                                                                                           |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| LLI                                                                                                                                                                   | RSVD                  |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+-----------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                                   |
+----------+----------+--------+-------------+-----------------------------------------------+
| 31:2     | LLI      | R/W    | 0           | First linked list item. Bits [1:0] must be 0. |
+----------+----------+--------+-------------+-----------------------------------------------+
| 1:0      | RSVD     |        |             |                                               |
+----------+----------+--------+-------------+-----------------------------------------------+

DMA_C1Control
---------------
 
**Address：**  0x4000c20c
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| I         | PROT                              | DI        | SI        | RSVD                  | DWIDTH                            | SWIDTH                            | DBSIZE                |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| DBSIZE    | SBSIZE                            | TRANSIZE                                                                                                                                      |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                                                                                                                   |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 31       | I        | R/W    | 0           | Terminal count interrupt enable bit. It controls whether the current LLI is expected to trigger the terminal count interrupt. |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 30:28    | PROT     | R/W    | 0           | Protection.                                                                                                                   |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 27       | DI       | R/W    | 1           | Destination increment. When set, the Destination address is incremented after each transfer.                                  |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 26       | SI       | R/W    | 1           | Source increment. When set, the source address is incremented after each transfer.                                            |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 25:24    | RSVD     |        |             |                                                                                                                               |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 23:21    | DWIDTH   | R/W    | 3'B010      | Destination transfer width: 8/16/32                                                                                           |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 20:18    | SWIDTH   | R/W    | 3'B010      | Source transfer width: 8/16/32                                                                                                |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 17:15    | DBSIZE   | R/W    | 3'B001      | Destination burst size: 1/4/8/16                                                                                              |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 14:12    | SBSIZE   | R/W    | 3'B001      | Source burst size: 1/4/8/16. Note CH FIFO Size is 16Bytes and SBSize*Swidth should <= 16B                                     |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 11:0     | TRANSIZE | R/W    | 0           | Transfer size: 0~4095. Number of data transfers left to complete when the SMDMA is the flow controller.                       |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+

DMA_C1Config
--------------
 
**Address：**  0x4000c210
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                                                                                      | H         | A         | L         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| ITC       | IE        | FLOWCTRL                          | DSTPH                                                     | SRCPH                                                     | E         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                                                                                                                                                                                                                                                                                                                                                                     |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 31:19    | RSVD     |        |             |                                                                                                                                                                                                                                                                                                                                                                                 |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 18       | H        | R/W    | 0           | Halt: 0 = enable DMA requests, 1 = ignore subsequent source DMA requests.                                                                                                                                                                                                                                                                                                       |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 17       | A        | R      | 0           | Active: 0 = no data in FIFO of the channel, 1 = FIFO of the channel has data.                                                                                                                                                                                                                                                                                                   |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 16       | L        | R/W    | 0           | Lock.                                                                                                                                                                                                                                                                                                                                                                           |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 15       | ITC      | R/W    | 0           | Terminal count interrupt mask.                                                                                                                                                                                                                                                                                                                                                  |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 14       | IE       | R/W    | 0           | Interrupt error mask.                                                                                                                                                                                                                                                                                                                                                           |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 13:11    | FLOWCTRL | R/W    | 0           | 000: Memory-to-memory (DMA)                                                                                                                                                                                                                                                                                                                                                     |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | 001: Memory-to-peripheral (DMA)                                                                                                                                                                                                                                                                                                                                                 |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | 010: Peripheral-to-memory (DMA)                                                                                                                                                                                                                                                                                                                                                 |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | 011: Source peripheral-to-Destination peripheral (DMA)                                                                                                                                                                                                                                                                                                                          |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | 100: Source peripheral-to-Destination peripheral (Destination peripheral)                                                                                                                                                                                                                                                                                                       |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | 101: Memory-to-peripheral (peripheral)                                                                                                                                                                                                                                                                                                                                          |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | 110: Peripheral-to-memory (peripheral)                                                                                                                                                                                                                                                                                                                                          |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | 111: Source peripheral-to-Destination peripheral (Source peripheral)                                                                                                                                                                                                                                                                                                            |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 10:6     | DSTPH    | R/W    | 0           | Destination peripheral.                                                                                                                                                                                                                                                                                                                                                         |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | [23:22] GPADC                                                                                                                                                                                                                                                                                                                                                                   |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | [21:18] I2S                                                                                                                                                                                                                                                                                                                                                                     |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | [17:14] PDM                                                                                                                                                                                                                                                                                                                                                                     |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | [13:10] SPI                                                                                                                                                                                                                                                                                                                                                                     |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | [ 9: 6] I2C                                                                                                                                                                                                                                                                                                                                                                     |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | [ 5: 0] UART                                                                                                                                                                                                                                                                                                                                                                    |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 5:1      | SRCPH    | R/W    | 0           | Source peripheral.                                                                                                                                                                                                                                                                                                                                                              |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 0        | E        | R/W    | 0           | Channel enable.                                                                                                                                                                                                                                                                                                                                                                 |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

DMA_C2SrcAddr
---------------
 
**Address：**  0x4000c300
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| SRCADDR                                                                                                                                                                                       |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| SRCADDR                                                                                                                                                                                       |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+--------------------+
| Bit      | Name     |Type    | Reset       | Description        |
+----------+----------+--------+-------------+--------------------+
| 31:0     | SRCADDR  | R/W    | 0           | DMA source address |
+----------+----------+--------+-------------+--------------------+

DMA_C2DstAddr
---------------
 
**Address：**  0x4000c304
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| DSTADDR                                                                                                                                                                                       |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| DSTADDR                                                                                                                                                                                       |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+-------------------------+
| Bit      | Name     |Type    | Reset       | Description             |
+----------+----------+--------+-------------+-------------------------+
| 31:0     | DSTADDR  | R/W    | 0           | DMA Destination address |
+----------+----------+--------+-------------+-------------------------+

DMA_C2LLI
-----------
 
**Address：**  0x4000c308
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| LLI                                                                                                                                                                                           |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| LLI                                                                                                                                                                   | RSVD                  |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+-----------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                                   |
+----------+----------+--------+-------------+-----------------------------------------------+
| 31:2     | LLI      | R/W    | 0           | First linked list item. Bits [1:0] must be 0. |
+----------+----------+--------+-------------+-----------------------------------------------+
| 1:0      | RSVD     |        |             |                                               |
+----------+----------+--------+-------------+-----------------------------------------------+

DMA_C2Control
---------------
 
**Address：**  0x4000c30c
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| I         | PROT                              | DI        | SI        | RSVD                  | DWIDTH                            | SWIDTH                            | DBSIZE                |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| DBSIZE    | SBSIZE                            | TRANSIZE                                                                                                                                      |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                                                                                                                   |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 31       | I        | R/W    | 0           | Terminal count interrupt enable bit. It controls whether the current LLI is expected to trigger the terminal count interrupt. |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 30:28    | PROT     | R/W    | 0           | Protection.                                                                                                                   |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 27       | DI       | R/W    | 1           | Destination increment. When set, the Destination address is incremented after each transfer.                                  |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 26       | SI       | R/W    | 1           | Source increment. When set, the source address is incremented after each transfer.                                            |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 25:24    | RSVD     |        |             |                                                                                                                               |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 23:21    | DWIDTH   | R/W    | 3'B010      | Destination transfer width: 8/16/32                                                                                           |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 20:18    | SWIDTH   | R/W    | 3'B010      | Source transfer width: 8/16/32                                                                                                |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 17:15    | DBSIZE   | R/W    | 3'B001      | Destination burst size: 1/4/8/16                                                                                              |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 14:12    | SBSIZE   | R/W    | 3'B001      | Source burst size: 1/4/8/16. Note CH FIFO Size is 16Bytes and SBSize*Swidth should <= 16B                                     |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 11:0     | TRANSIZE | R/W    | 0           | Transfer size: 0~4095. Number of data transfers left to complete when the SMDMA is the flow controller.                       |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+

DMA_C2Config
--------------
 
**Address：**  0x4000c310
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                                                                                      | H         | A         | L         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| ITC       | IE        | FLOWCTRL                          | DSTPH                                                     | SRCPH                                                     | E         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                                                                                                                                                                                                                                                                                                                                                                     |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 31:19    | RSVD     |        |             |                                                                                                                                                                                                                                                                                                                                                                                 |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 18       | H        | R/W    | 0           | Halt: 0 = enable DMA requests, 1 = ignore subsequent source DMA requests.                                                                                                                                                                                                                                                                                                       |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 17       | A        | R      | 0           | Active: 0 = no data in FIFO of the channel, 1 = FIFO of the channel has data.                                                                                                                                                                                                                                                                                                   |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 16       | L        | R/W    | 0           | Lock.                                                                                                                                                                                                                                                                                                                                                                           |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 15       | ITC      | R/W    | 0           | Terminal count interrupt mask.                                                                                                                                                                                                                                                                                                                                                  |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 14       | IE       | R/W    | 0           | Interrupt error mask.                                                                                                                                                                                                                                                                                                                                                           |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 13:11    | FLOWCTRL | R/W    | 0           | 000: Memory-to-memory (DMA)                                                                                                                                                                                                                                                                                                                                                     |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | 001: Memory-to-peripheral (DMA)                                                                                                                                                                                                                                                                                                                                                 |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | 010: Peripheral-to-memory (DMA)                                                                                                                                                                                                                                                                                                                                                 |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | 011: Source peripheral-to-Destination peripheral (DMA)                                                                                                                                                                                                                                                                                                                          |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | 100: Source peripheral-to-Destination peripheral (Destination peripheral)                                                                                                                                                                                                                                                                                                       |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | 101: Memory-to-peripheral (peripheral)                                                                                                                                                                                                                                                                                                                                          |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | 110: Peripheral-to-memory (peripheral)                                                                                                                                                                                                                                                                                                                                          |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | 111: Source peripheral-to-Destination peripheral (Source peripheral)                                                                                                                                                                                                                                                                                                            |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 10:6     | DSTPH    | R/W    | 0           | Destination peripheral.                                                                                                                                                                                                                                                                                                                                                         |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | [23:22] GPADC                                                                                                                                                                                                                                                                                                                                                                   |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | [21:18] I2S                                                                                                                                                                                                                                                                                                                                                                     |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | [17:14] PDM                                                                                                                                                                                                                                                                                                                                                                     |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | [13:10] SPI                                                                                                                                                                                                                                                                                                                                                                     |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | [ 9: 6] I2C                                                                                                                                                                                                                                                                                                                                                                     |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | [ 5: 0] UART                                                                                                                                                                                                                                                                                                                                                                    |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 5:1      | SRCPH    | R/W    | 0           | Source peripheral.                                                                                                                                                                                                                                                                                                                                                              |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 0        | E        | R/W    | 0           | Channel enable.                                                                                                                                                                                                                                                                                                                                                                 |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

DMA_C3SrcAddr
---------------
 
**Address：**  0x4000c400
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| SRCADDR                                                                                                                                                                                       |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| SRCADDR                                                                                                                                                                                       |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+--------------------+
| Bit      | Name     |Type    | Reset       | Description        |
+----------+----------+--------+-------------+--------------------+
| 31:0     | SRCADDR  | R/W    | 0           | DMA source address |
+----------+----------+--------+-------------+--------------------+

DMA_C3DstAddr
---------------
 
**Address：**  0x4000c404
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| DSTADDR                                                                                                                                                                                       |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| DSTADDR                                                                                                                                                                                       |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+-------------------------+
| Bit      | Name     |Type    | Reset       | Description             |
+----------+----------+--------+-------------+-------------------------+
| 31:0     | DSTADDR  | R/W    | 0           | DMA Destination address |
+----------+----------+--------+-------------+-------------------------+

DMA_C3LLI
-----------
 
**Address：**  0x4000c408
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| LLI                                                                                                                                                                                           |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| LLI                                                                                                                                                                   | RSVD                  |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+-----------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                                   |
+----------+----------+--------+-------------+-----------------------------------------------+
| 31:2     | LLI      | R/W    | 0           | First linked list item. Bits [1:0] must be 0. |
+----------+----------+--------+-------------+-----------------------------------------------+
| 1:0      | RSVD     |        |             |                                               |
+----------+----------+--------+-------------+-----------------------------------------------+

DMA_C3Control
---------------
 
**Address：**  0x4000c40c
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| I         | PROT                              | DI        | SI        | RSVD                  | DWIDTH                            | SWIDTH                            | DBSIZE                |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| DBSIZE    | SBSIZE                            | TRANSIZE                                                                                                                                      |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                                                                                                                   |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 31       | I        | R/W    | 0           | Terminal count interrupt enable bit. It controls whether the current LLI is expected to trigger the terminal count interrupt. |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 30:28    | PROT     | R/W    | 0           | Protection.                                                                                                                   |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 27       | DI       | R/W    | 1           | Destination increment. When set, the Destination address is incremented after each transfer.                                  |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 26       | SI       | R/W    | 1           | Source increment. When set, the source address is incremented after each transfer.                                            |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 25:24    | RSVD     |        |             |                                                                                                                               |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 23:21    | DWIDTH   | R/W    | 3'B010      | Destination transfer width: 8/16/32                                                                                           |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 20:18    | SWIDTH   | R/W    | 3'B010      | Source transfer width: 8/16/32                                                                                                |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 17:15    | DBSIZE   | R/W    | 3'B001      | Destination burst size: 1/4/8/16                                                                                              |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 14:12    | SBSIZE   | R/W    | 3'B001      | Source burst size: 1/4/8/16. Note CH FIFO Size is 16Bytes and SBSize*Swidth should <= 16B                                     |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+
| 11:0     | TRANSIZE | R/W    | 0           | Transfer size: 0~4095. Number of data transfers left to complete when the SMDMA is the flow controller.                       |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------------------------------------------+

DMA_C3Config
--------------
 
**Address：**  0x4000c410
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                                                                                      | H         | A         | L         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| ITC       | IE        | FLOWCTRL                          | DSTPH                                                     | SRCPH                                                     | E         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                                                                                                                                                                                                                                                                                                                                                                     |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 31:19    | RSVD     |        |             |                                                                                                                                                                                                                                                                                                                                                                                 |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 18       | H        | R/W    | 0           | Halt: 0 = enable DMA requests, 1 = ignore subsequent source DMA requests.                                                                                                                                                                                                                                                                                                       |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 17       | A        | R      | 0           | Active: 0 = no data in FIFO of the channel, 1 = FIFO of the channel has data.                                                                                                                                                                                                                                                                                                   |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 16       | L        | R/W    | 0           | Lock.                                                                                                                                                                                                                                                                                                                                                                           |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 15       | ITC      | R/W    | 0           | Terminal count interrupt mask.                                                                                                                                                                                                                                                                                                                                                  |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 14       | IE       | R/W    | 0           | Interrupt error mask.                                                                                                                                                                                                                                                                                                                                                           |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 13:11    | FLOWCTRL | R/W    | 0           | 000: Memory-to-memory (DMA)                                                                                                                                                                                                                                                                                                                                                     |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | 001: Memory-to-peripheral (DMA)                                                                                                                                                                                                                                                                                                                                                 |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | 010: Peripheral-to-memory (DMA)                                                                                                                                                                                                                                                                                                                                                 |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | 011: Source peripheral-to-Destination peripheral (DMA)                                                                                                                                                                                                                                                                                                                          |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | 100: Source peripheral-to-Destination peripheral (Destination peripheral)                                                                                                                                                                                                                                                                                                       |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | 101: Memory-to-peripheral (peripheral)                                                                                                                                                                                                                                                                                                                                          |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | 110: Peripheral-to-memory (peripheral)                                                                                                                                                                                                                                                                                                                                          |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | 111: Source peripheral-to-Destination peripheral (Source peripheral)                                                                                                                                                                                                                                                                                                            |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 10:6     | DSTPH    | R/W    | 0           | Destination peripheral.                                                                                                                                                                                                                                                                                                                                                         |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | [23:22] GPADC                                                                                                                                                                                                                                                                                                                                                                   |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | [21:18] I2S                                                                                                                                                                                                                                                                                                                                                                     |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | [17:14] PDM                                                                                                                                                                                                                                                                                                                                                                     |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | [13:10] SPI                                                                                                                                                                                                                                                                                                                                                                     |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | [ 9: 6] I2C                                                                                                                                                                                                                                                                                                                                                                     |
+          +          +        +             +                                                                                                                                                                                                                                                                                                                                                                                 +
|          |          |        |             | [ 5: 0] UART                                                                                                                                                                                                                                                                                                                                                                    |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 5:1      | SRCPH    | R/W    | 0           | Source peripheral.                                                                                                                                                                                                                                                                                                                                                              |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 0        | E        | R/W    | 0           | Channel enable.                                                                                                                                                                                                                                                                                                                                                                 |
+----------+----------+--------+-------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

