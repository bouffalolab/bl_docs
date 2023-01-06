==========
DMA2D
==========

Overview
=====
The 2 Dimensional Direct Memory Access (DMA2D or 2D DMA) is a technique extended from 1D DMA that allows a 2D DMA controller to transfer data in the set strides (see the DMA section for details about 1D DMA). The memory address of data can be non-contiguous, incremented or decremented at a certain interval, or unchanged. The 2D DMA controller has 2 independent dedicated lanes, managing data transmission between peripherals and memory to enhance bus efficiency. It supports the transfer from memory to memory, memory to peripherals, and peripheral to memory, and provides the LLI linked list function.

Features
=========
- Two dedicated independent lanes

- Configurable as 1D DMA or 2D DMA

- Independent control source and destination access width (single byte, double bytes, and four bytes)

- Each lane acts as a read-write cache independently

- Each lane can be triggered by independent peripheral hardware or software

- Supported peripherals: UART, I2C, SPI, DBI, and DSI

- Eight kinds of process control
  
  - DMA process control, source memory, destination memory
  
  - DMA process control, source memory, destination peripherals
  
  - DMA process control, source peripherals, destination memory
  
  - DMA process control, source peripherals, destination peripherals
  
  - Destination peripheral process control, source peripherals, destination peripherals
  
  - Destination peripheral process control, source memory, destination peripherals
  
  - Source peripheral process control, source peripherals, destination memory
  
  - Source peripheral process control, source peripherals, destination peripherals

- Supports the LLI linked list function to improve DMA efficiency, and configures whether to trigger interrupt for each node separately

- Exclusive features of 2D DMA:
  
  - Supports translation of 8/16/32-bit images in any direction
  
  - Supports rotation of 8/16/32-bit images at 90°/180°/270°
  
  - Supports horizontal/vertical folding of 8/16/32-bit images
  
  - Supports filling by 8/16/32-bit images
  
  - Supports 8/16/24/32-bit Color Key

Functional Description
=========
Operating Principle
---------
On the basis of 1D DMA, 2D DMA adds the SRC XINCR, SRC XCOUNT, SRC YINCR, and SRC YCOUNT; and adds the DEST XINCR, DEST XCOUNT, and DEST YINCR. DEST YCOUNT will not be configured, as it will be derived from other values. Here the stride value can be negative. The 2D DMA transfer can be viewed as a nested loop, where the outer loop is specified by YCOUNT and each time the loop address increments by YINCR bytes (decrements when YINCR is negative); the inner loop is specified by XCOUNT and each time the loop address increments XINCR bytes (decrements when XINCR is negative). The working principle is illustrated as follows:

.. figure:: ../../picture/DMA2DMemory.svg
   :align: center

   Operating Principle

Image Translation
----------
If the memory data is considered as a rectangular image, the image can be translated by setting the stride and loop values, as shown below:

.. figure:: ../../picture/DMA2DTranslate.svg
   :align: center

   Image Translation

As shown in the figure, it is assumed that a pixel width (P) is 8 bit, and the transfer width of 2D DMA is set to 8 bit. In an M × N image (30 × 8 in the figure), a rectangular area of W × H (3 × 5 in the figure) is translated from the coordinate (x1, y1) \[(3, 1) in the figure] to the coordinate (x2, y2) \[(21, 2) in the figure]. The start address of image data starts from addr0 (0 in the figure). Relevant parameters of 2D DMA are calculated as follows:

- SRC ADDR START = addr0+(M\*y1+x1)\*P = 0+(30\*1+3)\*1 = 33

- SRC XINCR = P = 1

- SRC XCOUNT = W = 3

- SRC YINCR = (M-(W-1))*P = (30-(3-1))*1 = 28

- SRC YCOUNT = H = 5

- DEST ADDR START = addr0+(M\*y2+x2)\*P = 0+(30\*2+21)\*1 = 81

- DEST XINCR = P = 1

- DEST XCOUNT = W = 3

-  DEST YINCR = (M-(W-1))*P = (30-(3-1))*1 = 28

Image Rotation
----------
If the memory data is considered as a rectangular image, the image can be rotated at 90°/180°/270° by setting the stride and loop values, as shown below:

.. figure:: ../../picture/DMA2DRotate.svg
   :align: center

   Image Rotation

As shown in the figure, it is assumed that a pixel width (P) is 8 bit, and the transfer width of 2D DMA is set to 8 bit. In an M × N image (30 × 8 in the figure), a rectangular area of W × H (3 × 5 in the figure) is rotated clockwise at 90° from the coordinate (x1, y1) \[(3, 1) in the figure] to the coordinate (x2, y2) \[(14,5) in the figure]. The start address of image data starts from addr0 (0 in the figure). Relevant parameters of 2D DMA are calculated as follows:

- SRC ADDR START = addr0+(M\*y1+x1)\*P = 0+(30\*1+3)\*1 = 33

- SRC XINCR = P = 1

- SRC XCOUNT = W = 3

- SRC YINCR = (M-(W-1))*P = (30-(3-1))*1 = 28

- SRC YCOUNT = H = 5

- DEST ADDR START = addr0+(M\*y2+x2)\*P = 0+(30\*5+14)\*1 = 164

- DEST XINCR = M\*P = 30\*1 = 30

- DEST XCOUNT = W = 3

- DEST YINCR = -(M*(W-1)+1)*P = -(30*(3-1)+1)*1 = -61

For the clockwise rotation of the rectangular area at 180° from the coordinate (x1, y1) \[(3, 1) in the figure] to the coordinate (x3, y3) \[(19, 7) in the figure], relevant parameters are calculated as follows:

- SRC ADDR START = addr0+(M\*y1+x1)\*P = 0+(30\*1+3)\*1 = 33

- SRC XINCR = P = 1

- SRC XCOUNT = W = 3

- SRC YINCR = (M-(W-1))*P = (30-(3-1))*1 = 28

- SRC YCOUNT = H = 5

- DEST ADDR START = addr0+(M\*y3+x3)\*P = 0+(30\*7+19)\*1 = 229

- DEST XINCR = -P = -1

- DEST XCOUNT = W = 3

- DEST YINCR = -(M-(W-1))*P = -(30-(3-1))*1 = -28

For the clockwise rotation of the rectangular area at 270° from the coordinate (x1, y1) \[(3, 1) in the figure] to the coordinate (x4, y4) \[(24, 2) in the figure], relevant parameters are calculated as follows:

- SRC ADDR START = addr0+(M\*y1+x1)\*P = 0+(30\*1+3)\*1 = 33

- SRC XINCR = P = 1

- SRC XCOUNT = W = 3

- SRC YINCR = (M(W1))\*P = (30(31))\*1 = 28

- SRC YCOUNT = H = 5

- DEST ADDR START = addr0+(M\*y4+x4)\*P = 0+(30\*2+24)\*1 = 84

- DEST XINCR = -M*P = -30*1 = -30

- DEST XCOUNT = W = 3

- DEST YINCR = (M*(W-1)+1)*P = (30*(3-1)+1)*1 = 61

Image Folding
----------
If the memory data is considered as a rectangular image, the image can be folded horizontally or vertically by setting the stride and loop values, as shown below:

.. figure:: ../../picture/DMA2DFold.svg
   :align: center

   Image Folding

As shown in the figure, it is assumed that a pixel width (P) is 8 bit, and the transfer width of 2D DMA is set to 8 bit. In an M × N image (30 × 8 in the figure), a rectangular area of W × H (3 × 5 in the figure) is folded horizontally from the coordinate (x1, y1) \[(3, 1) in the figure] to the coordinate (x2, y2) \[(15, 3) in the figure]. The start address of image data starts from addr0 (0 in the figure). Relevant parameters of 2D DMA are calculated as follows:

- SRC ADDR START = addr0+(M\*y1+x1)\*P = 0+(30\*1+3)\*1 = 33

- SRC XINCR = P = 1

- SRC XCOUNT = W = 3

- SRC YINCR = (M-(W-1))*P = (30-(3-1))*1 = 28

- SRC YCOUNT = H = 5

- DEST ADDR START = addr0+(M\*y2+x2)\*P = 0+(30\*3+15)\*1 = 105

- DEST XINCR = -P = -1

- DEST XCOUNT = W = 3

- DEST YINCR = (M+(W-1))*P = (30+(3-1))*1 = 32

For vertical folding of the rectangular area from the coordinate (x1, y1) \[(3, 1) in the figure] to the coordinate (x3, y3) \[(22, 4) in the figure], relevant parameters are calculated as follows:

- SRC ADDR START = addr0+(M*y1+x1)*P = 0+(30*1+3)*1 = 33
- SRC XINCR = P = 1
- SRC XCOUNT = W = 3
- SRC YINCR = (M-(W-1))*P = (30-(3-1))*1 = 28
- SRC YCOUNT = H = 5
- DEST ADDR START = addr0+(M*y3+x3)*P = 0+(30*4+22)*1 = 142
- DEST XINCR = P = 1
- DEST XCOUNT = W = 3
- DEST YINCR = -(M+(W-1))*P = -(30+(3-1))*1 = -32

Image Filling
----------
If the memory data is considered as a rectangular image, the image can be filled by setting the stride and loop values, as shown below:

.. figure:: ../../picture/DMA2DFill.svg
   :align: center

   Image Filling

As shown in the figure, it is assumed that a pixel width (P) is 8 bit, and the transfer width of 2D DMA is set to 8 bit. In an M × N image (30 × 8 in the figure), a rectangular area of W × H (3 × 5 in the figure) with starting coordinate (x2, y2) \[(19, 2) in the figure] is filled by the value of the coordinate (x1, y1) \[(3, 1) in the figure]. The start address of image data starts from addr0 (0 in the figure). Relevant parameters of 2D DMA are calculated as follows:

- SRC ADDR START = addr0+(M*y1+x1)*P = 0+(30*1+3)*1 = 33
- SRC XINCR = 0
- SRC XCOUNT = W = 3
- SRC YINCR = 0
- SRC YCOUNT = H = 5
- DEST ADDR START = addr0+(M*y2+x2)*P = 0+(30*2+19)*1 = 79
- DEST XINCR = P = 1
- DEST XCOUNT = W = 3
- DEST YINCR = (M-(W-1))*P = (30-(3-1))*1 = 28

Color Key
-------------
When Color Key is enabled, if 2D DMA encounters a value equal to the preset Color Key value during data transfer, it skips the value and the original value is still stored in the skipped address. The width of Color Key can be set to 8/16/24/32-bit. The working principle is shown below:

.. figure:: ../../picture/DMA2DColorKey.svg
   :align: center

   Color Key

As shown in the above figure, the width of Color Key is set to 8 bit and the Key value is set to 0x55. When transferring the F-shaped rectangular area, 2D DMA skips the 0x55 value. So it can be seen from the right side that only the F-shaped rectangular area is transferred and the area that contains the 0x55 value is not transferred. 

.. only:: html

   .. include:: dma2d_register.rst

.. raw:: latex

   \input{../../en/content/dma2d}
