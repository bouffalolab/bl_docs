===========
USB
===========

简介
=====
USB(Universal Serial Bus)通用串行总线，是一个外部总线标准，用于规范电脑与外部设备的连接和通讯。BL808 支持USB2.0（HighSpeed + FullSpeed），可作为主机控制器(Host)或者外围控制器(Device)。
作为主机控制器时，它包含一个支持所有速度事务的 USB 主机控制器。在没有软件干预的情况下，主机控制器可以自行处理基于事务的数据结构以减轻CPU的负载并自动在 USB 总线上发送和接收数据。 当作为外围控制器时，除端点 0 外的每个端点都支持USB规范的传输类型，以适配各种应用类型。此外，BL808 的USB控制器符合 OTG 标准，支持会话请求协议(SRP)和主机协商协议(HNP)。

主要特征
=========
 - 兼容USB2.0（HighSpeed + FullSpeed）
 - 兼容OTG revision1.3
 - 支持UTMI+ Level 3
 - 支持OTG SRP和HNP协议
 - 支持Host、OTG、Device模式
 - 支持LPM
 - 兼容EHCI数据结构（不支持FSTN和SITD）
 - 支持9个端点（EP0：control endpoint    EP1-EP8：interrupt/isolation/bulk endpoint）
 - 支持1个控制传输专用FIFO和4个一般FIFO（控制FIFO：64字节    一般FIFO：512字节）
 - 支持双FIFO、三FIFO模式工作
 - 支持DMA
 - 支持VDMA

功能描述
===========
.. USB框架描述
.. -------------------------------

.. .. figure:: ../../picture/USB_block_diagram.svg
..    :align: center
.. 
.. .. figure:: ../../picture/USB_host_schedule.svg
..    :align: center



USB使用步骤
--------------

1.配置内部transceiver

2.配置usb controller

3.配置usb中断

4.配置usb dma/vdma（不支持cpu直接读写fifo）

5.完成


部分寄存器配置及功能描述
-------------------------------


USB枚举阶段中断处理流程
--------------------------


