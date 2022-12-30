===========
AudioDAC
===========

简介
=====
芯片内置一个单声道音频解调模块，可选 GPDAC 模拟输出模式与 PWM 输出模式，输出音频差分模拟信号实现音频播放。

主要特征
===========
+ 集成1路 PWM 调制输出，可输出一对占空比的值差分的PWM调制信号，经过外置RC滤波电路后，形成差分模拟信号。

   + 采样率：8k~48k
   + 信噪比(A-W)：95dB 增益
   + 谐波失真+噪声：-70dB @ 0dB增益

+ 可以将 GPDAC 模块作为音频模拟信号输出，将处理后的音频数据，直接由双通道的 GPDAC 模块转化为模拟差分信号输出。

   + 独立的数字音量控制，并支持斜率可调的渐变音量调节与渐变静音
   + 支持Mixer混合器，支持双声道音源数据输入，并可以选择其中一个单声道输出或混合成一个声道后输出
   + 32位宽度FIFO，深度为32，支持 32bit/24bit/20bit/16bit 四种数据格式，支持单声道数据源与双声道混合数据源
   + 支持DMA传输模式

时钟树
====================
用户需要先将Audio PLL的输出时钟配置到491.52M或451.58M，分别对应48K系列与44.1K系列的采样率，选择Audio PLL输出的5分频作为AudioDAC时钟源。
AudioDAC模块内，会根据采样率设置，自动配置每个子模块的分频系数，无需手动设置分频系数。
时钟分频如下所示：

.. figure:: ../../picture/clockTreePWM.svg
   :align: center

   时钟示意图

功能描述
===========

AudioDAC模块基本框图如图所示。

.. figure:: ../../picture/AudioDAC_Arc.svg
   :align: center

   模块示意图

AudioDAC会将TX FIFO中的数据先经过Mixer混合器处理，再进行数字音量调整、插值滤波等处理后，进行DSM调制，并根据设置的输出模式，
生成对应格式的数据到PWM调制器或者GPDAC模块。GPDAC模块可以直接输出模拟信号，PWM调制器输出的信号需要经过外部RC滤波器电路转化为模拟信号。


AudioDAC中断
-------------
AudioDAC有着丰富的中断控制，包括以下几种中断模式：

- TX FIFO 请求中断

  * 当TX_FIFO_CTRL中TX_DRQ_CNT大于TX_TRG_LEVEL时，产生TX FIFO请求中断。当条件不满足时该中断标志会自动清除。

- TX FIFO undderrun中断

  * 当TX FIFO中并没有数据，但是用户却通过TX_FIFO_CTRL中的TX_CH_EN使能了TX FIFO状态机，则会进入TX FIFO underrun中断。

- TX FIFO overrun中断

  * 当TX FIFO已满，但继续向FIFO中填入数据时，会导致TX FIFO溢出，会产生TX FIFO overrun中断。

- 音量调节完成中断

  * Audio DAC音量控制支持渐入/渐出的功能，当音量调节到设定值后，会触发音量调节完成中断，以通知渐入/渐出调节完成。

FIFO格式控制
--------------
AUPWM_TX_FIFO_CTRL可以控制音频数据储存在FIFO的格式，和设置音频数据源的通道数。
FIFO会根据数据格式与通道数设置，截取其中有效的16bit数据，并作为下一级Mixer混合器的对应通道的输入。

Audio DAC支持如下四种数据存储格式，由FIFO_CTRL[25:24]来决定。

 - Mode 0:

    32bit模式，DATA[15:0] = {FIFO[31:16]}，有效数据的最高位在31 bits

 - Mode 1:

    24bit模式，DATA[15:0] = {FIFO[23:8]}}，有效数据的最高位在23 bits

 - Mode 2:

    20bit模式，DATA[15:0] = {FIFO[19:4]}，有效数据的最高位在19 bits

 - Mode 3:

    16bit模式，DATA[15:0] = {FIFO[15:0]}，有效数据的最高位在15 bits

多数情况下，音频源数据是16bit的宽度时，选择Mode3即可。Audio DAC的最大分辨率也为16bits。
其他模式存在的意义在于，如果待播放的音频数据大小为32bit，其中有效数据宽度为 32/24/20 bits时，用户就无需在软件中将数据转化为16bit模式，
由 FIFO 自动转化，提高效率。

Audio DAC的数据源通道数由 audac_fifo_ctrl 寄存器的 tx_ch_en 段控制，对应的效果如下:

+ 0: 此时左右通道都关闭，FIFO里的数据不会被传输到Mixer混合器，Audio DAC停止播放
+ 1: 单声道模式，此时FIFO里的数据全部被作为左声道数据，会被逐个输入到Mixer混合器的左声道中
+ 2: 单声道模式，此时FIFO里的数据全部被作为右声道数据，会被逐个输入到Mixer混合器的右声道中
+ 3: 双声道模式，此时FIFO里的奇数个数据被作为左声道数据，偶数个数据被作为右声道数据，分别输入到Mixer混合器的左右声道中

因为Audio DAC只支持一路声道输出，所以在多数情况下应当选择单声道音源数据。但在声源数据已经是双声道无法改变时，可以借助Mixer混合器，选择其中一路或者混合播放，
详见后文中对Mixer混合器的介绍。


FIFO的启动与DMA搬运
------------------------
Audio DAC的TX FIFO数据可以通过DMA进行搬运，开启DMA模式需要将 audac_0 寄存器的 dac_itf_en 位DMA接口使能置1，和 audac_fifo_ctrl 寄存器的 tx_drq_en 位DMA请求使能位置1。

通过配置 audac_fifo_ctrl 寄存器的 tx_drq_cnt 段来选择触发DMA请求的FIFO数据量阈值，有4种可选，分别为8/16/32，和与 tx_trg_level 段配置的中断阈值相同。

用户可以通过 audac_fifo_status 寄存器的 txa_cnt 实时获得目前空闲的FIFO数量的数量。

当FIFO内空闲数量的值大于设定阈值，则会触发一次DMA搬运。

注意，如果TX FIFO里剩余的空闲数小于DMA一次搬入的数量， 则触发overrun错误，如果在Audio ADC工作时DMA没有及时搬入数据，也会触发underrun错误，因此要注意FIFO阈值与DMA brust的配置。


可配置的 Mixer 声道混合器
----------------------------
Audio DAC只支持播放一路音频播放通道，如果源数据是双声道的交叉混合数据，则可以使用Mixer混合器，选择其中一路音频播放，或者将两路音频混合后播放。
由寄存器 audac_1 寄存器的 dac_mix_sel 段控制，支持以下4种模式:

    + 仅选择左声道播放(奇数次的数据)
    + 仅选择右声道播放(偶数次的数据)
    + 将左右声道值相加后播放
    + 取左右声道的平均值播放

注意，Mixer混声器的声道配置，必须与FIFO的声道配置相互匹配，否则Audio DAC会停止工作。
具体过程见下图所示。

.. figure:: ../../picture/audacMixer.svg
   :align: center

   Mixer示意图


当配置播放源为立体声的时候，此时DMA顺序搬来的数据会依次填入左右声道中，这要求数据源也是 L-R-L-R 交叉排序方式存储。
此时可以通过Mixer选择器选择需要哪一路声道或者将两个声道求和或取平均进入下一级调制电路。

注意tx_ch_en和Mixer选择。如果通过tc_ch_en将单声道数据放入左声道，但是mixer却是选择右声道，会造成错误无法播放。

音量控制
--------------------------------
用户可以通过audac_s0的dac_s0_volume寄存器配置音量，增益范围为-95.5dB-18dB，步进单位为0.5dB。
当dac_s0_volume_update寄存器为1时，音量才会被更新生效。

用户可以通过dac_s0_mute_softmode，dac_s0_ctrl_mode选择音量调节模式，有3种调节模式:

+ 0: 直接改变，新的音量生效后，会被立即从原来的音量改变到新的音量配置。
+ 1: 过零点渐变模式: 新的音量生效后，会从原来的音量逐渐线性变化到新的音量，并且只会在音频数据为0时变化音量争议。
+ 2: 渐变模式，新的音量生效后，会从原来的音量逐渐线性变化到新的音量。

过零点渐变的斜率由 dac_s0_ctrl_zcd_rate 控制，渐变模式的斜率由 dac_s0_ctrl_rmp_rate 控制，可配置为每2到2048个采样点变化0.5dB，采样点计算方式为2^(n+1)次幂。
当渐变中，音量试图改变但是长时间没有遇到零点数据时，超过 dac_s0_ctrl_zcd_timeout 设定的值时，会强行更新一个步进量0.5dB。

另外还支持在不改变音量时，直接设置静音，同时静音也支持渐变模式，静音的渐变斜率与静音解除的渐变斜率分别为dac_s0_mute_rmpdn_rate与dac_s0_mute_rmpup_rate控制

ZeroDetect
-----------------------------------
AudioDAC 提供ZeroDetect功能，当打开此功能(audac_zd_0寄存器的zd_en位置1)时，如果Mixer混声器输出的数据一直为零，并且数量超过阈值(audac_zd_0寄存器的zd_time段)，
则会关闭DSM调制器，保持输出为0。这样做的目的是减少播零数据与静音时带来的底噪。


配置流程
================================

1. 根据待播的音频采样率，选择对应采样率。
2. 根据实际需求，配置成PWM输出模式或GPDAC输出模式。GPDAC模式时需要额外配置DAC模块，详见DAC说明文档。
3. 根据待播的音频声道数，配置FIFO与Mixer声道混合器。
4. 配置DMA将数据搬运到AudioDAC TX FIFO。
5. 通过audac_fifo_ctrl的tx_ch_en使能通道，开始播放。
6. 在播放的过程中调整音量（可选）。

.. only:: html

   .. include:: audac_register.rst

.. raw:: latex

   \input{../../zh_CN/content/audac}