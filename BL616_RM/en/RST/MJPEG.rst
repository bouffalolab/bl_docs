===========
MJPEG
===========

Overview
=====
MJPEG (Motion Joint Photographic Experts Group) is a video coding format that can be accurate to frame editing and multi-layer image processing. It handles motion video sequences as continuous still images. This compression method compresses each frame individually and completely. . By compressing the original data in YCbCr format, the memory space occupied by one frame of image can be greatly reduced.

Features
=========
- Configurable input formats including:
 
   * YCbCr4:2:2 plane or packed mode
   * YCbCr4:2:0 plane mode
   * YCbCr4:0:0
  
- Configurable input data Y, Cb, Cr sort order
- Quantization coefficient table can be freely configured
- Support software mode and linkage mode
- Support swap mode
- Support kick mode
- Reserve jpg head space and automatically add jpg tail
- Continuously cache up to 4 sets of picture information
- A variety of application interruptions are conducive to flexible use and error prompts

Function description
===========
Input configuration
-----------
The YCbCr format of the input data can be selected by bit <REG_YUV_MODE> of register MJPEG_CONTROL_1, including YCbCr4:2:2 plane or packed mode, YCbCr4:2:0 plane mode and YCbCr4:0:0 grayscale image. When the YCbCr4:2:2 packing mode is selected, the arrangement order of Y, Cb and Cr can be configured through the upper 8 bits of the register MJPEG_HEADER_BYTE. When other modes are selected, the order of Cb and Cr can be set by bit <REG_ORDER_U_EVEN> of register MJPEG_CONTROL_1. The detailed configuration is shown in the figure below.

.. figure:: ../../picture/MJPEGYUYVInterleaveOrderConfiguration.svg
   :align: center

Quantization coefficient table
-----------
The quantization coefficient table can be freely configured by the user, <reg_q_0_00> to <reg_q_0_3F> represent the quantization tables of the Y component of grayscale information, and <reg_q_1_00> to <reg_q_1_3F> represent the quantization tables of the chrominance information Cb and Cr. The quantization table is arranged in the order from the upper left corner to the bottom, the first column is followed by the second column, that is, reg_q_0_00 represents the quantization value of the first row and the first column of the Y component, and reg_q_0_01 represents the quantization of the second row and the first column of the Y component. Value, reg_q_0_07 represents the quantized value of the Y component in the eighth row and first column, reg_q_0_08 represents the quantized value in the first row and second column of the Y component, and so on.

Software mode and link mode
--------------------
When the bit <REG_MJPEG_SW_MODE> of the MJPEG_CONTROL_2 register is set to 1, the software mode is enabled. In this mode, MJPEG will read the original image data of the specified number of frames from the memory for compression. In this mode, the image data needs to be prepared in advance.

When the bit <REG_MJPEG_SW_MODE> of the MJPEG_CONTROL_2 register is cleared to 0, the linkage mode is enabled. In this mode, MJPEG will process the output of the CAM module as its input. MJPEG is processed with 8*8 data blocks as a unit. When the CAM finishes writing 8 lines of data to the memory, MJPEG will start to work, and the memory space processed by MJPEG will be released to the CAM for reuse, so that the CAM can complete the linkage with MJPEG without the memory space of a whole picture.

Swap mode
-----------
When using swap mode, the MJPEG storage space will be evenly divided into two blocks, when MJPEG finishes writing one block, an interrupt will be generated to notify the software to read the data, and it will write the data to the other block. Alternate use back and forth, so as to use less than one frame of picture storage space for data processing.

Kick mode
-----------
The kick mode is enabled when bit <REG_SW_KICK_MODE> of the MJPEG_CONTROL_2 register is set to 1. In this mode, each time a 1 is written to the bit <REG_SW_KICK> of the MJPEG_CONTROL_2 register, a compression is started. The number of compressed lines is determined by the bit <REG_SW_KICK_HBLK> of the MJPEG_YUV_MEM_SW register. It should be noted that the kick mode can only be used in software mode, not in linkage mode.

Jpg function
-----------
The jpg function will automatically reserve a certain number of bytes of space at the beginning of each frame of data and fill it with the specified data. The size of this space is set by the lower 12 bits of the register MJPEG_HEADER_BYTE. There is 768 bytes of memory at offset 0x800 for storing the data that needs to be filled at the beginning of each frame. The user only needs to write the jpg header information into it, and this information will be included at the beginning of each frame of data generated. In addition, if the auto-fill jpg end is enabled, two bytes of data will be added at the end of each frame of data, and the value of these two bytes is 0xFFD9.

Cache image information
-------------
The module contains 4 groups of FIFO to record the picture address, picture size and picture ID. Whenever this module completely writes a frame to the memory, it will record the start address, image size and image ID of the frame image in this FIFO, but it should be noted that when the memory is insufficient, or there are 4 sets of FIFO When the storage is full, the module will automatically discard the information of the next picture. In the part where the picture information is taken out, the pop action can be performed through the APB interface to empty the oldest picture information. At this time, the FIFO will be automatically advanced to ensure the FIFO. Timing of internal picture information.

Support a variety of interrupt information (can be configured independently of the switch)
-------------------------------------
A. Normal interrupt - can be set to interrupt after writing several pictures

B. Camera Interrupt - When the speed of MJPEG processing cannot keep up with the speed of CAM module writing, and the CAM overflows, an interrupt is issued

C. Memory interrupt - when the memory is overwritten, an interrupt is issued

D. Frame interrupt - when there are more than 4 groups of unprocessed pictures, an interrupt is issued

E. Swap interrupt - when a memory block is full in swap mode, an interrupt is issued
