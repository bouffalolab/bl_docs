===========
MJPEG
===========

Overview
=====
Motion Joint Photographic Experts Group (MJPEG), a video encoding format, can be accurate to frame editing and multi-layer image processing. The motion video sequence is treated as continuous still images, and this compression format compresses each frame separately and completely. Compressing the original data in the YCbCr format can greatly save the memory space occupied by a frame of image.

Features
=========
- Configurable input formats:
 
   - YCbCr 4:2:2 planar or packed format
   - YCbCr 4:2:0 planar format
   - YCbCr 4:0:0
  
- Configurable arrangement order of Y, Cb and Cr input data
- Free configuration of quantization coefficient table 
- Supports software and linked modes
- Supports swap mode
- Supports kick mode
- Reserves jpg header space and automatically adds jpg tail
- Caches up to 4 images continuously
- Multiple application interrupts are beneficial to flexible use and error prompt

Functional Description
===========
Input Configuration
-----------
You can select the YCbCr format of input data by setting the <REG_YUV_MODE> bit in the register MJPEG_CONTROL_1, including YCbCr 4:2:2 planar or packed format, YCbCr 4:2:0 planar format, and YCbCr 4:0:0 grayscale image. When the YCbCr 4:2:2 packed format is selected, the order of Y, Cb and Cr can be configured by the high 8 bits of the register MJPEG_HEADER_BYTE. When other formats are selected, the order of Cb and Cr can be set by the \<REG_ORDER_U\_EVEN\> bit in the register MJPEG_CONTROL_1. The configuration is shown as follows.

.. figure:: ../../picture/MJPEGYUYVInterleaveOrderConfiguration.svg
   :align: center

Quantization Coefficient Table
-----------
The quantization coefficient table can be freely configured by the user. <reg_q_0_00> to <reg_q_0_3F> represents the quantization table of grayscale Y component, while <reg_q_1_00> to <reg_q_1_3F> represents the quantization table of chrominance information Cb and Cr. In the quantization table, the first column is followed by the second column in descending order from the upper left corner. That is, reg_q_0_00 represents the quantized values of the first column on the first row of Y component, reg_q_0_01 represents that of the first column on the second row of the same, reg_q_0_07 represents that of the first column on the eighth row of the same, reg_q_0_08 represents that of the second column on the first row of the same, and so on.

Software Mode and Linked Mode
--------------------
Setting the <REG_MJPEG_SW_MODE> bit in the register MJPEG_CONTROL_2 to 1 will enable the software mode. In this mode, MJPEG will read the specified frames of original image data from the memory for compression, but the image data must be prepared in advance.

When the <REG_MJPEG_SW_MODE> bit in the register MJPEG_CONTROL_2 is cleared to 0, the linked mode is enabled. In this mode, MJPEG will process the output of CAM as its input by taking 8\*8 data blocks as a unit. After CAM writes 8 lines of data into the memory, MJPEG will start to work. The memory space processed by MJPEG will be released to CAM for reuse, so that CAM can link with MJPEG using the storage space that is less than the size of an image.

Swap Mode
-----------
In the swap mode, MJPEG's storage space will be evenly divided into two blocks. When the space of one block is used up, an interrupt will be generated to inform the software to read out the data, and then MJPEG will write data into the other block. Two blocks are used alternatively, so that you can process data with the storage space that is less than the size of a frame of image.

Kick Mode
-----------
When the <REG_SW_KICK_MODE> bit in the register MJPEG_CONTROL_2 is set to 1, the kick mode is enabled. In this mode, every time one "1" is written to the \<REG_SW_KICK\> bit in the register MJPEG_CONTROL_2, one compression will be performed. The number of compressed rows is determined by the \<REG_SW_KICK_HBLK\> bit in the register MJPEG_YUV_MEM_SW. It should be noted that the kick mode can only be used in the software mode, but not in the linked mode.

Jpg Function
-----------
The jpg function will automatically leave some bytes of space at the beginning of each frame of data and fill in zeros. The size of this space is set by the low 12 bits of the register MJPEG_HEADER_BYTE. If automatic padding of jpg tail is enabled, two bytes of data will be added at the end of each frame of data, and the value of the two bytes is 0xFFD9.

Cached Image Information
-------------
The module contains four FIFOs to record the image address, image size, and quantization coefficient. Every time this module completely writes a frame into the memory, it will record the start address, image size, and quantization coefficient of this frame of image in FIFO. However, when the memory is insufficient or four FIFOs are full, the module will automatically discard the information of the incoming image. In the part where the image information has been taken out, the oldest image information will be removed by performing the pop operation through APB. Then, FIFO will automatically push data to ensure the timing of the image information in FIFO.

Multiple Interrupts (Separately Configured)
-------------------------------------
A. Normal interrupt: You can set to generate an interrupt after a fixed number of images are written

B. Camera interrupt: When CAM overflows as MJPEG's processing speed cannot keep up with the speed of writing in CAM, an interrupt will be generated

C. Memory interrupt: When the memory is rewritten, an interrupt will be generated

D. Frame interrupt: When there are more than 4 unprocessed images, an interrupt will be generated

E. Swap interrupt: When a memory block is full in the swap mode, an interrupt will be generated
