===========
DPI
===========

Overview
=====
The Display Pixel Interface (DPI), also known as RGB interface, is an interface protocol defined by MIPI Alliance for communicating with display screens. This module has 16-bit parallel data lines, with the system memory acting as the display memory, and drives the display directly by continuously outputting data and synchronization signals.

Features
===========

- Meets MIPI® Alliance standard

- Supports single buffer and ping-pong buffer serving as the display memory

- Supports hardware and software control for ping-pong operation

- Supports the Test Mode

- Programmable timing parameters

- Supported input pixel format (buffer data format):
  
  - YUV422 (interleaved)
  
  - RGB888
  
  - RGB565
  
  - RGBA8888
  
  - YUV420 (planar)

- Supports output pixel format of RGB565

- One programmable interrupt

- Use with OSD

Functional Description
===========

The block diagram of DPI Module is shown as follows.

.. figure:: ../../picture/DPIBlock.svg
   :align: center

   Block Diagram of DPI

DPI Timing
-------------
The 16-bit DPI interface timing is shown as follows:

.. figure:: ../../picture/DPITiming.svg
   :align: center

   DPI Timing


- Vertical Sync (VSYNC): Vertically synchronized timing signal, used to indicate the beginning of each displayed image frame

- Horizontal Sync (HSYNC): Horizontally synchronized timing signal, used to indicate the beginning of each horizontal pixel line

- Pixel Clock (PCLK): Pixel clock, continuously generated; VSYNC, HSYNC, DE, and DB \[15:0] will be sampled at the rising edge of PCLK

- Data Enable (DE): Data enable signal, used to indicate valid pixel data

- Data Bit (DB) \[15:0]: Parallel data bits – 16 bits in total, used to transfer pixel data.

Programmable timing parameters
------------------
The timing parameters of DPI are shown as follows:

.. figure:: ../../picture/DPIParameter.svg
   :align: center

   Timing Parameters

The vertical period (one frame) is equal to the sum of Vsync, VBP, VACT, and VFP. The horizontal period (one line) is equal to the sum of Hsync, HBP, HACT, and HFP. The programmable ranges of the timing parameters are shown in the following table:

.. figure:: ../../picture/DPITable.svg
   :align: center

   Ranges of Timing Parameters

Single Buffer
-------------
The DPI Module can use one memory with a size of "display resolution × number of bytes per pixel" as the display memory. When it starts working, DPI transfers the data in this memory to the Display Module repeatedly and continuously. If changes are made to the data in this memory, the changes may take effect on the current or next frame, depending on whether the changed data is located in memory after or before the location in memory of the data currently being transferred by the DPI.

Ping-Pong Buffer
-------------
The DPI Module can use two memories with each size of "display resolution × number of bytes per pixel" as the display memory. DPI transfers the data in one memory to the Display Module repeatedly and continuously. The control mode for switching the current memory for data transfer to the other memory can be set to hardware or software control. When it is set to hardware control, the data for DPI is provided by the Cam Module (see Cam Module section); DPI switches the current memory in use to another memory after the Cam Module has written a complete frame of data to the memory not involved in transfer, and the data in the current memory is completely transferred. When it is set to software control, the memory currently used by DPI for transfer is set by \<SWAP\_IDX\_SWV>, with "0" and "1" corresponding to two memory blocks respectively.

Test Mode
-------------
In Test Mode, DPI can output images with the same pixel color in the same row and a gradient color between different rows. The Test Mode requires three parameters, namely the minimum value of pixel data VAL\_MIN (16 bit), the maximum value of pixel data VAL\_MAX (16 bit), and the step size STEP (8 bit). DPI uses VAL\_MIN as the value of each pixel in the 1st row, (VAL\_MIN+STEP) as the one in the 2nd row, and (VAL\_MIN+STEP\*2) as the one in the 3rd row…and so on, and returns to VAL\_MIN when the pixel value of the Nth row (VAL\_MIN+STEP\*(N1)) is greater than VAL\_MAX. This cycle repeats until the number of output rows reaches the set value. Taking VAL\_MIN=0x00E0, VAL\_MAX=0x012F, and STEP=0x10 as an example, the display in the Test Mode is shown as follows:

.. figure:: ../../picture/DPITest.svg
   :align: center

   Test Mode

Input Pixel Format
---------------
DPI supports input in multiple pixel formats. Internally, it will convert these formats to RGB888 uniformly and then convert RGB888 to RGB565 for sending out.

YUV422 (Interleaved)
**************
The processing of input data in YUV422 (interleaved) format is shown as follows:

.. figure:: ../../picture/DPIYUV422.svg
   :align: center

   YUV422 Processing Process

RGB888
*********
The processing of input data in RGB888 format is shown as follows:

.. figure:: ../../picture/DPIRGB888.svg
   :align: center

   RGB888 Processing Process

RGB565
*********
The processing of input data in RGB565 format is shown as follows:

.. figure:: ../../picture/DPIRGB565.svg
   :align: center

   RGB565 Processing Process

RGBA8888
***********
The processing of input data in RGBA8888 format is shown as follows:

.. figure:: ../../picture/DPIRGBA8888.svg
   :align: center

   RGBA8888 Processing Process

YUV420 (Planar)
**************
The processing of input data in YUV420 (planar) format is shown as follows:

.. figure:: ../../picture/DPIYUV420.svg
   :align: center

   YUV420 Processing Process

Programmable Interrupt
-------------
The DPI has a programmable interrupt. In configuration, the trigger source of the interrupt can be set to the input Vsync or output Vsync of the DPI, and the hopping edge can be the rising or falling edge. For example, if the interrupt is configured to be triggered on the rising edge of the input Vsync, DPI will generate an interrupt request on each Vsync rising edge of its input module. If that is configured to be triggered on the falling edge of the output Vsync, DPI generates an interrupt request on each Vsync falling edge of its output module.

Use with OSD
----------------
The input data of DSI can be configured to be processed by the OSD first. See the OSD Module section for functional description.
