=====
概述
=====
BL808 是高度集成的 AIoT 芯片组，
具有 Wi-Fi/BT/BLE/Zigbee等无线互联单元，
包含多个CPU以及音频编码译码器
、视频编码译码器和AI 硬件加速器
，适用于各种高性能和低功耗应用领域。


BL808 系列芯片主要包含无线和多媒体两个子系统。

无线子系统包含一颗 RISC-V 32-bit 高性能CPU，集成 Wi-Fi/BT/Zigbee 无线子系统，
可以实现多种无线连接和数据传输，提供多样化的连接与传输体验。

多媒体子系统包含一颗 RISC-V 64-bit 超高性能CPU，
集成DVP/CSI/
H264/NPU等视频处理模块，
可以广泛应用于视频监控/智能音箱等多种AI领域。

多媒体子系统组成部分如下：

- NPU HW NN 协处理器(BLAI-100)，适用于人工智能应用领域
- 摄像头接口
- 音频编码译码器
- 视频编码解码器
- 传感器
- 显示接口

电源管理单元控制低功耗模式。 此外，还支持各种安全功能。

外围接口包括 USB2.0、Ethernet、SD/MMC、SPI、UART、I2C、I2S、PWM、GPDAC/GPADC、ACOMP、PIR、Touch、IR remote、Display和 GPIO。

支持灵活的 GPIO 配置，BL808 最多可达 40 个 GPIO。

.. figure:: ../../picture/Functionalblockdiagram.svg
   :align: center

   功能框图

