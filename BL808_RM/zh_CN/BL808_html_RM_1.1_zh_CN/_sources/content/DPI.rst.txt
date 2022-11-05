===========
DPI
===========

简介
=====
DPI是显示像素接口(Display Pixel Interface)的简称，也被称为RGB接口，是MIPI联盟定义的用于与显示屏通信的接口协议。
本模块拥有16位并行数据线，由系统内存充当显存，通过持续输出数据和同步信号，直接驱动显示屏进行显示。

主要特点
===========

 - 符合 MIPI® 联盟标准
 - 支持单缓冲区和乒乓缓冲区作为显存
 - 乒乓操作支持硬件控制和软件控制
 - 支持测试模式
 - 可编程定时参数
 - 输入像素格式(缓冲区数据格式)支持：

     + YUV422(交织)
     + RGB888
     + RGB565
     + RGBA8888
     + YUV420(平面)

 - 输出像素格式支持RGB565
 - 一个可编程中断
 - 可与OSD配合使用

功能描述
===========

DPI模块基本框图如图所示。

.. figure:: ../../picture/DPIBlock.svg
   :align: center

   DPI基本框图

DPI时序
-------------
16位DPI接口时序如下图所示：

.. figure:: ../../picture/DPITiming.svg
   :align: center

   DPI时序图

其中：

 - VSYNC(Vertical Sync)：垂直同步定时信号，用于表示每帧显示图像的开始；
 - HSYNC(Horizontal Sync)：水平同步定时信号，用于表示每行水平像素的开始；
 - PCLK(Pixel Clock)：像素时钟，持续不断地产生，VSYNC、HSYNC、DE和DB[15:0]会在PCLK的上升沿被采样；
 - DE(Data Enable)：数据使能信号，用于指示有效像素数据；
 - DB[15:0](Data Bit)：并行数据位，总共有16位，用于传输像素数据。

可编程定时参数
------------------
DPI的定时参数如下图所示：

.. figure:: ../../picture/DPIParameter.svg
   :align: center

   定时参数图

垂直周期(一帧)等于Vsync、VBP、VACT与VFP之和；水平周期(一行)等于Hsync、HBP、HACT与HFP之和。
定时参数的可编程范围如下表所示：

.. figure:: ../../picture/DPITable.svg
   :align: center

   定时参数取值范围表

单缓冲区
-------------
DPI模块可以使用一块大小为显示分辨率*每个像素所占字节数的内存作为显存，当开始工作时，DPI会重复不断地将这块内存中的数据传输给显示模块。
如果对这块内存中的数据进行修改，则修改内容可能在当前帧生效，也可能在下一帧生效，这取决于修改的数据在内存中的位置位于DPI当前传输的数据在内存中的位置之后还是之前。

乒乓缓冲区
-------------
DPI模块可以使用两块大小分别为显示分辨率*每个像素所占字节数的内存作为显存，并重复不断地将其中一块内存中的数据传输给显示模块。将当前传输的内存从一块切换到另一块的控制方式可以设置为硬件控制或软件控制。
当设置为硬件控制时，DPI的数据由Cam模块提供(参阅Cam模块介绍)，DPI会在Cam模块将一帧数据完整写入没有参与传输的那块内存，并且当前正在传输的内存块中的数据传输完成之后将使用的内存块切换为另一块。
当设置为软件控制时，DPI当前用于传输的内存块由<SWAP_IDX_SWV>进行设置，写0和写1分别对应两个内存块。

测试模式
-------------
测试模式下，DPI可以输出同一行像素颜色相同、不同行之间颜色渐变的图像。测试模式需要设置3个参数：像素数据最小值VAL_MIN(16-bit)、像素数据最大值VAL_MAX(16-bit)和步长STEP(8-bit)。
DPI会以VAL_MIN作为第1行每个像素的值，(VAL_MIN+STEP)作为第2行每个像素的值，(VAL_MIN+STEP*2)作为第3行每个像素的值...以此类推，直到第N行(VAL_MIN+STEP*(N-1))大于VAL_MAX时回到VAL_MIN，循环往复直到输出的行数达到设置值。
以VAL_MIN=0x00E0，VAL_MAX=0x012F，STEP=0x10为例，测试模式下显示屏的显示如下图所示：

.. figure:: ../../picture/DPITest.svg
   :align: center

   测试模式

输入像素格式
---------------
DPI支持多种像素格式输入，内部会将各种像素格式统一转成RGB888格式，再将RGB888格式转成RGB565格式发送出去。

YUV422(交织)
**************
YUV422(交织)格式输入数据的处理过程如下图所示：

.. figure:: ../../picture/DPIYUV422.svg
   :align: center

   YUV422处理过程

RGB888
*********
RGB888格式输入数据的处理过程如下图所示：

.. figure:: ../../picture/DPIRGB888.svg
   :align: center

   RGB888处理过程

RGB565
*********
RGB565格式输入数据的处理过程如下图所示：

.. figure:: ../../picture/DPIRGB565.svg
   :align: center

   RGB565处理过程

RGBA8888
***********
RGBA8888格式输入数据的处理过程如下图所示：

.. figure:: ../../picture/DPIRGBA8888.svg
   :align: center

   RGBA8888处理过程

YUV420(平面)
**************
YUV420(平面)格式输入数据的处理过程如下图所示：

.. figure:: ../../picture/DPIYUV420.svg
   :align: center

   YUV420处理过程

可编程中断
-------------
DPI拥有一个可编程中断，通过配置，中断的触发源可以选择为DPI的输入Vsync或者输出Vsync，跳变沿可以选择为上升沿或者下降沿。
例如将中断配置为输入Vsync上升沿触发，则DPI会在其输入模块的每个Vsync上升沿产生一次中断请求；将中断配置为输出Vsync下降沿触发，则DPI会在其输出的每个Vsync下降沿产生一次中断请求。

与OSD配合使用
----------------
DSI的数据输入可以配置为先经过OSD处理，OSD的功能介绍请参阅OSD模块。