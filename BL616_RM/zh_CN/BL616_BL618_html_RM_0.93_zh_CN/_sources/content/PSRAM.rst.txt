===================
PSRAM Contorller
===================

简介
=============

芯片内拥有一个 PSRAM(Pseudo static random access memory) 控制器，可以控制多个型号的 PSRAM , PSRAM 支持内封或者外置，PSRAM 初始化完成后即可当作普通 RAM 使用，使用 PSRAM 可以极大提高芯片的 RAM 空间。

主要特征
===========

  - 支持多个型号的 PSRAM
    - 支持 Winbond 系列和 APmemory 系列
  - 支持自动刷新
  - 支持 Linear burst、Warp burst 和 Hybrid burst
  - 支持 X8 接口
  - 最大支持 8MB PSRAM 扩展
  - 控制器最大支持 PSRAM 频率 200Mhz
  - 支持单端和差分 PSRAM clock 时钟


功能描述
==========

PSRAM 模块基本框图如图所示。

.. figure:: ../../picture/pSRAM_x8_common.svg
   :align: center

   PSRAM基本框图


TODO

PSRAM 读时序

PSRAM 写时序

