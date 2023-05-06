===========
SDH
===========

Overview
=====
The SD Host Controller (SDH) is used to control the communication with SDIO or SD card, and all accesses are set through standard control registers.

Features
=========
- Comply with the SD Host Controller Specification defined by the SD Association

- Suitable for SDIO/SD memory card

- Supports the standard register settings for the SDH

- Supports the DMA operation for high-speed data transfer

- Supports interrupt

- Supports data block transmission

Functional Description
===========
SDH Overall Structure
---------------

.. figure:: ../../picture/SDHArch.svg
   :align: center

   SDH Hardware and Driver Structure

Register Mapping
--------------

.. figure:: ../../picture/SDHMap.svg
   :align: center

   Standard Register Mapping Classification

- SD Command Generation: parameter used to generate SD command

- Response: the response value from the card

- Buffer Data Port: the data access port of the internal buffer

- Host Control 1 and Others: current status, SD bus control, host reset, etc.

- Interrupt Controls: Interrupt Status and Interrupt Enable

- Host Control 2: vendor-specific host controller support information

- Capabilities: expansion of host control register

- Force Event: a test register that generates events through software

- ADMA: advanced DMA register

- Preset Value: preset values for clock frequency selection and driving strength selection

- Shared Bus: device control of shared bus system

- Common Area: public information area

Support for Multiple Card Slots
--------------
A standard register set is defined for each card slot. If the host controller has two card slots, two register sets are required. Each card slot is independently controlled. This makes it possible to support the combination of bus interface voltage, bus timing and SD clock frequency.

.. figure:: ../../picture/SDHSlot.svg
   :align: center

   Register Mapping of Multi-Card Slot Controller

The above figure shows the register mapping of the multi-card slot host controller. The host driver shall use PCI configuration registers or the vendor-specific methods to determine the number of card slots and the base pointer to the standard register set of each card slot. The offset from 0F0h to 0FFh is reserved for the common register area, which defines the information of card slot control and public state. The common register area can be accessed from the register set of any card slot. This allows the software to control each card slot independently, because it can access the Card Slot Interrupt Status register and the Host Controller Version register from each register set.

Supporting DMA
-----------
The host controller provides a "programmed I/O" method for the host driver to transfer data using the Buffer Data Port register. Optionally, host controller implementers may support data transfer using DMA. Prior to using DMA, the host driver shall confirm that both the host controller and the system bus support it (PCI bus can support DMA). DMA shall support both single block and multiple-block transfers. The host controller registers shall remain accessible for issuing non-DAT line commands during a DMA transfer execution. The result of a DMA transfer shall be the same regardless of the system bus data transfer method. The host driver can stop and restart a DMA operation by the control bits in the block gap control register. By setting Stop At Block Gap Request, a DMA operation can be stopped at block gap. By setting Continue Request, DMA operation can be restarted. If an error occurs, DMA operation shall be stopped. To abort DMA transfer, the host driver shall reset the host controller through the "Software Reset" of DAT line in the software reset register, and issue CMD12 when executing read/write commands on multiple blocks.

SD Command Generation
--------------

.. figure:: ../../picture/SDHCMDGeneration.svg
   :align: center

   Register for Generating SD Commands

The table above shows the register settings (offset from 000h to 00Fh in a register set) required for three types of transactions: DMA-generated transfer (SDMA or ADMA), CPU data transfer (using "programmed I/O") and non-DAT transfer. When starting a transaction, the host driver shall program these registers from 000h to 00Fh. The offset of the starting register can be calculated based on the transaction type. The last written offset shall always be 00Fh, because the high byte written to the command register will trigger the SD command. During a data transaction, the host driver shall not read the SDMA system address, block size and block count registers, unless transfer is stopped or suspended because the values are changing and unstable. To prevent data transfer from damaging registers when commands are issued, the block size, block count and transfer mode registers shall be write-protected by the host controller, and the command disable (DAT) shall be set to 1 in the current status register (this signal cannot protect the SDMA system address). When command disable (CMD) is set to 1, the host driver must not write parameter 1 and command register.

Suspend and Resume Mechanism
------------------
Support for Suspend/Resume can be determined by checking Suspend/Resume Support in the Capabilities register. When the SD card accepts a suspend request, the host driver saves information in the first 14 bytes registers (that is, offsets 000h-00dh) before issuing other SD commands. On resuming, the host driver restores these registers and then issues the Resume command to continue suspended operation. The SDIO card sets the DF (Resume Data Flag) in the response to the Resume command. (Since the Suspend and Resume commands are CMD52 operations, the response data is actually the Function Select Register in the CCCR.) If DF is set to 0, it means that the SDIO card cannot continue data transfer while suspended. This bit can be used to control data transfers and interrupt cycles. If the Resume Data Flag is set to 0, no more data is transferred and an interrupt cycle is started if the transaction being resumed is in 4-bit mode. If DF is set to 1, data transfers continue. It is worth noting that to use Suspend/Resume function, the SDIO card must support Suspend and Resume commands and Read Wait control.

.. figure:: ../../picture/SDHSuspend.svg
   :align: center

   Suspend and Resume Mechanism

Buffer Control
--------------
The host controller has a data buffer for data transfer. The host driver accesses internal buffer through the 32-bit Buffer Data Port register. Here are some rules for accessing the buffer.

Control of Buffer Pointer
*********************
Internally, the host controller maintains a pointer to control the data buffer. The pointer is not directly accessible by the host driver. Every time the Buffer Data Port register is accessed, the pointer is incremented depending on amount of data written to the buffer. In order to accommodate a variety of system buses, this pointer shall be implemented regardless of system bus width (8/16/32/64-bit system bus width can be supported).

Determination of Buffer Block Length
*******************
To transfer data blocks at a burst, the relationship between host controller and SD card buffer sizes is very important. The host driver shall use the same data block length for both host controller and SD card. If the controller and card buffer sizes are different, the host driver shall use the smaller one. The maximum host controller buffer size is defined by the Max Block Length field in the Capabilities register.

Dividing Large Data Transfer
*******************
The SDIO command CMD53 defines the maximum data size for transfer according to the following formula: Maximum data size = block size x block count. For example, the block size is specified by the buffer size, and the maximum value of block count is 512 (9-bit count), as specified in the command parameter CMD53. In the worst case, if the card only has a 1-byte buffer, CMD53 can be used to transfer 512 bytes of data at most (block size = 1, block count = 512). If the card does not support the multi-block mode, only one byte can be transferred. If an application or card driver wants to transfer data with a larger size, the host driver shall divide the large-size data into multiple CMD53 blocks.

Relationship Between Interrupt Control Registers
-----------------------------
The host controller implements a number of interrupt sources. Interrupt sources can be enabled as interrupts or as system wakeup signals. If the interrupt source's corresponding bit in the Normal Interrupt Status Enable or Error Interrupt Status Enable register is 1 and the interrupt becomes active, its active state is latched and made available to the host driver in the Normal Interrupt Status register or the Error Interrupt Status register. Interrupt Status shall be cleared when Interrupt Status Enable is cleared. An interrupt source with its bit set in an Interrupt Status register shall assert a system interrupt signal if its corresponding bit is also set in the Normal Interrupt Signal Enable register or the Error Interrupt Signal Enable register. Once signaled, most interrupts are cleared by writing a 1 to the associated bit in the Interrupt Status register. Card interrupts, however, shall be cleared by the card driver. If a card interrupt is generated, the host driver may clear Card Interrupt Status Enable to disable card interrupts while the card driver is processing them. After all interrupt sources are cleared, the host driver sets it again to enable another card interrupt. Disabling the Card Interrupt Status Enable avoids generating multiple interrupts during interrupt service processing. The Wakeup Control register enables Card Interrupt, Card Insertion, or Card Removal status changes to be configured to generate a system wakeup signal. These interrupts are enabled or masked independently of the Normal Interrupt Signal Enable register. The kind of wakeup event can be read from the Normal Interrupt Status register. The interrupt signal and wakeup signal are logical ORed and shall be read from the Slot Interrupt Status register.

Hardware Block Diagram and Timing Part
-----------------------

.. figure:: ../../picture/SDHDiagram.svg
   :align: center

   Block Diagram of Host Controller

The host controller has two bus interfaces, the system bus interface and the SD bus interface. The host controller assumes that these interfaces are asynchronous, that is, working at different clock frequencies. The host driver is on system bus time, because it is software executed by the host controller CPU on its system clock. The SD card is on SD bus time. That is, its operation is synchronized by SDCLK. The host controller shall synchronize signals to communicate between these interfaces. Data blocks shall be synchronized at the buffer module. All status registers shall be synchronized by the system clock and maintain synchronization during output to the system interface. Control registers, which trigger SD bus transactions, shall be synchronized by SDCLK. Therefore, there will be a timing delay when propagating signals between the two interfaces. This means that the host driver cannot do real time control of the SD bus and needs to rely on the host controller to control the SD bus according to register settings. The buffer interface enables internal read and write buffers. The transfer complete interrupt status indicates completion of the read/write transfer for both DMA and non-DMA transfers. However, this timing is different between reads and writes. Read transfers shall be completed after all valid data have been transferred to the host system and are ready for the host driver to access. Write transfers shall be completed after all valid data have been transferred to the SD card and the busy state is over.

Auto CMD12
-------------
Multiple block transfers for SD memory require CMD12 to stop the transactions. The host controller automatically issues CMD12 when the last block transfer is completed. This feature of the host controller is called Auto CMD12. The host driver shall set Auto CMD12 Enable in the Transfer Mode register when issuing a multiple block transfer command. Auto CMD12 timing synchronization with the last data block shall be done by hardware in the host controller. Commands that do not use the DAT line can be issued during multiple block transfers. These commands are referred to using the notation CMD\_wo\_DAT. To prevent DAT line commands and CMD\_wo\_DAT commands from conflicting, the host controller shall arbitrate the timing by which each command is issued on the SD Bus. Therefore, a command might not immediately be issued after the host driver writes to the Command register. The command may be issued before or after Auto CMD12, depending on the timing. To be able to distinguish the responses of DAT line and CMD\_wo\_DAT commands, the Auto CMD12 response can be determined from the upper four bytes of the Response register (at offset 01Ch in the standard register set). If errors related to Auto CMD12 are detected, the host controller shall issue an Auto CMD error interrupt. The host driver can check the Auto CMD12 error status (Command Index/End bit/CRC/Timeout Error) by reading the Auto CMD Error Status register. If the Auto CMD12 was not executed, the host driver needs to recover from the CMD\_wo\_DAT error and issue CMD12 to stop the multiple block transfer. If the CMD\_wo\_DAT was not executed, the host driver can issue it again after recovering from the Auto CMD12 error. In UHS mode SDR104, the host driver shall use Auto CMD23 to stop multiple block read/write operation instead of using Auto CMD12. In other bus speed modes, if the card supports CMD23, the host driver shall use Auto CMD23 instead of using CMD12.

Controlling SDCLK
-------------
This table shows how SDCLK is controlled by the SD Bus Power in the Power Control register and the SD Clock Enable in the Clock Control register.

.. figure:: ../../picture/SDHSDCLK.svg
   :align: center

   Controlling SDCLK by the SD Bus Power and SD Clock Enable

The clock period of SDCLK is specified by the SDCLK Frequency Select in the Clock Control register and the Base Clock Frequency For SD Clock in the Capabilities register. Since the SD card may use both clock edges, the duty ratio of SD clock shall be average 50% (scattering within 45-55%) and the Period of High should be half of the clock period. The oscillation of SDCLK starts from driving specified Period of High. When SDCLK is stopped by the SD Clock Enable, the host controller shall stop SDCLK after driving Period of High to maintain the duty ratio of clock. When SDCLK is stopped by the SD Bus Power, the host controller shall stop SDCLK immediately (drive Low) and SD Clock Enable shall be cleared.

Advanced DMA
------------
The SD Host Controller Standard Specification Version 2.00 defines the advanced DMA (ADMA) transfer algorithm. The DMA algorithm defined in the SD Host Controller Standard Specification Version 1.00 is called single DMA (SDMA). The disadvantage of SDMA is that the DMA interrupt generated at every page boundary disturbs CPU to reprogram the new system address. This SDMA algorithm develops a performance bottleneck by interrupting at each page boundary. ADMA adopts the scatter gather DMA algorithm so that higher data transfer speed is available. Before executing ADMA, the host driver can program a list of data transfers between system memory and SD card to the descriptor table. It enables ADMA to run without interrupting the host driver. In addition, ADMA supports both 32-bit system memory addressing and 64-bit system memory addressing. The 32-bit system memory addressing uses the lower 32-bit field of the 64-bit address register. ADMA is divided into ADMA1 and ADMA2. ADMA1 only supports the transfer of 4KB aligned data in the system memory. ADMA2 optimizes this restriction, so that the data of any size at any location can be transferred in the system memory. The formats of descriptor tables are different among them. In this document, the term "ADMA" refers to ADMA2.

Block Diagram of ADMA2
***************

.. figure:: ../../picture/SDHADMADiagram.svg
   :align: center

   Block Diagram of ADMA2

The descriptor table is created by the host driver in the system memory. The 32-bit address descriptor table is used for the 32-bit addressing system, and 64-bit address descriptor table is used for the 64-bit addressing system. Each descriptor line (one executable unit) consists of address, length and attribute fields. This attribute specifies the operation of the descriptor line. ADMA2 consists of SDMA, state machine and register circuit. ADMA2 uses the 64-bit advanced DMA system address register (offset 058h) as the descriptor pointer, instead of the 32-bit SDMA system address register (offset 0). Writing to the command register triggers the termination of ADMA2 transfer. ADMA2 takes a descriptor line and executes it. This process is repeated until the end of the descriptor (the attribute "end" equals to 1) is found.

Data Address and Data Length Requirements
***************************
There are three requirements for programming descriptors:

- The minimum unit of address is 4 bytes.

- The maximum data length of each descriptor line is less than 64 KB.

- Total length = length 1 + length 2 + length 3 + ...+ length n = multiple of block size.

If the total length of the descriptor is not a multiple of the block size, the ADMA2 transfer may not be terminated. In this case, the transfer shall be terminated by data timeout. The block count register allows the transfer of maximum 65,535 blocks. If the ADMA2 operation is less than or equal to the transfer of 65,535 blocks, the block count register can be used. In this case, the total length of the descriptor table shall be equal to the product of the block size and the block count. If the ADMA2 operation exceeds the transfer of 65,535 blocks, the block count register shall be disabled by setting Block Count Enable to 0 in the transfer mode register. In this case, the length of data transfer is not specified by the block count but by the descriptor table.  Thus, the timing of detecting the last block on the SD bus may be different, and it affects the control of read transfer activity, write transfer activity and DAT line activity in the current status register. In the case of a read operation, multiple blocks may be read. If the read operation is for the last memory area, the host driver shall ignore the OUT\_OF\_RANGE error.

Descriptor Table
*************

.. figure:: ../../picture/SDHADMADescriptor.svg
   :align: center

   32-Bit Address Descriptor Table

The above figure shows the definition of the 32-bit address descriptor table. One descriptor line consumes 64 bits (8 bytes) of memory space. The attribute is used to control descriptors. Three action symbols are specified here. The "Nop" operation skips the current descriptor line and goes to the next line. The "Tran" operation transfers the data specified by the address and length fields. The "Link" operation is used to link two separate descriptors. The link address field points to the next descriptor table. The combination of Act2=0 and Act1=1 is reserved and defined as the same operation as "Nop". Future versions of the controller may use this field and redefine new operations. The 32-bit address is stored in the lower 32 bits of the 64-bit address register. For a 32-bit address descriptor table, the address field shall be set on the 32-bit boundary (the lower 2 bits are always set to 0).

ADMA2 State
*************
This figure illustrates the four states of ADMA2: fetch descriptor state, change address state, transfer data state and stop ADMA state:

.. figure:: ../../picture/SDHADMAState.svg
   :align: center

   ADMA2 States

- ST\_FDS (fetch descriptor): ADMA2 fetches the descriptor line and sets the parameters in the internal register, and then switches to the ST\_CADR state.

- ST\_CADR (change address): The "Link" operation loads another descriptor address into the ADMA system address register. In other operations, the ADMA system address register is incremented to the next descriptor line. If End=0, it switches to the ST\_TFR state. Even if some errors occur, ADMA2 must not stop in this state.

- ST\_TFR (transfer data): The data of one descriptor line is transferred between system memory and SD card. If data transfer continues (End=0), it switches to the ST\_FDS state. If data transfer is completed, it switches to the ST\_STOP state.

- ST\_STOP (stop DMA): ADMA2 will remain in this state in these cases: after power-on reset or software reset; all the descriptor data has been transferred. It switches to the ST\_FDS state if a new ADMA2 operation is started by writing to the command register.

ADMA2 does not support the Suspend/Resume function, but can use the Stop and Resume functions. When the block gap stop request in the block gap control register is set during ADMA2 operation, the block gap event interrupt will be generated when ADMA2 stops at the block gap. The host controller shall use the Read Wait or Stop SD Clock to stop the ADMA2 read operation. When ADMA2 is stopped, no SD command can be issued. Errors occurred during ADMA2 transfer may stop ADMA2 operation and generate an ADMA error interrupt. The ADMA Error Status field in the ADMA Error Status register stores the status that ADMA2 has stopped. The host driver can identify the location of the error descriptor in two ways. That is, if ADMA stops in the ST\_FDS state, the ADMA system address register will point to the error descriptor line. If ADMA stops in the ST\_TFR or ST\_STOP state, the ADMA system address register will point to the next line of the error descriptor line. Therefore, ADMA2 must not stop in the ST\_CADR state.

Test Register
--------------
The test register is defined for the purpose of testing. When it is difficult to intentionally generate some interrupts, you may use this function to manually generate such interrupts for driver debugging. To that end, a force event register is defined to control the Error Interrupt Status and the automatic CMD error status. It is also difficult to control card insertion and removal intentionally. The card detection signal selection and card detection test level in the host control 1 register make it possible to manually control the card inserted in the current status register and generate card insertion and card removal interrupts in the Normal Interrupt Status register.

Block Count
-----------
The block count command (CMD23) provides an untimed method to stop multi-block operations. The block count is set in the parameter of CMD23 to specify the transfer length of CMD18 or CMD25 after that. Automatic CMD23 is the function of automatically sending CMD23 before sending CMD18 or CMD25. This function aims to avoid performance degradation during memory access by deleting the interrupt service of CMD23. The offset 008h parameter 1 register is used for CMD18 or CMD25. Then an offset of 000h is assigned to the parameter 2 register of CMD23. The host controller does not use the parameter 2 register to calculate the data transfer length. There are two cases of data length of the host-side data transfer operation: non-ADMA and ADMA. The total length of AMDA descriptor refers to the sum of all the 16-bit data length of each ADMA descriptor line. The "block count enable" shall be disabled for ADMA. It is worth noting that the total data transfer length of the host controller shall be equal to the transfer length of the card.

Sampling Clock Tuning
----------------
In UHSI mode, the SD bus can run in the high clock frequency mode, and then the data window of cards on CMD and DAT\[3:0] lines becomes smaller. The position of the data window varies with the implementation of the card and host system. Hence, when SDR104 or SDR50 is supported by executing the tuning program and adjusting the sampling clock (if the usage tuning of SDR50 is set to 1 in the capability register), the host controller shall support the tuning circuit. The execution tuning and sampling clock selection in the host control 2 register is used to control the tuning circuit.

SD Host Standard Register
--------------------

SD Host Control Register Mapping
*************************
The following table summarizes the standard SD host controller register sets. The host driver needs to determine the base address of the register set by the host system specific method. The size of the register set is 256 bytes. For multiple card slot controllers, one register set is assigned to each card slot, but the register at the offset 0F0h0FFh is assigned as a common area, and these registers contain the same value from each card slot register set.

.. figure:: ../../picture/SDHRegisterMap.svg
   :align: center

   SD Host Control Register Mapping

Configuration of Register Type
*******************
The configuration register field is assigned an attribute described in the following table:

.. figure:: ../../picture/SDHRegisterType.svg
   :align: center

   Types of Registers and Register Bit Fields

Initial Value of Register
*****************
The host controller sets all registers to their initial values at power-on reset, and the default values of all other registers shall be all bits that are set to zero. The values of the capability register and the maximum current capability register depend on the host controller, and the value of the host controller version register also depends on the host controller.

Reserved Bits of a Register
******************
"Reserved" means that this bit can be defined for future use, and it is currently set to 0. These bits shall be set to 0.
