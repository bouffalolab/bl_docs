===========
SDIO
===========

Overview
=====
Secure Digital Input and Output (SDIO) is a peripheral interface developed on the basis of the SD memory card interface. The SDIO card that is compatible with the SD memory card aims to provide high-speed data I/O and low-power mobile electronic devices.

Features
=========
- For portable and stationary applications

- Minimal modification or no modification to the SD physical bus

- Minimal changes to memory driver software

- Extended physical form factor for special applications

- Plug and play

- Multi-functional support, including multiple I/O, combined I/O and memory

- One card supports up to seven I/O functions and one memory.

- Allow the card to interrupt the host

Functional Description
===========
Definition of SDIO Signal
-------------

SDIO Card Type
*************
There are two types of SDIO cards, full-speed and low-speed SDIO cards. In the full clock range of 0–25 MHz, the full-speed card supports SPI, 1-bit SD and 4-bit SD transfer modes. The data transfer rate of a full-speed SDIO card exceeds 100 Mb/s (10 MB/s). The low-speed SDIO card only needs SPI and 1-bit SD transfer modes and supports the full clock range of 0–400 KHz. The low-speed card intends to provide the low-speed I/O capability with minimum hardware. It supports modem, barcode scanner, GPS receiver, and other functions.

SDIO Card Mode
*************
Here are three signal modes defined for SD memory cards that also apply to SDIO cards. First, SPI mode, where Pin 8 is used as an interrupt pin, and all other pins and signaling protocols are the same as that specified in the SD Physical Layer Specification. Second, 1-bit SD data transfer mode, where data is only transferred on the DAT\[0] pin. Pin 8 is used as an interrupt pin, and all other pins and signaling protocols are the same as that specified in the SD memory specification. Third, 4-bit SD data transfer mode, where data is transferred on all 4 data pins DAT\[3:0]. In this mode, the interrupt pin must not be used exclusively, because it is used as a data transfer line.  Thus, if the interrupt function is required, special timing is required to provide the interrupt. The 4-bit SD mode provides the fastest possible data transfer, up to 100 Mb/s.

Signal Pin
***********
.. figure:: ../../picture/SDIOPin.svg
   :align: center

   Signal Connection of Two 4-Bit SDIO Cards

SDIO Card Initialization
---------------

Difference in I/O Card Initialization
*****************
The SDIO Specification specifies that when SDIO card is inserted, it shall not cause the failure of non-I/O aware host. To prevent I/O functions from being operated in non-I/O aware hosts, the flow chart of SD card recognition mode shall be changed. A new command (IO\_SEND\_OP\_COND, CMD5) is added to replace ACMD41 and initialize SDIO through the I/O aware host. After reset or power-on, all I/O functions on the card are disabled. When CS is Low, the I/O part of the card will not perform any operation except CMD5 or CMD0. If a SD memory is installed on the card, it shall normally respond to all normal forced memory commands. I/O only cards must not respond to ACMD41, so they are initially displayed as MMC cards. I/O only cards must not respond to the CMD1 used to initialize the MMC card, so they are displayed as non-responding cards. Then the host abandons and disables the card. Therefore, the non-aware host does not receive the response from the I/O only card, and forces it into the inactive state. The following figure shows the operation of the I/O card with non-I/O aware host. The solid line is the actual path, and the dotted line part is not executed:

.. figure:: ../../picture/SDIOInit.svg
   :align: center

   SDIO's Response to Non-I/O Aware Initialization

IO\_SEND\_OP\_COND command (CMD5)
******************************
The function of the IO\_SEND\_OP\_COND command (CMD5) for the SDIO card is similar to that of ACMD41 for the SD memory card, and it is used to query the required voltage range of the I/O card. The normal response of CMD5 is R4 in SD or SPI format. The format of CMD5 is shown as follows:

.. figure:: ../../picture/SDIOCMD5.svg
   :align: center

   IO_SEND_OP_COND Command

其中：

- S: Start bit, always 0.

- D: Direction, always 1, indicating transfer from host to card.

- Command Index: Identifies the CMD5 command with a value of 000101b.

- Stuff Bits: Not used, shall be set to 0.

- I/O OCR: Operation Conditions Register. The supported minimum and maximum values for VDD.

- CRC7: 7 bits of CRC data.

- E: End bit, always 1.

IO\_SEND\_OP\_COND Response (R4)
****************************
An SDIO card receiving CMD5 shall respond with a SDIO unique response, R4. The format of R4 for both the SD and SPI modes is:

.. figure:: ../../picture/SDIOR4SD.svg
   :align: center

   Response R4 in SD Mode

.. figure:: ../../picture/SDIOR4SPI.svg
   :align: center

   Response R4 in SPI Mode

- S: Start bit, always 0.

- D: Direction, always 0, indicating the transfer from card to host.

- Reserved: Bits reserved for future use. These bits shall be set to 1.

- C: Set to 1 if the card is ready to operate after initialization.

- Stuff Bits: Not used, shall be set to 0.

- I/O OCR: Operation Conditions Register. The supported minimum and maximum values for VDD.

- E: End bit, always 1.

- Memory Present: Set to 1 if the card also contains SD memory. Set to 0 if the card is I/O only.

- Number of I/O Functions: Indicates the total number of I/O functions supported by this card. The range is 0-7. Note that the common area present on all I/O cards at function 0 is not included in this count. The I/O functions shall be implemented sequentially beginning at function 1.

- Modified R1: The SPI R1 response byte as described in the SD Physical Layer Specification is modified for I/O as follows:

.. figure:: ../../picture/SDIOR1.svg
   :align: center

   Modified R1 Response

Special Initialization Considerations for combo Cards
******************************
The host shall be aware of some special situations when initializing a combo card (SDIO plus SD memory on the same card). This is because an implementation of the combo card could actually use two separate controllers (Memory and I/O) in the same package and share the same bus lines. It is important for the host to both detect and properly configure both parts (controllers) of a combo card in order to prevent conflicts between the SDIO and the SD memory controller. These concerns are due to the different responses to a reset (hard or soft) by the two controllers. Another issue is the value of the RCA (Relative Card Address) that exists within the Memory controller. Note that this consideration is for SD 1-bit and SD 4-bit modes only. In SPI mode, card select/de-select is accomplished using the hardware CS line rather than the RCA.

Differences with SD Memory Specification
----------------------

SDIO Command List
****************
The following figure shows the list of commands accepted by SD memory and SDIO cards when using the SD bus interface.

.. figure:: ../../picture/SDIOSDCmdList.svg
   :align: center

   Command List for SD Mode

The following figure shows the list of commands accepted by SD memory and SDIO cards when using the SPI bus interface.

.. figure:: ../../picture/SDIOSPICmdList.svg
   :align: center

   Command List for SPI Mode

Unsupported SD Memory Commands
*********************
Several commands required for SD memory cards are not supported by either SDIO-only cards or the I/O portion of combo cards. Some of these commands have no use in SDIO cards such as Erase commands and thus are not supported in SDIO. Moreover, there are several commands for SD memory cards that have different commands when used with the SDIO section of a card. The following table lists these SD memory commands and the equivalent SDIO commands.

.. figure:: ../../picture/SDIOUnsupported.svg
   :align: center

   Unsupported SD Memory Commands

Modified R6 Response
***************
The normal response to CMD3 by a memory card is R6, as shown in the following table:

.. figure:: ../../picture/SDIOR6.svg
   :align: center

   R6 Response of CMD3

The card status bits (8–23) are changed when CMD3 is sent to an I/O only card. In this case, the 16 bits of response shall be the SDIO-only values shown in the following table.

.. figure:: ../../picture/SDIOR6Status.svg
   :align: center

   SDIO R6 Status Bit

Reset for SDIO
************
In order to reset all functions within an SDIO card or the SDIO portion of a combo card, a method different than that used for SD memory is defined. The reset command (CMD0) is only used for memory or the memory portion of combo cards.  In order to reset an I/O only card or the I/O portion of a combo card, use CMD52 to write a 1 to the RES bit in the CCCR (bit 3 of register 6). Note that in the SD mode, CMD0 is only used to indicate entry into SPI mode. An I/O only card or the I/O portion of a combo card is not reset by CMD0.

Bus Width
************
For an SD memory card, the bus width for SD mode is set using ACMD6. For an SDIO card, a write to the CCCR using CMD52 is used to select bus width. In the case of a combo card, both selection methods exist. In this case, the host shall set the bus width in both locations by issuing both the ACMD6 and the CCCR write using CMD52 with the same width before starting any data transfer.

Card Detect Resistor
**************
SD memory and I/O cards use a pull-up resistor on DAT\[3] to detect card insertion. The procedure to enable/disable this resistor is different between SD memory and SDIO. SD memory uses ACMD42 to control this resistor while SDIO uses writes to the CCCR using CMD52. In the case of a combo card, both control modes exist and shall be managed by the host. For a combo card, the resistor is enabled only when both the memory and the I/O control registers have the resistor enabled. That is, after a power on, the host shall disable the resistor by sending ACMD42 to the memory controller or a CCCR write to the SDIO controller since the resistor enable is a logical AND of the two enables. After power-up, both locations default to resistor enabled. It is worth noting that after an I/O reset, the I/O resistor enable is not changed.

Data Transfer Block Sizes
******************
SDIO cards may transfer data in either a multi-byte (1 to 512 bytes) or an optional block format, while the SD memory cards are fixed in the block transfer mode. The SD Physical Layer Specification limits the block size for data transfer to powers of 2 (i.e. 512, 1024, 2048) unless using partial read and write. The SDIO Specification allows any block size from 1 byte to 2048 bytes in order to accommodate the various natural block sizes for I/O functions. It is worth noting that an SDIO card function may define a maximum block size or byte count in the CIS that is smaller than the maximum values described above.

Data Transfer Abort
****************
A host communicating with a SD memory card uses CMD12 to abort the transfer of read or write data from/to the card. For an SDIO card, CMD12 abort is replaced by a write to the ASx bits in the CCCR. Normally, the abort is used to stop an infinite block transfer (block count=0). If an exact number of blocks are to be transferred, it is recommended that the host issue a block command with the correct block count, rather than using an infinite count and aborting the data at the correct time.

New I/O Read/Write Commands
----------------------
Two additional data transfer instructions have been added to support I/O. IO\_RW\_DIRECT is a direct I/O command similar to MMC's "fast I/O" command. IO\_RW\_EXTENDED allows fast access with byte or block addresses. Both commands are in class 9 (I/O Commands).

IO\_RW\_DIRECT Command (CMD52)
***************************
The IO\_RW\_DIRECT is the simplest means to access a single register within the 128K of register space in any I/O function, including the common I/O area (CIA). This command reads or writes 1 byte using only 1 command/response pair. A common use is to initialize registers or monitor status values for I/O functions. This command is the fastest means to read or write single I/O registers, as it requires only a single command/response pair. The command structure is shown as follows:

.. figure:: ../../picture/SDIOCMD52.svg
   :align: center

   IO_RW_DIRECT Command

- S: Start bit, always 0.

- D: Direction, always 1, indicating transfer from host to card.

- Command Index: Identifies the "IO\_RW\_DIRECT" command with a value of 110100b.

- R/W Flag: This bit determines the direction of the I/O operation. If this bit is 0, this command shall read data from the SDIO card at the address specified by the Function Number and the Register Address to the host. The data byte is returned in the response, R5. If this bit is set to 1, the command shall write the bytes in the Write Data field to the I/O location addressed by the Function Number and the Register Address. If the RAW flag is 0, then the data in the register that was written shall be read and that value returned in the response.

- RAW Flag: The Read after Write flag. If this bit is set to 1 and the R/W flag is set to 1, then the command shall read the value of the register after the write. This is useful for allowing writing to the control register and reading the state of the same address. If this bit is cleared, the value returned in the R5 response shall be the same as the write data in the command. If this bit is set, the data field of the R5 response shall contain the value read from the addressed register after the write operation.

- Function Number: The number of the function within the I/O card you wish to read or write. Note that function 0 selects the common I/O area (CIA).

- Register Address: This is the address of the byte of data inside of the selected function to read or write. There are 17 bits of address available so the register is located within the first 128K (131,072) addresses of that function.

- Write Data/Stuff Bits: For a direct write command (R/W=1), this is the byte that is written to the selected address. For a direct read (R/W=0), this field is not used and shall be set to 0.

- CRC7: 7 bits of CRC data.

- E: End bit, always 1.

IO\_RW\_DIRECT Response (R5)
**************************
The SDIO card's response to CMD52 shall be in one of two formats. If the communication between card and host is in the 1-bit or 4-bit SD mode, the response shall be in a 48-bit response (R5). If the operation was a read command, the data being read is returned as an 8-bit value. In addition, 15 bits of status information is returned. The format of the response is as follows:

.. figure:: ../../picture/SDIOR5SD.svg
   :align: center

   IO\_RW\_DIRECT Response in SD Mode

- S: Start bit, always 0.

- D: Direction, always 0, indicating the transfer from card to host.

- Command Index: Identifies the "IO\_RW\_DIRECT" command with a value of 110100b.

- Stuff Bits: Not used, shall be set to 0.

- Response Flags: 8 bits of flag data indicating the status of the SDIO card.

- Read or Write Data: For an I/O write (R/W=1) with the RAW Flag set (RAW=1), this field shall contain the value read from the addressed register after the write of the data contained in the command. Note that in this case, the read-back data may not be the same as the data written to the register, depending on the design of the hardware. For an I/O write with the RAW bit=0, the SDIO function shall not do a read after write operation, and the data in this field shall be identical to the data byte in the write command. For an I/O read (R/W=0), the actual value read from that I/O location is returned in this field.

- CRC7: 7 bits of CRC data.

- E: End bit, always 1.

If the communication is using the SPI mode, the response shall be a 16-bit R5 response. If the operation was a read command, the data being read is returned as an 8-bit value. In addition, 8 bits of status information is returned in a SPI R1 response byte. The format of the response is as follows:

.. figure:: ../../picture/SDIOR5SPI.svg
   :align: center

   IO\_RW\_DIRECT Response in SPI Mode

Note: The read/write (R/W) data is identical to the read/write data described for the SD R5 response. Parameter error status in SPI mode corresponds to OUT\_OF\_RANGE and ERROR in the SD mode response. In the case of CMD53, Data Error Token shall also be used to indicate OUT\_OF\_RANGE and ERROR.

IO\_RW\_EXTENDED Command (CMD53)
******************************
In order to read and write multiple I/O registers with a single command, a new command, IO\_RW\_EXTENDED is defined. This command is included in command class 9 (I/O Commands). This command allows the reading or writing of a large number of I/O registers with a single command. Since this is a data transfer command, it provides the highest possible transfer rate. The IO\_RW\_EXTENDED command is shown as follows:

.. figure:: ../../picture/SDIOCMD53.svg
   :align: center

   IO_RW_EXTENDED Command

- S: Start bit, always 0.

- D: Direction, always 1, indicating transfer from host to card.

- Command Index: Identifies the "IO\_RW\_EXTENDED" command with a value of 110101b.

- R/W Flag: This bit determines the direction of the I/O operation. If this bit is 0, this command reads data from the SDIO card at the address specified by the Function Number and the Register Address to the host. The read data shall be returned on the DAT\[x] lines. If this bit is set to 1, the command shall write the bytes from the DAT\[x] lines to the I/O location addressed by the Function Number and the Register Address.

- Function Number: The number of the function within the I/O card you wish to read or write. Note that function 0 selects the common I/O area (CIA).

- Block Mode: (Optional) this bit, if set to 1, indicates that the read or write operation shall be performed on a block basis, rather than the normal byte basis. If this bit is set, the byte/block count value shall contain the number of blocks to be read/written. The block size for functions 1-7 is set by writing the block size to the I/O block size register in the FBR. The block size for function 0 is set by writing to the FN0 block size register in the CCCR. Card and host support of the block I/O mode is optional. The host can determine if a card supports block I/O by reading the Card supports MBIO bit (SMB) in the CCCR. The block size used when Block Mode = 1 and the maximum byte count per command used when Block Mode = 0 can be read from the CIS in the tuple TPLFE\_MAX\_BLK\_SIZE on a per-function basis.

- OP code: Defines the read/write operation. 0 is used to read or write multiple bytes of data to/from a fixed address. 1 is used to read or write multiple bytes of data to/from an incremental address.

- Register Address: Start Address of I/O register to read or write. Range is \[0~0x1FFF].

- Byte/Block Count: If the command is operating on bytes (Block Mode = 0), this field contains the number of bytes to read or write. A value of 0X000 shall cause 512 bytes to be read or written.

- CRC7: 7 bits of CRC data.

- E: End bit, always 1.

SDIO Card Internal Operation
-------------------
I/O access differs from memory in that the registers can be written and read individually and directly without a FAT file structure or the concept of blocks (although block access is supported). These registers allow access to the I/O data, control of the I/O function, report on status or transfer I/O data to the host. SD memory relies on the concept of a fixed block length with commands reading/writing multiples of these fixed size blocks. I/O may or may not have fixed block lengths and the read size may be different from the write size. Because of this, I/O operations may be based on either a length (byte count) or a block size.

Overview
********
Each SDIO card may have from 1 to 7 functions plus one built-in memory function. A function is a self-contained I/O device. I/O functions may be identical or completely different from each other. All I/O functions are organized as a collection of registers. There is a maximum of 131,072 (2\^17) registers possible for each I/O function. These registers and their individual bits may be Read Only (RO), Write Only (WO) or Read/Write (R/W). These registers can be 8, 16 or 32 bits wide within the card. All addressing is based on byte access. These registers can be written and/or read one at a time, multiply to the same address or multiply to an incremental address. The single R/W access is often used to initialize the I/O function or to read a single status or data value. The multiple reads to a fixed address are used to read or write data from a data FIFO register in the card. The read to incremental addresses is used to read or write a collection of data to/from a RAM area inside of the card. The following figure shows the mapping of the CIA and optional CSA space for an SDIO card.

.. figure:: ../../picture/SDIOInternalMap.svg
   :align: center

   SDIO Internal Mapping

Register Access Time
*****************
All registers in SDIO only cards and the SDIO portion of combo cards shall complete read and write data transfers in less than one second. This timeout value relates to the time for the requested data to be transferred to/from the host on the DAT\[X] lines and not the timing between the command and the response. This wait time is signaled to the host by the card using "busy" for a write or delaying the start bit for a read operation. The host can use one second as the timeout value for a non-responding location.

Interrupt
********
All SDIO hosts shall support hardware interrupts. If a host does not support interrupts, it may have difficulties working with SDIO cards that expect fast response to interrupt conditions. Each function within an SDIO or combo card may implement interrupts as needed. The interrupt used on SDIO functions is a type commonly called "level sensitive". Level sensitive means that any function may signal for an interrupt at any time, but once the function has signaled an interrupt, it shall not release (stop signaling) the interrupt until the cause of the interrupt is removed or commanded to do so by the host. Since there is only one interrupt line, it may be shared by multiple interrupt sources. The function shall continue to signal the interrupt until the host responds and clears the interrupt. Since multiple interrupts may be active at once, it is the responsibility of the host to determine the interrupt source(s) and deal with it as needed. This is done on the SDIO function by the use of two bits, the interrupt enable and interrupt suspend. Each function that may generate an interrupt has an interrupt enable bit. In addition, the SDIO card has a master interrupt enable that controls all functions. An interrupt shall only be signaled to the SD bus if both the function's enable and the card's master enable are set. The second interrupt bit is called interrupt suspend. This read-only bit tells the host which function(s) may be signaling for an interrupt. There is an interrupt suspend bit for each function that can generate interrupts. These bits are located in the CCCR area.

Suspend/Resume
*************
Within a multi-function SDIO or a combo card, there are multiple devices (I/O and memory) that share access to the SD bus. In order to allow the sharing of access to the host among multiple devices, SDIO and combo cards can implement the optional concept of Suspend/Resume. If a card supports Suspend/Resume, the host may temporarily halt a data transfer operation to one function or memory (suspend) in order to free the bus for a higher priority transfer to a different function or memory. Once this higher-priority transfer is completed, the original transfer is re-started where it left off (resume). Support of Suspend/Resume is optional on a per-card basis If Suspend/Resume is implemented, it shall be supported by the memory of a combo card and all I/O functions except 0 (the CIA). It is worth noting that the host can suspend multiple transactions and resume them in any order desired. The I/O function 0 does not support Suspend/Resume. Any card that supports Suspend/Resume shall also support Read Wait and Direct Commands (SRW and SDC = 1) It is worth noting that Suspend/Resume is defined only for the SD 1 and 4-bit modes. It does not apply to SPI transfers.

Read Wait
**********
Host devices built based on the SD Physical Layer Specification shall control the SDCLK to stop the read data block output from a card executing a multiple read command whenever the host cannot accept more data. During the time that the host has stopped the SDCLK, a CMD52 cannot be issued. This limitation causes a problem in that a host device built based on the SD Physical Layer Specification cannot perform the I/O command during a multiple read cycle. In order to eliminate this limitation, the SDIO Specification adds the Read Wait control to enable the host to issue CMD52 during a multiple read cycle. Read Wait uses the DAT\[2] line to allow the host to signal the card to temporarily halt the sending of read data by a card. This feature is optional for an SDIO or combo card. However, if an SDIO or combo supports Read Wait, all functions and any memory shall support Read Wait. Any card that supports Suspend/Resume shall also support Read Wait. It is worth noting that Read Wait is defined only for the SD 1 and 4-bit modes. It does not apply to SPI transfers.

CMD52 During Data Transfer
********************
A card may accept CMD52 during data transfer if it supports Direct Commands. For both SD and SPI modes, if an error occurs during data transfer the SDIO card shall accept CMD52 to allow I/O abort and reset regardless of this bit value of the value of SDC.

Fixed Internal Mapping
****************
The SDIO card has a fixed internal register space and a function unique area. The fixed area contains information about the card and certain mandatory and optional registers in fixed locations. The fixed locations allow any host to obtain information about the card and perform simple operations such as Enable in a common manner. The function unique area is a per-function area, which is defined either by the Application Specifications for Standard SDIO functions or by the vendor for non-standard functions.

Common I/O Area (CIA)
*******************
The CIA shall be implemented on all SDIO cards. The CIA is accessed by the host via I/O reads and writes to function 0. The registers within the CIA are provided to enable/disable the operation of the I/O function(s), control the generation of interrupts and optionally load software to support the I/O functions. The registers in the CIA also provide information about the function(s) abilities and requirements. There are three distinct register structures supported within the CIA. They are:

- Card Common Control Registers (CCCR)

- Function Basic Registers (FBR)

- Card Information Structure (CIS)

Card Common Control Registers (CCCR)
***************************
CCCR allows for quick host checking and control of an I/O card's enable and interrupts on a per card (master) and per function basis. The bits in the CCCR are mixed Read/Write and read only. If any of the possible 7 functions are not provided on an SDIO card, the bits corresponding to unused functions shall all be read-only and read as 0. All reserved for future use bits (RFU) shall be read-only and return a value of 0. All writeable bits are set to 0 after power-up or reset. Access to the CCCR is possible even after initialization when the I/O functions are disabled. This allows the host to enable functions after initialization.

Function Basic Registers (FBR)
************************
In addition to the CCCR, each supported I/O function has a 256-byte area used to allow the host to quickly determine the abilities and requirements of each function, enable power selection for each function and to enable software loading. The address of this area is from 0x00n00 to 0x00nFF where n is the function number (0x1 to 0x7).

Card Information Structure (CIS)
********************
The Card Information Structure provides more complete information about the card and the individual functions. The CIS is the common area to read information about all I/O functions that exist in a card. The design is based on the PC Card16 design standardized by PCMCIA. All cards that support I/O shall have a common CIS and a CIS for each function. The CIS is accessed by reading the 0x1000 to 0x17FFF area. This area serves the card as a Common CIS and also as the storage area for each function. The common area and each function have a pointer to the start of its CIS within this memory space.

Multiple Function SDIO Cards
****************
Multiple Function SDIO Cards shall have a separate set of configuration registers for each function on the card. Multiple Function SDIO Cards shall use a combination of a CIS common to all functions on the card and a separate function-specific CIS specific to each function on the card. The common CIS describes features that are common to all functions on the card. Each function-specific CIS describes features specific to a particular function on the SDIO Card. Functions are numbered sequentially beginning with 1. The CMD5 response indicates the total number of functions, which includes 'dummy' functions. The host shall iterate through the CIS entries based on the CMD5 response. The ERROR status flag of an R5 response is type "E R X", and can indicate an error in the previous command. Since the host software needs a method to determine which function detected the error, a Multiple Function SDIO Card shall only return the R5 ERROR status flag in the subsequent command issued to the same function.

Setting Block Size with CMD53
**********************
The host sets the block size for a function's multiple block transfers by writing to the 16-bit function I/O block size register in the FBR. The host shall not write this register using CMD53 with Block Mode set to 1. If the card detects an invalid block size before executing CMD53 with Block Mode set to 1, it shall indicate an OUT\_OF\_RANGE error in the current response and shall not perform data transfer. This will also stop the interrupt cycle 

Bus State Diagram
**************
The following figure shows the Bus State Diagram for an SDIO card. It shows the bus states and their relations to SDIO commands and Suspend/Resume.

.. figure:: ../../picture/SDIOBus.svg
   :align: center

   State Diagram for Bus State Machine

Embedded I/O Code Storage Area (CSA)
-----------------------------
In order to support the concept of "Plug-and-Play" for SDIO cards, each function contained in a card may need to contain a block of memory for the storage of drivers and/or applications. In addition, since the same SDIO card may be used on multiple different host platforms, several different versions of the code may be needed for each function. One option is to store these programs in a standard SD memory section of a combo card. Alternately, a standard access means to load the code is contained in the optional Code Storage Area (CSA). The CSA is a separate 16 MB memory area that is accessed using the CSA address pointer and the CSA window register contained in the FBR registers.

CSA Access
***********
In order for the host to access a function's CSA, it first shall determine if that function supports a CSA. The host reads the FBR register at address 0x00n00 where n is the function number (0x1 to 0x7). If bit 6=1, then the function supports a CSA and the host enables access by writing bit 7=1. The next step is for the host to load the 24-bit address to start reading or writing. This is accomplished by writing the 24 bits (A23-0) to registers 0x00n0C to 0x00n0E where n is the function number (0x1 to 0x7). Once the start address is written, data can be read or written by accessing the register 0x00n0F, the CSA data window register. If more than 1 byte needs to be read or written, an extended I/O command (byte or block) can be performed with an OP code of 0 (fixed address). The address pointer shall be automatically incremented with each access to the window register, so the access will be to sequential addresses within the CSA. Once the operation is completed, the address of the next operation shall be held in the 24-bit address register for the host to read.

CSA Data Format
***************
The data stored in the CSA shall be structured using the FAT12/FAT16 format. The use of the CSA for program or data storage for different host types requires that the SDIO card manufacturers load the programs and data in a file format that may be recognized by the host. An example of this would be the use of a specific file name saved within a specific subdirectory that is recognized and executed by a particular host operating system. Such formats are specific and sometimes proprietary to different host implementations and operating systems.

SDIO Interrupts
-----------
In order to allow the SDIO card to interrupt the host, an interrupt function is added to a pin on the SD interface. Pin number 8, which is used as DAT\[1] when operating in the 4-bit SD mode, is used to signal the card's interrupt to the host. The use of interrupt is optional for each card or function within a card. The SDIO interrupt is "level sensitive". That is, the interrupt line shall be held active (Low) until it is either recognized and acted upon by the host or de-asserted due to the end of the interrupt cycle. Once the host has serviced the interrupt, it is cleared via some function unique I/O operation. All hosts shall provide pull-up resistors on all data lines DAT\[3:0].

SPI and SD 1-Bit Mode Interrupts
***********************
In the SPI and 1-bit SD mode, Pin 8 is dedicated to the interrupt function. Thus, in the SPI and SD 1-bit modes, there are no timing constraints on interrupts. A card in the SPI or 1-bit SD mode signals an interrupt to the host at any time by asserting pin 8 low. The host detects this pending interrupt using a level sensitive input. The host is responsible for clearing the interrupt. If the SDIO card is operating in the SPI mode, the interrupt from the card may not be asserted if the card is not selected. (CS=0). The exception to this requirement occurs only if the card is both capable of interrupting when not selected (the SCSI bit in the CCCR = 1), and has that feature turned on (the ECSI bit = 1). In this case, the card may assert the interrupt irrespective of the state of the CS line.

SD 4-Bit Mode Interrupt
******************
Since Pin 8 is shared between the IRQ and DAT\[1] when used in 4-bit SD mode, an interrupt shall only be sent by the card and recognized by the host during a specific time. The time that a low level on Pin 8 shall be recognized as an interrupt is defined as the interrupt cycle. An SDIO host shall only sample the level on Pin 8 (DAT\[1]/IRQ) into the interrupt detector during the Interrupt Period. At all other times, the host interrupt controller shall ignore the level on Pin 8. Note that the interrupt cycle is applicable for both memory and I/O operations. The definition of the interrupt cycle is different for operations with single block and multiple block data transfer.

Interrupt Clear Timing
****************
Since the SDIO card uses level sensitive interrupts, the host shall clear pending interrupts with an I/O read or write to some function unique area. In some host implementations, the sending of a CMD52 to the card is handled by host adapter hardware while the host CPU can execute other operations. This condition may allow an interrupt that has already been handled to re-interrupt the host if the timing of the interrupt clear is not controlled. To prevent this condition, any SDIO card that implements interrupts shall follow some required timing with respect to removing the interrupt from the DAT\[1] line after the write to the function unique area that clears the interrupt. The clearing of the interrupt can be caused by an I/O write in a function unique method, or by a function unique I/O read. An example of clearing an interrupt using an I/O read would be a function where the reading of a data register may automatically clear the data ready interrupt.

SDIO Suspend/Resume Operation
---------------------
The procedure used to perform the Suspend/Resume operation on the SD bus is:

- The host determines which function is currently using the DAT\[3:0] line(s).

- The host requests the lower priority or slower transaction to suspend.

- The host checks for the transaction suspension to complete.

- The host begins the higher priority transaction.

- The host waits for the completion of the higher priority transaction.

- The host restores the suspended transaction.

If the current transaction can accept suspend and the card receives a Suspend command during Read Wait, it shall accept the Suspend request.

SDIO Read Wait Operation
---------------------
The optional Read Wait (RW) operation is defined only for the SD 1-bit and 4-bit modes. The read Wait operation allows a host to signal a card that is executing a read multiple (CMD53) operation to temporarily stall the data transfer while allowing the host to send commands to any function within the SDIO card. To determine if a card supports the Read Wait protocol, the host shall test the SRW capability bit in the card capability byte of the CCCR. The timing for Read Wait is based on the interrupt cycle. If a card does not support the Read Wait protocol, the only means a host has to stall (not abort) data in the middle of a read multiple command is to control the SDCLK. Read Wait support is mandatory for the card to support Suspend/Resume.
