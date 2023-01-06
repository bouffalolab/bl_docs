==========
DMA
==========

简介
=====
DMA(Direct Memory Access)是一种内存存取技术，可以独立地直接读写系统内存，而不需处理器介入处理。
在同等程度的处理器负担下，DMA是一种快速的数据传送方式。
DMA控制器有8组独立专用通道，管理外围设备和内存之间的数据传输以提高总线效率。
主要有四种类型传输包括内存至内存、内存至外设、外设至内存和外设至外设。并支持LLI链接列表功能。
使用上由软件配置传输数据大小、数据源地址和目标地址。

主要特征
=========
- 8组独立专用通道
- 独立控制来源与目标存取宽度(单字节、双字节、四字节)
- 每个通道独立作为读写缓存
- 每个通道可被独立的外设硬件触发或是软件触发
- 支持外设包括UART、I2C、SPI、ADC、I2S、DAC
- 四种流程控制

  * DMA流程控制，来源内存、目标内存
  * DMA流程控制，来源内存、目标外设
  * DMA流程控制，来源外设、目标内存
  * DMA流程控制，来源外设、目标外设

- 支持LLI链表功能，提高DMA效率

功能描述
=========
工作原理
---------
当一个设备试图通过总线直接向另一个设备传输数据(一般是大批量的数据)时，
它会先向CPU发送DMA请求信号。外设通过DMA
向CPU提出接管总线控制权的总线请求，CPU收到该信号后，在当前的总线周期结束后，
会按DMA信号的优先级和提出DMA请求的先后顺序响应DMA信号。
CPU对某个设备接口响应DMA请求时，会让出总线控制权。
于是在DMA控制器的管理下，外设和存储器直接进行数据交换，
而不需CPU干预。数据传送完毕后，设备会向CPU发送DMA结束信号，交还总线控制权。

.. figure:: ../../picture/DmaArch.svg
   :align: center

   DMA框图

DMA 包含一组 AHB Master 接口和一组 AHB Slave 接口。AHB Master 接口根据当前配置需求通过系
统总线主动存取内存或是外设，做为数据搬移的端口。AHB Slave 接口作为配置 DMA 的接口，只支持32-bit 存取。

DMA通道配置
-------------
DMA共支持8路通道，各通道之间互不干涉，可以同时运行，下面是DMA通道x的配置过程：

1. 在DMA_C0SrcAddr寄存器中设置32-bit来源地址

2. 在DMA_C0DstAddr寄存器中设置32-bit目标地址

3. 地址自动累加，可通过配置DMA_C0Control寄存器中SI(来源)、DI(目标)设定是否开启地址自动累加模式，设置为1时，开启地址自动累加模式

4. 设置传输数据宽度，可通过配置DMA_C0Control寄存器中STW(来源)、DTW(目标)位，宽度选项有单字节、双字节、四字节

5. Burst型态，可通过配置DMA_C0Control寄存器中SBS(来源)、DBS(目标)位来设置，配置选项有Single、INCR4、INCR8、INCR16

6. 需要特别注意的是所配置的组合，单笔burst不能超过16字节

7. 设置数据传输长度的范围为：0-4095

外设支持
-------------
可通过配置SrcPeripheral(来源)和DstPeripheral(目标)来决定当前DMA配合的外设，关系为0-5 : UART / 6-9 : I2C / 10-13 : SPI / 18-21 : I2S / 22: ADC / 23: DAC

**UART使用DMA传输数据**

UART发送数据包，使用DMA方式能大量减轻CPU处理的时间，使其CPU资源不被大量浪费，
尤其在UART收发大量数据包（如高频率收发指令）时具有明显优势。

以UART0传输为例，配置过程如下：

1. 将寄存器DMA_C0Config[SRCPH]位的值设置为1，即将Source peripheral设置为UART_TX

2. 将寄存器DMA_C0Config[DSTPH]位的值设置为0，即将Destination peripheral设置为UART_RX

**I2C使用DMA传输数据**

配置如下：

1. 将寄存器DMA_C0Config[SRCPH]位的值设置为7，即将Source peripheral设置为I2C_TX

2. 将寄存器DMA_C0Config[DSTPH]位的值设置为6，即将Destination peripheral设置为I2C_RX

**SPI使用DMA传输数据**

配置如下：

1. 将寄存器DMA_C0Config[SRCPH]位的值设置为11，即将Source peripheral设置为SPI_TX

2. 将寄存器DMA_C0Config[DSTPH]位的值设置为10，即将Destination peripheral设置为SPI_RX

**ADC使用DMA传输数据**

配置如下：

1. 将寄存器DMA_C0Config[SRCPH]位的值设置为22，即将Source peripheral设置为GPADC

**DAC使用DMA传输数据**

配置如下：

1. 将寄存器DMA_C0Config[SRCPH]位的值设置为23，即将Source peripheral设置为GPDAC

链表模式
-----------
DMA支持链表工作模式。在进行一次DMA读或写操作时，可以向下一条链表中填写数据，
当完成当前链表的数据传输后，通过读取DMA_C0LLI寄存器的数值获取下一条链表的起始地址，
直接传输下一条链表中的数据。保证DMA传输过程中连续不间断的工作，提高CPU和DMA的效率。

.. figure:: ../../picture/DMALLI.svg
   :align: center

   LLI 框架

DMA中断
----------

- DMA_INT_TCOMPLETED

   * 数据传输完成中断，当一次数据传输完毕后，会进入此中断

- DMA_INT_ERR

   * 数据传输出错中断，当数据传输过程中出现错误时，会进入此中断


传输模式
==========
内存到内存
------------
这个模式启动后，DMA会根据设定好的搬移数量(TransferSize)，将数据从来源地址搬到目标地址，
传输完毕后DMA控制器会自动回到空闲状态，等待下一次的搬运。

具体配置流程如下：

1. 将寄存器DMA_C0SrcAddr的值设置为来源的内存地址

2. 将寄存器DMA_C0DstAddr的值设置为目标的内存地址

3. 选择传输模式，将寄存器DMA_C0Config[FLOWCTRL]位的值设置为0，即选择memory-to-memory模式

4. 设置DMA_C0Control寄存器中对应的位的数值：DI、SI位设置为1，开启地址自动累加模式，
   DTW、STW位分别设置目标和来源的传输宽度，DBS、SBS位分别设置目标和来源的burst型态，

5. 选择合适的通道，使能DMA，完成数据传输

内存到外设
------------
在这种工作模式下，DMA会根据设定好的搬移数量(TransferSize)，
把数据从来源端搬至内部缓存，当缓存空间不够时自动暂停，
待有足够的缓存空间时继续，直到设定的搬移数量达到。
另外一方面当目标外设请求触发会将目标配置burst到目标地址，
直到达到设定搬移数量完成自动回到空闲状态，等待下一次启动。

具体配置流程如下：

1. 将寄存器DMA_C0SrcAddr的值设置为来源的内存地址

2. 将寄存器DMA_C0DstAddr的值设置为目标的外设地址

3. 选择传输模式，将寄存器DMA_C0Config[FLOWCTRL]位的值设置为1，
   即选择Memory-to-peripheral模式

4. 设置DMA_C0Control寄存器中对应的位的数值：SI位设置为1，开启地址自动累加模式，DI位设置为0，禁用地址自动累加模式，
   DTW、STW位分别设置来源和目标的传输宽度，DBS、SBS位分别设置来源和目标的burst型态，

5. 选择合适的通道，使能DMA，完成数据传输

外设到内存
------------
在这种工作模式下，当来源外设请求触发时将来源配置burst到缓存，
直到设定的搬移数量达到停止。另外一方面，当内部缓存足够一次目标burst数量时，
DMA会自动将缓存的内容搬到目标地址直到达到设定搬移数量完成自动回到空闲状态，
等待下一次启动。

具体配置流程如下：

1. 将寄存器DMA_C0SrcAddr的值设置为来源的外设地址

2. 将寄存器DMA_C0DstAddr的值设置为目标的内存地址

3. 选择传输模式，将寄存器DMA_C0Config[FLOWCTRL]位的值设置为2，
   即选择Peripheral-to-memory模式

4. 设置DMA_C0Control寄存器中对应的位的数值：DI、SI位设置为1，开启地址自动累加模式，
   DTW、STW位分别设置来源和目标的传输宽度，DBS、SBS位分别设置来源和目标的burst型态，

5. 选择合适的通道，使能DMA，完成数据传输

外设到外设
-----------------
在这种工作模式下，当来源外设请求触发时将来源配置burst 到缓存，直到设定的搬移数量达到停止。另外一方面，当内部缓存足够一次目标burst 数量时，DMA 会自动将缓存的内容搬到目标地址直到达到设定搬移数量完成自动回到空闲状态，等待下一次启动。

具体配置流程如下：

1. 将寄存器DMA_C0SrcAddr 的值设置为来源的外设地址

2. 将寄存器DMA_C0DstAddr 的值设置为目标的外设地址

3. 选择传输模式，将寄存器DMA_C0Config[FLOWCTRL] 位的值设置为3，即选择Peripheral-to-Peripheral 模式

4. 设置DMA_C0Control 寄存器中对应的位的数值：DI、SI 位设置为0，禁用地址自动累加模式，STW、DTW 位分别设置来源和目标的传输宽度，SBS、DBS 位分别设置来源和目标的burst 型态

5. 选择合适的通道，使能DMA，完成数据传输

.. only:: html

   .. include:: dma_register.rst

.. raw:: latex

   \input{../../zh_CN/content/dma}
