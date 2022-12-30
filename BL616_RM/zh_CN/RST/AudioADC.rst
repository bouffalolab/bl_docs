===========
AudioADC
===========

简介
=====
芯片内置一个AudioADC模块，可实现单路录音功能，集成一路16bit高精度ADC用于模拟音频信号采集，此外支持PDM数字接口。

主要特征
===========
+  集成1个 Sigma-Delta 高性能ADC,可支持差分或单端的模拟mic信号输入,复用GPIO模拟输入

   + 采样率：8k~96k
   + 信噪比(A-W)：90dB @ 6dB增益
   + 谐波失真+噪声：-80dB @ 6dB增益
   + 模拟前置增益： 6~42 dB, 以3dB步进
   + ADC的输入阻抗等模拟参数可配置

+ 额外支持PDM数字接口，可用于数字mic录音，复用GPIO输入
+ 除音频录制模式外，额外支持直流测量模式，将AudioADC作为高精度ADC使用，支持差分与单端模式，分辨率可达到16bit
+ 可调节的高通滤波器和独立的数字音量控制
+ 32位宽度FIFO 深度为32，数据支持 32bit/24bit/20bit/16bit 四种存储格式
+ 支持DMA传输模式

时钟树
====================

用户需要先将Audio PLL的输出时钟配置到491.52M或451.58M，分别对应48K系列与44.1K系列的采样率，选择Audio PLL输出的5分频作为AudioDAC时钟源，
AudioADC模块内，会根据工作模式与采样率设置，自动配置每个子模块的分频系数，无需手动设置分频系数。
此寄存器数值与分频比如下所示

.. figure:: ../../picture/clockTreeADC.svg
   :align: center

   时钟示意图

功能描述
===========

AudioADC模块基本框图如图所示。

.. figure:: ../../picture/AudioADC_Arc.svg
   :align: center

   模块框图

AUADC模块输入既支持音频模拟信号，也支持PDM数字信号。
当选择为PDM数字信号的时候，输入数据在经过PDM电路处理后，进入音频处理通道中。
当选择为数字信号时，模拟信号会先经过PGA放大，再结果ADC量化处理后，进入音频处理通道中。
音量处理通道会对初始数据使用SINC函数低通滤波和重采样，然后进行高通滤波与音量增益控制的处理，最后将数据推入FIFO中。

音源通道选择
----------------------------
AudioADC 模块支持模拟mic输入与PDM数字mic接口，其中PDM时序规范中，在时钟上升沿与下降沿的数据分别为左声道与右声道。
当使用模拟mic输入时，需要配置内置ADC并选择好要  在所集成的ADC的配置完成后，通过pdm_dac_0寄存器的adc_0_src位选择音源为ADC即可。
当使用PDM数字mic时，需要通过pdm_dac_0中的pdm_0_en启动PDM，再通过pdm_pdm_0中的adc_0_pdm_sel选择左声道或右声道。

AUADC中断
-------------
AUADC模块有着丰富的中断控制，包括以下几种中断模式：

 - RX FIFO 阈值请求中断
 - RX FIFO undderrun中断
 - RX FIFO overrun中断

当 FIFO 中的有效的数据量大于audadc_rx_fifo_ctrl中的rx_trg_level时, 产生RX FIFO请求中断。当条件不满足时该中断标志会自动清除。
FIFO 中有效数据量，可以在 audadc_rx_fifo_status 寄存器的 rxa_cnt 段查询。

当RX FIFO中并没有数据，但CPU或DMA读取了FIFO，则会产生 RX FIFO underrun 中断。

启动 AudioADC 后，如果 FIFO 中的数据没有及时读出，数据累积超过最大深度，会导致 RX FIFO 溢出，从而产生 RX FIFO overrun 中断。

FIFO格式控制
--------------
audadc_rx_fifo_ctrl寄存器中的rx_data_mode可以控制音频数据储存在FIFO中的格式。

FIFO控制器支持如下四种数据存储格式:

 - Mode 0:

    DATA[31:0] = {FIFO[15:0],16'h0}

 - Mode 1:

    DATA[31:0] = {8{FIFO[15]},FIFO[15:0],8'h0}

 - Mode 2:

    DATA[31:0] = {12{FIFO[15]},FIFO[15:0],4'h0}

 - Mode 3:

    DATA[31:0] = {16{FIFO[15]},FIFO[15:0]}


最高有效位的分布

 - Mode 0:

    有效数据的最高位在31 bits

 - Mode 1:

    有效数据的最高位在23 bits

 - Mode 2:

    有效数据的最高位在19 bits

 - Mode 3:

    有效数据的最高位在15 bits

如果对储存格式没有特殊要求，一般来说选择Mode3，因为ADC的分辨率为16bit，使用16bit宽度去储存音频时内存的占用最低。
其他格式会将有效的16bit数据，左移到32bits中的不同位置，低位使用0去补齐，高位使用符号位进行补齐，用以满足一些特殊的存储要求。


测量模式
------------------------
AudioADC 中的高性能ADC，除了可以音频模拟信号进行采样外，还支持作为高精度ADC使用，并且自带了增益可调节的PGA放大器，可用于单端与差分的微弱信号采集。
测量模式需要在 audpdm_top 寄存器的 adc_rate 段中选择 测量模式，在 audadc_cmd 寄存器中，audadc_meas_filter_en 位置1使能测量模式，
在 audadc_pga_mode 段与 audadc_pga_gain 段中选择测量的模拟通道，在 audadc_pga_mode 段中配置ADC模式为直流差分或者直流单端模式，此时音频处理通道将会被旁路。
测量模式下的FIFO格式等其他配置，与音频模式相同。


FIFO的启动与DMA搬运
------------------------
AudioADC 的 FIFO 数据可以通过 DMA 进行搬运。

用户可以通过PDM_RX_FIFO_STATUS寄存器实时获得目前FIFO有效数据的数量。

通过配置 audadc_rx_fifo_ctrl 中的 rx_drq_cnt 选择触发 DMA request 的 FIFO 阈值，有4种选择，8、16、32，或者与 rx_trg_level 配置的 FIFO 中断阈值相同。

当 FIFO count 的值大于设定阈值，并且 AUDADC_RX_FIFO_CTRL[4] DMA 模式被使能后，则会向 DMA 发起请求。

注意，启动 AudioADC 后，如果 FIFO 中的数据没有及时读出，当 FIFO 溢出后会触发 overrun 错误，同时会造成数据丢失，需要注意配置顺序。

配置流程
================================
1. 选择录制音频采样率
2. 根据录制的数据源是否是PDM数字信号或者是模拟信号配置pdm_dac_0的adc_0_src寄存器
3. 如果是pdm格式，通过pdm_pdm_0的adc_0_pdm_sel选择pdm的声道
4. 配置DMA将Audio的RX FIFO数据实时搬运到指定区域
5. 通过audadc_rx_fifo_ctrl的rx_ch_en打开状态机开始录音
6. 在录制的过程中调整音量（可选）

.. only:: html

   .. include:: ausolo_register.rst

.. raw:: latex

   \input{../../zh_CN/content/ausolo}