==========
DAC
==========

简介
=====
芯片内置一个 10bits 的数字模拟转换器(DAC）,FIFO 深度为 1，支持 2 路 DAC 调制输出。
可用于音频播放，变送器电压调制。

主要特点
=========
    + DAC 调制精度为 10-bits
    + DAC 的输入时钟可选为 32k、16k、8k 或 512k
    + 支持 DMA 将内存搬运至 DAC 调制寄存器
    + 支持双声道播放 DMA 搬运模式
    + DAC 的输出引脚固定为 ChannelA 为 GPIO11,ChannelB 为 GPIO4

功能描述
==========
DAC模块基本框图如图所示。

.. figure:: ../../picture/gpdac.svg
   :align: center

   DAC基本框图

- DAC模块支持最多两路调制输出

- DAC模块支持双声道DMA数据搬运模式

- DAC模块支持长度为32-bit的DMA数据接口，其中高16位将会调制在ChannelA的引脚上，低16位调制在ChannelB引脚。

.. only:: html

   .. include:: dac_register.rst

.. raw:: latex

   \input{../../zh_CN/content/dac}

