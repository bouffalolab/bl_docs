===============
JDEC
===============

Overview
=====
JDEC can decode the JPG compressed images in memory into the original images, and then convert them into RGB format for display through the YUV2RGB module.

Features
===========
- Supports YUV400/YUV420/YUV422 formats.

- Adjustable quantization coefficient: 1–75, 100.

- Supports swap mode

- Supports Skip Mode, where the JPG header is automatically skipped.

- Buffer with up to 16 JPG images.

- Supports flexible interrupt configuration mode, which can set an interrupt at the specified number of images or all complete interrupt.
   
Functional Description
============
Output Configuration
----------
You can select the YCbCr format of output data by setting the \<REG\_YUV\_MODE> bit in the register JDEC\_CONTROL\_1, including YCbCr4:2:2 planar format, YCbCr4:2:0 planar format, and YCbCr4:0:0 grayscale image. The start address of the output image data is determined by the registers JDEC\_YY\_FRAME\_ADDR and JDEC\_UV\_FRAME\_ADDR. The resolution of the output image data is determined by the register JDEC\_FRAME\_SIZE, in which the \<REG\_FRAME\_HBLK> bit represents the number of 8×8 blocks in the vertical direction and the \<REG\_FRAME\_WBLK> bit represents the number of 8×8 blocks in the horizontal direction.

Quantization Coefficient
----------
The quantization coefficient can be set by the \<REG\_Q\_MODE> bit in the register JDEC\_CONTROL\_1. When the set value does not exceed 75, the quantization coefficient is the set value. When that exceeds 75, the quantization coefficient is 100.

Swap Mode
----------
In Swap Mode, a space with a size of two frames of output images is used as a buffer. When one frame of space is full, the decompressed image data continues to be stored in another frame of space buffer. The two frames of space are used alternately back and forth. This mode can be linked with the Display Module to improve efficiency.

Skip Mode
----------
When the \<REG\_HDER\_SKIP> bit in the register JDEC\_HEADER\_SKIP is set to 1, the Skip Mode is enabled. Then, the JDEC Module takes the value that equals to the start address of JPG image set by the \<REG\_JP\_ADDR> bit in the register JDEC\_FRAM\_PUSH plus the number of byte offset set by the \<REG\_HDER\_SKIP\_BYTE> bit in the register JDEC\_HEADER\_SKIP as the final actual start address for decompressing JPG images. This applies to the scenario of skipping JPG header information.

Buffer
-------
The JDEC input buffer can store the information of up to 16 JPG images. Every time one "1" is written to the \<REG\_JP\_PUSH> bit in the register JDEC\_FRAM\_PUSH, the free space of the buffer decreases. Every time JDEC decompresses a JPG image, this free space increases. The number of JPG images to be decompressed in the current buffer can be obtained by reading the \<JP\_FRAME\_CNT> bit in the register JDEC\_FRAM\_STS. The \<FRAME\_VALID\_CNT> bit in the register JDEC\_CONTROL\_3 indicates the number of decompressed images yet not processed.

Operating Process
----------
- Disable the JDEC Module and clear the REG\_MJ\_DEC\_ENABLE bit in the register JDEC\_CONTROL\_1.

- Set the formats of output image data as YUV422/YUV420/YUV400/…

- Choose whether to enable Swap Mode according to the application scenario.

- Set the quantization coefficient of images.

- Set the start addresses of Y and UV components of the output image in memory respectively.

- Set the numbers of horizontal and vertical 8×8 blocks according to the resolution of image data.

- If you need to enable Skip Mode, set the number of bytes to skip.

- Enable the JDEC Module and set the REG\_MJ\_DEC\_ENABLE bit in the register JDEC\_CONTROL\_1 to "1".

- Write the start address of the input JPG images and the command to cache in the buffer synchronously, that is, writing the start address of JPG images into the register JDEC\_FRAM\_PUSH and setting its \<REG\_JP\_PUSH> bit to "1".
   
Precautions
===============
- The start address of JPG image in memory shall be 8-byte aligned.

- The resolution of JPG images must be an integer multiple of 8.

- Currently, only the standard Huffman coding is supported.

