===========
DBI
===========

Overview
==========
The Display Bus Interface (DBI), also known as MCU interface, is an interface protocol defined by MIPI Alliance for communicating with display screens. The data and control lines of the DBI protocol are multiplexed, and the driven display module must have GRAM.

DBI is subdivided into MIPI DBI Type A, MIPI DBI Type B, and MIPI DBI Type C modes. The hardware interfaces and data sampling in different modes are different. For example, MIPI DBI Type A uses falling edge sampling (based on Motorola 6800 bus), while MIPI DBI Type B uses the rising edge sampling (based on Intel 8080 bus).

Features
===========

- Meets MIPI® Alliance standard

- Supports Type B and Type C modes

- Type B is an 8-bit data interface

- Type C supports 3Line and 4Line

- Supports 8 input pixel formats:

   - RGBA8888

   - BGRA8888

   - ARGB8888

   - ABGR8888

   - RGB888

   - BGR888

   - RGB565

   - BGR565

- Supported output pixel formats: RGB565, RGB666 and RGB888

- Multiple interrupt control

- Supports DMA transfer mode

Functional Description
===========

The block diagram of DBI module is shown as follows.

.. figure:: ../../picture/DBIBlock.svg
   :align: center

   Block Diagram of DBI

DBI Type B
-------------

Write Timing
*************
The DBI Type B mode has 8-bit parallel data lines, and the timing of MCU writing data to the display module is shown as follows:

.. figure:: ../../picture/DBITypeBWrite.svg
   :align: center

   Write Timing

- CSX: chip select signal. When the signal is Low, the display module will be selected; when it is High, the display module will ignore all other interface signals.

- RESX: external reset signal. When the signal is Low, the display module will be reset.

- D/CX: data/command signal. When the signal is 0, the DB[7:0] bit is a command; when the signal is 1, the DB[7:0] bit is the RAM data or command parameter.

- WRX: parallel data write strobe signal. The signal is driven from high to low in the write cycle and then pulled back to high. The display module reads the information transmitted by MCU at the rising edge of the signal. This is an asynchronous signal and can be terminated if not in use.

- RDX: parallel data read strobe signal. The signal is driven from high to low and then pulled back to high in the read cycle. MCU reads the information transmitted by the display module at the rising edge of the signal. This is an asynchronous signal and can be terminated if not in use.

- DB[7:0]: 8-bit data signal, used to transmit commands, command parameters or data

Read Timing
*********
The timing of MCU reading data from the display module is shown as follows:

.. figure:: ../../picture/DBITypeBRead.svg
   :align: center

   Read Timing

Output RGB565
*************
The data output in RGB565 format is shown as follows:

.. figure:: ../../picture/DBITypeBRGB565.svg
   :align: center
   :scale: 80%

   RGB565 Output

Output RGB666
*************
The data output in RGB666 format is shown as follows:

.. figure:: ../../picture/DBITypeBRGB666.svg
   :align: center
   :scale: 80%

   RGB666 Output

Output RGB888
*************
The data output in RGB888 format is shown as follows:

.. figure:: ../../picture/DBITypeBRGB888.svg
   :align: center
   :scale: 80%

   RGB888 Output

DBI Type C 3-Line
-------------------

Write Timing
*********
The DBI Type C mode has 3-line 9-bit serial interface, and the timing of MCU writing data to the display module is shown as follows:

.. figure:: ../../picture/DBITypeC3Write.svg
   :align: center

   Write Timing

- CSX: chip select signal. When the signal is Low, the display module will be selected; when it is High, the display module will ignore all other interface signals.

- SCL: serial clock line, which provides a clock signal for data transfer.

- SDA: serial data line. In a write operation, the serial data packet contains a D/CX (data/command) selection bit and a transfer byte. If the D/CX bit is Low, the transfer byte is a command. If the D/CX bit is High, that is the display data or command parameter.

Read Timing
*********
The timing of MCU reading data from the display module is shown as follows:

.. figure:: ../../picture/DBITypeC3Read.svg
   :align: center

   Read Timing

Output RGB565
*************
The data output in RGB565 format is shown as follows:

.. figure:: ../../picture/DBITypeC3RGB565.svg
   :align: center

   RGB565 Output

Output RGB666
*************
The data output in RGB666 format is shown as follows:

.. figure:: ../../picture/DBITypeC3RGB666.svg
   :align: center

   RGB666 Output

Output RGB888
*************
The data output in RGB888 format is shown as follows:

.. figure:: ../../picture/DBITypeC3RGB888.svg
   :align: center

   RGB888 Output

DBI Type C 4-Line
-------------------

Write Timing
*********
The DBI Type C mode has 4-line 8-bit serial interface, and the timing of MCU writing data to the display module is shown as follows:

.. figure:: ../../picture/DBITypeC4Write.svg
   :align: center

   Write Timing

- CSX: chip select signal. When the signal is Low, the display module will be selected; when it is High, the display module will ignore all other interface signals.

- D/CX: data/command signal. If the signal is Low, the information transferred by SDA is a command. If the signal is High, that is the display data or command parameter.

- SCL: serial clock line, which provides a clock signal for data transfer.

- SDA: serial data line. used to transmit commands, command parameters or data.

Read Timing
*********
The timing of MCU reading data from the display module is shown as follows:

.. figure:: ../../picture/DBITypeC4Read.svg
   :align: center

   Read Timing

Output RGB565
*************
The data output in RGB565 format is shown as follows:

.. figure:: ../../picture/DBITypeC4RGB565.svg
   :align: center

   RGB565 Output

Output RGB666
*************
The data output in RGB666 format is shown as follows:

.. figure:: ../../picture/DBITypeC4RGB666.svg
   :align: center

   RGB666 Output

Output RGB888
*************
The data output in RGB888 format is shown as follows:

.. figure:: ../../picture/DBITypeC4RGB888.svg
   :align: center

   RGB888 Output

Input Pixel Format
-------------------
The DBI module supports 8 different input pixel formats, whose arrangement in the memory is shown as follows: 

.. figure:: ../../picture/DBIMemory.svg
   :align: center

   Input Pixel Format

Interrupt
--------
DBI has the following interrupts:

- End of transfer interrupt

- TX FIFO request interrupt

- FIFO overflow error interrupt

When a pixel count value is set, the end of transfer interrupt will be generated when the transferred pixel data reaches the count value, and both Type B and Type C will generate this interrupt.

When TFICNT in DBI_FIFO_CONFIG_1 is greater than TFITH, the TX FIFO request interrupt will be generated, and the interrupt flag will be automatically cleared when the condition is not met.

The FIFO overflow error interrupt will be generated in the case of Overflow or Underflow of TX.

DMA
--------
DBI TX supports the DMA transfer mode, which requires to set the threshold of TX FIFO by configuring the <TFITH> bit in the register DBI_FIFO_CONFIG_1. When this mode is enabled, if TFICNT is greater than TFITH, the DMA TX request will be triggered. After DMA is configured, when receiving this request, DMA will transfer data from memory to TX FIFO as configured.

.. only:: html

   .. include:: dbi_register.rst

.. raw:: latex

   \input{../../en/content/dbi}