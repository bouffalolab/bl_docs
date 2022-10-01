==========
Overview
==========
BL808 is a highly integrated AIoT chipset with 
Wi-Fi/BT/BLE/Zigbee,
Multi-Core CPUs, Audio Codec
, Video Codec and AI HW accelerator
for high-performance & low-power application.


BL808 mainly includes two subsystems, wireless and multimedia.

The wireless subsystem includes a RISC-V 32-bit high-performance CPU, integrated Wi-Fi /BT/Zigbee wireless subsystem, which can realize a variety of wireless connections and data transmission, and provide a variety of connection and transmission experiences.

The multimedia subsystem includes a RISC-V 64-bit ultra-high-performance CPU and integrates video processing modules such as 
DVP/CSI/
H264/NPU,
which can be widely used in various AI fields such as video surveillance/smart speakers.

The components of the multimedia subsystem are as follows:

- NPU HW NN co-processor (BLAI-100) generally used for AI applications
- Camera interface
- Audio Codec
- Video Codec
- Sensor
- Display interfaces

Power Management Unit provides low-power modes. Moreover, variety of security features are supported.

Peripheral interfaces include USB2.0, Ethernet, SD/MMC, SPI, UART, I2C, I2S, PWM, GPDAC, GPADC, ACOMP, PIR, Touch, IR remote, Display and GPIOs.

Flexible GPIO configurations are supported. BL808 has total 36 or 40 GPIOs.

.. figure:: ../../picture/Functionalblockdiagram.svg
   :align: center

   Block Diagram

