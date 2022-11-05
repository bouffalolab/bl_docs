===========
USB
===========

Overview
=====
Universal Serial Bus (USB) is an external bus standard that establishes specifications for the connection and communication between computers and external devices. xx supports USB 2.0 (HighSpeed + FullSpeed) and can be used as a host controller (Host) or a peripheral controller (Device).
As a Host, it contains a USB Host that supports all-speed transactions. Without software intervention, the Host can handle the transaction-based data structure by itself to reduce the load of CPU, and automatically send and receive data on the USB bus. As a Device, each endpoint except endpoint 0 supports the transfer types in the USB specification to adapt to various applications. In addition, xx's USB controller complies with the OTG standard and supports the session request protocol (SRP) and host negotiation protocol (HNP).

Features
=========
- Compatible with USB 2.0 (HighSpeed + FullSpeed)

- Compatible with OTG revision1.3

- UTMI + Level 3

- OTG SRP and HNP protocols

- Host, OTG, and Device modes

- Supports LPM

- Compatible with EHCI data structure (FSTN and SITD unsupported)

- Nine endpoints (EP0: control endpoint EP1EP8: interrupt/isolation/bulk endpoint)

- One dedicated FIFO for control transmission and four general FIFOs (control FIFO: 64 bytes; general FIFO: 512 bytes)

- Supports 2-FIFO and 3-FIFO modes

- Supports DMA

- Supports VDMA

Functional Description
===========
.. USB框架描述
.. -------------------------------

.. .. figure:: ../../picture/USB_block_diagram.svg
..    :align: center
.. 
.. .. figure:: ../../picture/USB_host_schedule.svg
..    :align: center



Operating Steps of USB
--------------

1. Configure internal transceiver

2. Configure USB controller

3. Configure USB interrupt

4. Configure USB DMA/VDMA (direct reading and writing of FIFO by CPU is unsupported)

5. Finish


Partial Register Configuration and Functional Description
-------------------------------


Interrupt Processing Flow for USB Enumeration Phase
--------------------------


