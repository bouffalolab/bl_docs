=====
概述
=====
BL602/BL604是一款Wi-Fi + BLE组合的芯片组，用于低功耗和高性能应用开发。

无线子系统包含2.4G无线电，Wi-Fi 802.11b/g/n和BLE 5.0 基带/MAC设计。 
微控制器子系统包含一个低功耗的32位RISC CPU，高速缓存和存储器。 
电源管理单元控制低功耗模式。 此外，还支持各种安全性能。

外围接口包括SDIO，SPI，UART，I2C，IR remote，PWM，ADC，DAC，PIR和GPIO。

.. figure:: picture/Functionalblockdiagram.png
   :align: center

   功能框图

无线
===========
- 2.4GHz射频收发器
- Wi-Fi 802.11b/g/n
- Bluetooth®低能耗 5.0

  支持BLE 5.0通道选择＃2

  不支持2M PHY /编码PHY / ADV扩展
- Wi-Fi 20MHz带宽
- Wi-Fi 安全WPS/WEP/WPA/WPA2 Personal/WPA2 Enterprise/WPA3
- STA，SoftAP，STA+SoftAP和Sniffer模式
- 支持多个云连接
- BLE协助实现Wi-Fi快速连接
- Wi-Fi和BLE共存
- 集成balun，PA/LNA

MCU子系统
===========
- 带FPU（浮点单元）的32位RISC CPU
- 一级缓存
- 一个RTC计时器一年更新
- 两个32位通用定时器
- 四个DMA通道
- DFS（动态频率缩放）从1MHz到192MHz
- JTAG开发支持
- XIP QSPI Flash具有硬件加密支持

内存
========
- 276KB RAM
- 128KB ROM
- 1Kb eFuse
- 嵌入式Flash闪存 (选配)

安全机制
=========
- 安全启动
- 安全调试端口
- QSPI Flash即时AES解密（OTFAD）- AES - 128，CTR模式
- 支持AES 128/192/256位加密引擎
- 支持SHA-1/224/256
- 真实随机数发生器(TRNG)
- 公钥加速器(PKA)

外设
=====
- 一个SDIO2.0从机
- 一个SPI 主/从机
- 两个UART
- 一个I2C 主/从机
- 五个PWM通道
- 10-bit 通用DAC
- 12-bit 通用ADC
- 两个通用模拟比较器（ACOMP）
- PIR（被动红外）检测
- IR remote
- 16或23个GPIO

电源管理模式
=============
- 关闭
- 休眠（灵活模式）
- 掉电睡眠（灵活模式）
- 正常运作

时钟架构
=========
- 支持外部晶振频率24/32/38.4/40/MHz
- 内部RC 32kHz振荡器
- 内部RC 32MHz振荡器
- 内部系统PLL
- XTAL 32kHz 晶振

