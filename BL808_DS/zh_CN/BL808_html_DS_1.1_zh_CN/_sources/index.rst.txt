
Features
============
.. raw:: latex

   \section*{Features}
   \begin{multicols}{2}

- 无线（业内顶尖射频性能）
  
  * 2.4GHz 射频收发器
  * Wi-Fi 802.11 b/g/n
  * Bluetooth® 5.x Dual-mode (BT+BLE)
  * Zigbee / IEEE 802.15.4
  * Wi-Fi/蓝牙/zigbee 共存
  * 集成 balun，PA/LNA
  * 支持外部PA/LNA

- 微控制器子系统
  
  * 多核 RISC-V CPUs (Max Freq 480MHz)
  * RTC 定时器最长计数周期为 1 年
  * 通用定时器
  * DMA 通道
  * JTAG 开发支持
  * XIP QSPI 闪存支持

- 音频编码译码器

  * ADC*2 (MIC*2 or MIC*1+Line-in)
  * DAC*1 (Speaker)
  * 采样率8~192 KHz，24bit



- Video/Image/Display

  * MJPEG , H264 (Baseline/Main)
  * 最大分辨率：2M(1920x1080)
  * 视频编码格式:

     + MJPEG and H264 (Baseline/Main)
     + 1920x1080 @ 30fps + 640x480 @ 30fps
     + 高达 8-ROI(region-of-interest)
  * Camera Sensor接口：DVP和MIPI-CSI
  * 显示接口：SPI、DBI、DPI(RGB)、MIPI-DSI


- AI NN 通用硬件加速器

  * NPU BLAI-100 (BLAI engine) 用于视频/音频检测/识别

- Memory

  * 内嵌 32/64MB DRAM
  * 支持最大128MB SPI-Nor Flash
  * 支持最大256MB SPI-NAND Flash


- 安全
  
  * 安全启动、安全调试
  * XIP QSPI On-The-Fly AES 解密 (OTFAD)

  * 支持 RISC-V 安全 zone 分区
  * AES-CBC/GCM/XTS 模式
  * MD5，SHA-1/224/256/384/512
  * TRNG（真随机数生成器）
  * 用于 RSA/ECC 的 PKA（公钥加速器）


- 外设

  * USB 2.0 HS OTG
  * 以太网 RMII 接口
  * SD 卡接口
  * 4 个UART接口（支持RS485、ISO 17987-8、ISO 11898-1）
  * 2 个SPI接口 (Max Freq 80MHz)
  * 4 个I2C接口
  * 8 个PWM通道
  * I2S接口
  * PDM接口
  * 通用ADC
  * 通用DAC
  * 通用模拟比较器 (ACOMP)
  * PIR（被动红外）检测
  * IR remote 硬件加速器
  * 支持 12 通道 Touch
  * 可配置的 36 或 40 个 GPIO

- 功耗（超低功耗模式）

  * 关闭 (~1uA)、休眠
  * 断电睡眠

- 时钟

  * 支持 XTAL 24/26/32/38.4/40 MHz
  * 支持 XTAL 32/32.768 KHz
  * 内部 RC 32KHz/32MHz 振荡器
  * 内部系统 PLL

- 封装类型

  * 88 pin QFN

.. raw:: latex

   \end{multicols}
   \tableofcontents
   \listoffigures
   \listoftables

.. toctree::
   :maxdepth: 2
   :numbered:

   content/overview
   content/Functional
   content/Pindefinition
   content/RFCharacteristic
   content/AudioCharacteristic
   content/PowerConsumption
   content/Electricalcharacteristics
   content/ProductUse
   content/Referencedesign
   content/PackagingQFN88
   content/TopMarkingDefinition
   content/OrderingInformation
   content/version
