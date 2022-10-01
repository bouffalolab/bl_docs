===========
SPI
===========

简介
=====
串行外设接口（Serial Peripheral Interface Bus, SPI）是一种用于短程通信的同步串行通信接口规范，装置之间使用全双工模式通信，是一个主机和一个或多个从机的主从模式。
SPI使用4根线完成全双工的通信，这4根信号线分别是：CS（片选）、SCLK（时钟）、MOSI(主机输出从机输入)、MISO(主机输入从机输出)。

主要特征
=========
 - 既可作为SPI主设备，也可作为SPI从设备
 - 主从设备都支持4种时钟格式（CPOL，CPHA）
 - 主从设备都支持1/2/3/4字节传输模式
 - 发送和接收通道各有深度为32个字节的FIFO
 - 自适应的FIFO深度变化特性，适配高性能的场景应用

     + 当Frame为32Bits时，FIFO的深度为8
     + 当Frame为24Bits时，FIFO的深度为8
     + 当Frame为16Bits时，FIFO的深度为16
     + 当Frame为8Bits时，FIFO的深度为32

 - 可配置MSB/LSB传输
 - 可调整字节传输顺序
 - 灵活的时钟配置，最高可支持80M时钟
 - 可配置MSB/LSB优先传输
 - 接收忽略功能，可以设定忽略对指定位置数据的接收
 - 支持从设备模式下的超时机制
 - 支持DMA传输模式

功能描述
===========
时钟控制
-------------
依照不同的时钟相位以及极性设定，SPI时钟共有四种模式，可以通过寄存器spi_config的cr_spi_sclk_pol（CPOL）和cr_spi_sclk_ph（CPHA）进行设置。CPOL用来决定SCK时钟信号空闲时的电平，CPOL=0则空闲电平为低电平，CPOL=1则空闲电平为高电平。CPHA用来决定采样时刻，CPHA=0则在每个周期的第一个时钟沿采样，CPHA=1则在每个周期的第二个时钟沿采样。
通过设置寄存器spi_prd_0和spi_prd_1，还可以调整时钟的开始和结束电平持续时间、相位0/1的时间以及每帧数据之间的间隔。四种模式下的具体设置如下图所示：

.. figure:: ../../picture/SPIClock.svg
   :align: center

   SPI时序图

其中各数字含义如下：

 - 1是起始条件的长度，通过配置寄存器spi_prd_0中cr_spi_prd_s。
 - 2是停止条件的长度，通过配置寄存器spi_prd_0中cr_spi_prd_p。
 - 3是相位0的长度，通过配置寄存器spi_prd_0中cr_spi_prd_d_ph_0。
 - 4是相位1的长度，通过配置寄存器spi_prd_0中cr_spi_prd_d_ph_1。
 - 5是每帧数据之间的间隔，通过配置寄存器spi_prd_1中cr_spi_prd_i。


主设备持续传输模式
-------------------
开启该模式后，在发送完当前数据而FIFO里还存在可用数据时，CS信号不会被释放。

主从设备传输接收数据
--------------------------------------
主从设备传输接收数据时应设置相同的framesize,可通过设置寄存器spi_config中cr_spi_frame_size。如果主设备和从设备的约定以32bits的framesize进行通信时，在某一帧数据中，主设备的clk由于异常不满足32时，会出现下列现象。

 - 主设备发送的数据无法被送到从设备的RX FIFO中。从设备无法接收到数据。
 - 从设备发送的数据时，会跳过这一帧数据，等下次主设备clk正常时，继续发送下一帧数据。

接收忽略功能
-------------
通过设置需要过滤掉的开始位和结束位，SPI会将接收到数据中的对应数据段丢弃。如下图所示：

.. figure:: ../../picture/SPIIgnore.svg
   :align: center

   SPI Ignore波形图

通过配置寄存器spi_config中cr_spi_rxd_ignr_en开启忽略功能。通过配置寄存器spi_rxd_ignr中cr_spi_rxd_ignr_s设置忽略功能的起始位。通过配置寄存器spi_rxd_ignr中cr_spi_rxd_ignr_p设置忽略功能的结束位。

上图中过滤的开始位设为0，结束位设为7则Dummy Byte会被收到，结束位设为15则Dummy Byte会被丢弃。

滤波功能
----------------
通过使能该功能和设置门限值，SPI会将小于或等于门限值宽度的数据过滤。
通过配置寄存器spi_config中cr_spi_deg_en使能该功能和配置cr_spi_deg_cnt设置门限值，SPI会将达不到门限值宽度的数据过滤掉。数据宽度小于cr_spi_deg_cnt+1；
如下图所示，若想滤去数据宽度小于4的数据，需要将cr_urx_deg_cnt的值设置为4。input为初始数据,output为滤波后的数据。

滤波逻辑过程：

 - tgl为input和output的异或结果。
 - deg_cnt从0开始计数，计数条件为tgl为高电平，并且reached为低电平。
 - 若deg_cnt计数值达到cr_urx_deg_cnt设置的值时,reached为高电平。
 - 当reached为高电平时，将input输出到output。
 - 注释:deg_cnt自加的条件：tgl为高电平且reached为低电平，其余情况下deg_cnt会被清0。

.. figure:: ../../picture/SPIDeg.svg
   :align: center

   SPI滤波波形图

可配置MSB/LSB传输
-----------------
可配置MSB/LSB传输方式仅限于1个byte中的8个bits的优先传输顺序，通过配置寄存器spi_config中cr_spi_bit_inv位设置字节内bit的传输顺序。0表示MSB，1表示LSB。
以frame size等于24 bits传输数据为例，数据格式为：Data[23:0]=0x123456。
当设置为MSB传输时，传输的顺序为：01010110(二进制，第1个字节：0x56);00110100(二进制，第2个字节：0x34);00010010(二进制，第3个字节：0x12)；
当设置为LSB传输时，传输的顺序为：01101010(二进制，第1个字节：0x56);00101100(二进制，第2个字节：0x34);01001000(二进制，第3个字节：0x12)。

可调整字节传输顺序
-----------------
可调整字节传输顺序方式仅限于FIFO中不同的byte间的优先传输顺序。通过配置寄存器spi_config中cr_spi_byte_inv位设置FIFO内byte的传输顺序。0表示优先发送低字节，1表示优先发送高字节。
同样以frame size等于24 bits传输数据为例，数据格式为：Data[23:0]=0x123456。
当设置优先发送低字节，传输的顺序为：0x56(第1个字节：低字节)；0x34(第2个字节：中间字节)；0x12(第3个字节：高字节)；
当设置优先发送高字节，传输的顺序为：0x12(第3个字节：高字节)；0x34(第2个字节：中间字节)；0x56(第1个字节：低字节)；

可调整字节传输可以和可配置MSB/LSB传输配合使用。

从模式超时机制
--------------------------------
通过设定一个超时门限，当从模式下SPI超过该时间值未收到时钟信号时，会触发中断。

I/O传输模式
-------------
芯片通信处理器可以响应来自FIFO的中断来执行FIFO填充和清空操作。每个FIFO都有一个可编程的FIFO触发阈值来触发中断。当寄存器spi_fifo_config_1中rx_fifo_cnt大于rx_fifo_th触发阈值时，将产生一个中断，向芯片通信处理器发送信号来清空RX FIFO。当寄存器spi_fifo_config_1中rx_fifo_cnt大于rx_fifo_th时，将产生中断，向芯片通信处理器发送信号来重新填充TX FIFO。
可以通过查询SPI状态寄存器来确定FIFO中的采样值以及FIFO的状态。软件负责确保正确的RX FIFO触发阈值和TX FIFO触发阈值，防止接收FIFO overflow和发送FIFO underflow。

DMA传输模式
-------------
SPI支持DMA传输模式。使用该模式需要分别设置TX和RX FIFO的阈值，将寄存器spi_fifo_config_0中spi_dma_tx_en置1，则开启DMA发送模式。将寄存器spi_fifo_config_0中spi_dma_rx_en置1,则开启DMA接收模式。当该模式启用后，UART会对TX/RX FIFO进行检查，一旦寄存器spi_fifo_config_1中tx_fifo_cnt/rx_fifo_cnt大于tx_fifo_th/rx_fifo_th，将会发起DMA请求，DMA会按照设定将数据搬移至TX FIFO中或从RX FIFO中移出。

SPI中断
-------------
SPI有着丰富的中断控制，包括以下几种中断模式：

 - SPI传输结束中断
 - TX FIFO请求中断
 - RX FIFO请求中断
 - 从模式传输超时中断
 - 从模式TX过载中断
 - TX/RX FIFO溢出中断

在主模式下，SPI传输结束中断会在每帧数据传输结束时触发；在从模式下，SPI传输结束中断会在CS信号被释放时触发。
TX/RX FIFO请求中断会在其FIFO可用计数值大于其设定的阈值时触发，当条件不满足时该中断标志会自动清除。
从模式传输超时中断会在从模式下超过超时门限值未收到时钟信号时触发。
如果TX/RX FIFO发生了上溢或者下溢，会触发TX/RX FIFO溢出中断，当FIFO清除寄存器spi_fifo_config_0中位tx_fifo_clr/rx_fifo_clr被置1时，对应的FIFO会被清空，同时溢出中断标志会自动清除。
可以通过寄存器SPI_INT_STS查询各中断状态和对相应的位写1清除中断。

.. only:: html

   .. include:: spi_register.rst

.. raw:: latex

   \input{../../zh_CN/content/spi}