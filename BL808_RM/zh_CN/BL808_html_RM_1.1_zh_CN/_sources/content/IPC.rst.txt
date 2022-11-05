===========
IPC
===========

简介
=====
IPC(inter-processor communication)是多核处理器之间通信的一种机制。
BL808 有三个异构内核，任意两个内核之间都可以互相通过IPC实现通信。

主要特征
=========
 - 每个内核都有独立的32位IPC通道
 - 任意两核之间都可以互相通信

功能描述
===========
基本原理
-----------
每个内核都有一组IPC的寄存器，包括IPCx_TRI、IPCx_STS、IPCx_ACK、IPCx_IEN、IPCx_IDIS、IPCx_ISTS共6个寄存器，
这些寄存器的长度都是32bits，每个bit都对应IPC的一个通道。核M0、LP、D0分别对应IPC0、IPC1、IPC2。当一个核需要向另一个核发通知时，只需要向接收核的IPCx_TRI的对应通道写1即可，此时接收核的IPCx_STS的对应通道即被设置为1，如果接收核的IPC对应通道的中断也被使能，则会收到一个中断，此时即获知了其他核发来的通知。

操作流程
---------
以核x向核y发通知，所使用的通道为z，其操作流程如下：

 - 核y使能通道z的中断，即核y执行操作IPCy_IEN = 1<<z
 - 核x向IPCy_TRI寄存器的第z位写入1，即核x执行操作IPCy_TRI = 1<<z
 - 核y收到中断，向核x做出应答，即核y执行操作IPCy_ACK = 1<<z
 - 核y执行通知的应用层相关操作
 
工作流程图如下所示：

.. figure:: ../../picture/IPCFlow.svg
   :align: center

   IPC 流程图

