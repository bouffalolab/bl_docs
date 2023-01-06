==========
QDEC
==========

简介
=====
正交解码器（quadrature decoder），用于将双路旋转编码器产生的两组相位相差90度的脉冲解码为对应转速和旋转方向。

主要特征
=========
- 三组QDEC可供使用

- QDEC的时钟源可以为32K（f32k_clk）或32M（xclk），正常工作时建议选择32M作为时钟源，当进入睡眠模式并希望可以由QDEC唤醒时建议使用32k

- 支持5位分频值，可分频1~32

- 16位脉冲计数范围（-32768~32767 pulse/sample）

- 12种可配置的sample周期（32us~131ms per sample at 1MHz）

- 16位可设置的report周期（0~65535 sample/report）

- 内置一个可随采样进行闪烁的LED功能（LED on/off 0~511 us/sample）

- 中断可配（sample中断、report中断、error中断、overflow中断）

- 可配置为PDS的唤醒源（clock source需要配置为32k）

功能描述
==========
QDEC预期的工作频率为1MHz，检测的速度越快所需的工作频率越高。
每次采样会将编码器输出的A/B两相脉冲解码为高低电平，对比前次采样结果可获得当前编码器旋转方向与脉冲计数变化情况（顺时针转+1，逆时针转-1，不转不变，出错报错并计数），经过report所设定的采样次数后，即可获得此段时间内编码器的旋转方向和脉冲计数，据此求解出此段report时间内转速方向均值。
每次采样的周期可配，在工作频率为1MHz时，最低32us一次采样，最高131ms一次采样。
可配置中断为单次采样结束触发（sample中断）、多次采样结束触发（report中断），以灵活测量转速。
可配置的LED闪烁功能，闪烁频率=LED周期/采样周期，每次闪烁中的on/off由LED极性决定。

.. figure:: ../../picture/qdec.svg
   :align: center

   QDEC功能框图

.. only:: html

   .. include:: qdec_register.rst

.. raw:: latex

   \input{../../zh_CN/content/qdec}