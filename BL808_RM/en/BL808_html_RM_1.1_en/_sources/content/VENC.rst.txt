===============
VENC
===============

Overview
=====
VENC follows the H264 video coding standard, which mainly compresses objects by prediction and motion compensation, and improves image quality by a loop filter, while meeting the requirements of stream transmission and image quality.

Features
===========
- 1920x1080p@30fps + 640x480@30fps, BP/MP
- Input: Semi-Planar YCbCr 4:2:0
- Output: NALU(Network Abstract Layer Uint) in byte stream format
- CBR/VBR mode
- Up to 8 ROI
- Up to 16 OSD coding areas
- Supports software and linked modes
- Dynamically configurable max/min quantization parameters
- Dynamically configurable I/P-frame target bit
- Dynamically configurable I-frame distance

Functional Description
============
Software Mode and Linked Mode
----------
Software Mode: The hardware interacts with the software in the unit of frame. The software sets that the hardware compresses one frame only after one frame is started, and a frame interrupt is returned after compression is completed. The input frame memory shall contain at least one frame.

Linked Mode: The software sets the \<CFG\_S\_ENC\_SEQ\_EN> or \<CFG\_ENC\_SEQ\_EN> bit in H264\_ENCODER\_CTRL to enable sequence encoding. The hardware automatically detects the state of the input frame and compresses it. Until software pulls down Sequence Enable, the hardware stops compression (ending at a complete frame) and the sequence interrupt is returned. When each frame in the sequence is finished, a frame interrupt is returned.

CBR/VBR Mode
----------
If \<NUM\_IMB\_BITS> or \<S\_NUM\_IMB\_BITS> in CORE\_REG23 is set to 0, the stream is in VBR Mode. If it is not 0, it is in CBR Mode. In VBR Mode, I/P-frame fixed quantization parameters (CORE\_REG3/CORE\_REG5) shall be set. It is not recommended to switch modes when hardware is working.

Dual-Stream
----------
The frame enable, frame interrupt, sequence enable, and sequence interrupt of the two streams are configured separately. Essentially, the frame rate of two streams shall be the same, but the resolution and other compression parameters can be different. 

Region of Interest (ROI)
----------
A single stream supports up to 8 ROI areas, and each area has its own enabling and start/end macroblock position settings. It is necessary to set the \<CFG\_ROI\_UPD> bit in H264\_ROI\_MODE or the \<CFG\_S\_ROI\_UPD> bit in H264\_S\_ROI\_MODE to "1", to make the new settings take effect. The hardware synchronizes ROI settings for each frame.

OSD Coding Area
-------
The hardware supports up to 16 OSD areas, and H264\_OSD\_EN decides the OSD area of each stream as configured in the software. The OSD area is represented by the start/end macroblock position.

Dynamically Configurable Max/Min Quantization Parameters
-------
To provide a better control stream, after the software sets CORE\_REG31, it is necessary to set H264\_ENCODER\_CTRL (the \<CFG\_QR\_UPD> and \<CFG\_S\_QR\_UPD> bits set two streams respectively) to make the new setting value take effect. The hardware will synchronize the new setting value in the unit of GOP to adjust the quantization parameters dynamically.

Dynamically Configurable I/P-Frame Target Bit
-------
The software can set CORE\_REG23/CORE\_REG24 to adjust the target bit of I/P-frame. After CORE\_REG23/CORE\_REG24 is set, it is necessary to set H264\_ENCODER\_CTRL (the \<CFG\_QR\_UPD> and \<CFG\_S\_QR\_UPD> bits set two streams respectively) to make the new setting value take effect. The hardware will synchronize the new setting value in the unit of GOP (CBR Mode).

Dynamically Configurable I-Frame Distance
-------
After the software sets CORE\_REG8, it is necessary to set H264\_ENCODER\_CTRL (the \<CFG\_QR\_UPD> and \<CFG\_S\_QR\_UPD> bits set two streams respectively) to make the new setting value take effect. The hardware will synchronize the new setting value in the unit of GOP (CBR/VBR available).

Video Sequence Interrupt
----------
In Linked Mode, when software pulls down Sequence Enable, the hardware generates sequence interrupt after processing the current frame (\<S\_SEQ\_DONE\_INT> or \<SEQ\_DONE\_INT> bit in VDO\_INT), and this interrupt requires the software to set the interrupt clear (\<S\_SEQ\_DONE\_INT\_CLR> or \<SEQ\_DONE\_INT\_CLR> bit in VDO\_INT\_CLR). This interrupt can be ignored by masking (\<S\_SEQ\_DONE\_INT\_MASK> or \<SEQ\_DONE\_INT\_MASK> bit in VDO\_INT\_MASK).

Frame Interrupt
----------
In Software Mode and Linked Mode, the frame interrupt (\<S\_FRM\_DONE\_INT> or \<FRM\_DONE\_INT> in VDO\_INT) will be generated as soon as the hardware compresses one frame. This interrupt requires the software to set the interrupt clear (\<S\_FRM\_DONE\_INT\_CLR> or \<FRM\_DONE\_INT\_CLR> bit in VDO\_INT\_CLR). This interrupt can be ignored by masking (\<S\_FRM\_DONE\_INT\_MASK> or \<FRM\_DONE\_INT\_MASK> bit in VDO\_INT\_MASK).

Input Cache Overflow Alert
----------
If the video compression speed is much lower than the camera pixel, buffer overflow will occur at output. The software can detect that through the status register VDO\_SRC\_R\_DBG (\<SRC\_WR\_OV\_RD> bit) or VDO\_S\_SRC\_R\_DBG (\<S\_SRC\_WR\_OV\_RD> bit).
