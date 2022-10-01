===========
Emac
===========

Overview
=====
The EMAC module is a 10/100Mbps Ethernet Media Access Controller (Ethernet MAC) compatible with IEEE 802.3.
It consists of status and control register set, RX/TX module, RX/TX buffer descriptor (BD) set, master interface, MDIO interface, and physical layer chip (PHY) interface.

The status and control register set contains the status and control bits of EMAC. As the interface with the user program, it controls data sending and receiving and reports the status.

The RX/TX module obtains data frames from the specified memory according to the control words in the RX/TX descriptor, adds preamble and CRC, and expands short frames to send them through PHY. Or, it receives data from PHY, and puts them into the specified memory according to the RX/TX BD. Relevant event flags are set after sending and receiving are completed. If the event interrupt is enabled, an interrupt request will be sent to the master for processing.

MDIO and MII/RMII interfaces communicate with PHY, including reading and writing the registers of PHY and sending and receiving data packets.

Features
=========
- Compatible with the MAC layer defined by IEEE 802.3

- PHY supporting MII/RMII interface defined by IEEE 802.3

- Interacts with PHY through MDIO interface

- Supports 10 Mbps and 100 Mbps Ethernet

- Supports half-duplex and full-duplex

- Supports automatic flow control and control frame generation in the full-duplex mode

- Supports collision detection and retransmission in the half-duplex mode

- Supports the generation and verification of CRC

- Generates and removes data frame preamble

- Supports automatic extension of short data frames when sending

- Detects too long/short data frames (length limit)

- Transmits long data frames (\> standard Ethernet frame length)

- Automatically discards data packets with over-limit retransmission times or too small frame gap

- Broadcast packet filtering

- Internal RAM for storing up to 128 BDs

- Splits and configures a data packet to multiple consecutive Bds when sending

- Various event flags sent or received

- Generates a corresponding interrupt when an event occurs

Functional Description
===========
The composition of EMAC module is as follows.

.. figure:: ../../picture/EMAC.svg
   :align: center

   Block diagram of EMAC

Through MDIO interface, the control register can read and write PHY's registers, to perform configuration, mode selection (half/full duplex), negotiation, and other operations.
The RX module filters and checks the received data frames for a valid preamble, FCS, and length, and stores the data in the specified memory address according to the BD.
The TX module gets data from the memory according to the data BD, adds preamble, FCS, and pad, and then sends them out using the CSMA/CD protocol.
If CRS is detected, retry will be delayed.
The RX/TX BD set is connected to the system RAM, which is used to store the sent and received Ethernet data frames. Each descriptor contains the corresponding control status word and buffer memory address. There are 128 descriptors for RX/TX, and can be allocated flexibly.

Clock
============
EMAC needs a clock for synchronous transmission and reception (25 MHz (MII) or 50 MHz (RMII) at 100 Mbps, and 2.5 MHz at 10 Mbps).
The clock must be synchronized between EMAC and PHY.

RX/TX BD
==========================================
The RX/TX BD provides the association between EAMC and data frame cache address information, controls RX/TX data frames, and gives RX/TX status prompt.
Each descriptor consists of two consecutive words (1 word = 32 bits). The word with low address provides the length, control bits, and status bits of the data frames contained in this buffer. The word with high address is the memory pointer.
More details of BD are presented in the register description section.
It should be noted that BD must be written by word.
EMAC supports 128 BDs, which are shared by the RX/TX logic and can be freely combined. But the TX BD always occupies the preceding contiguous area. The number of BD is specified by the TXBDNUM field in the register MAC_TX_BD_NUM.
EMAC circularly processes the RX/TX BDs according to their order, until it finds the BDs marked WR, and goes back to the first RX/TX BD respectively.

PHY Interaction
============
The interactive register set of PHY provides a way to communicate commands and data needed for interaction with PHY. EMAC controls the working mode of PHY through MDIO interface, and ensures the matching of both working modes (such as rate, full/half-duplex).
Data packets interact between EMAC and PHY through MII/RMII interface, which is selected by the RMII_EN bit in the EMAC's mode register (EMAC_MODE): When this bit is 1, RMII mode is selected, and otherwise the MII mode is selected.
Both MII and RMII modes support the transmission rates of 10 Mbps and 100 Mbps specified in IEEE 802.3u. The transmission signals of MII and RMII are described as follows.

.. table:: Transmission signal 

    +----------------------+----------------------------------+--------------------------------------------+
    | Name                 | MII                              | RMII                                       |
    +======================+==================================+============================================+
    | EXTCK_EREFCK         | ETXCK: Send clock signal         | EREFCK:reference clock                     |
    +----------------------+----------------------------------+--------------------------------------------+
    | ECRS                 | ECRS: carrier detection          | \-                                         |
    +----------------------+----------------------------------+--------------------------------------------+
    | ECOL                 | ECOL: collision detection        | \-                                         |
    +----------------------+----------------------------------+--------------------------------------------+
    | ERXDV                | ERXDV: valid data                | ECRSDV: carrier detection/valid data       |
    +----------------------+----------------------------------+--------------------------------------------+
    | ERX0-ERX3            | ERX0ERX3: 4-bit received data    | ERX0ERX1: 2-bit received data              |
    +----------------------+----------------------------------+--------------------------------------------+
    | ERXER                | ERXER: Receive error indication  | ERXER: Receive error indication            |
    +----------------------+----------------------------------+--------------------------------------------+
    | ERXCK                | ERXCK: Receive clock signal      | \-                                         |
    +----------------------+----------------------------------+--------------------------------------------+
    | ETXEN                | ETXEN: TX enable                 | ETXEN: TX enable                           |
    +----------------------+----------------------------------+--------------------------------------------+
    | ETX0-ETX3            | ETX0ETX3: 4-bit sent data        | ETX0ETX1: 2-bit sent data                  |
    +----------------------+----------------------------------+--------------------------------------------+
    | ETXER                | ETXER: Send error indication     | \-                                         |
    +----------------------+----------------------------------+--------------------------------------------+
    | EMDC                 | MDIO Clock                       | MDIO Clock                                 |
    +----------------------+----------------------------------+--------------------------------------------+
    | EMDIO                | MDIO Data Input Output           | MDIO Data Input Output                     |
    +----------------------+----------------------------------+--------------------------------------------+

The RMII interface has fewer pins, and 2-bit data lines are used for transmission and reception. At 100 Mbps, a reference clock of 50 MHz is required.

Programming Flow
===========

PHY initialization
-----------
- Select a proper connection mode by setting the RMII_EN bit in the register EMAC_MODE according to the PHY type.
- Set the MAC address of EMAC to EMAC_MAC_ADDR0 and EMAC_MAC_ADDR1
- Set an appropriate clock for the MDIO part by programming the CLKDIV field in the register EMAC_MIIMODE
- Set the address of the corresponding PHY to the FIAD field of the register EMAC_MIIADDRESS
- According to PHY's manual, send commands through registers EMAC_MIICOMMAND and EMAC_MIITX_DATA
- Store the data read from PHY in the register EMAC_MIIRX_DATA
- Query the status of interaction with PHY commands through the register EMAC_MIISTATUS

After basic interaction is completed, PHY shall be switched to the auto-negotiation state. Upon negotiation completed, the mode must be programmed to the FULLD bit in the EMAC_MODE register based on the negotiation result.

Send Data Frame
------------------
- Configure data frame format and interval bit fields in the register EMAC_MODE

- Specify the number of TX BDs by setting the TXBDNUM field in the register EMAC_TX_BD_NUM, so that the rest is the number of RX BDs

- Prepare the data frames to be sent in the memory

- Fill in the address of data frames in the data pointer field (word 1) corresponding to the TX BDs

- Clear the status flag in the control and status fields (word 0) corresponding to the TX BDs, and set the control field (CRC enable, PAD enable, and interrupt enable)

- Write the data frame length, and set the RD field to inform EMAC that this BD data needs to be sent. If necessary, set the upper IRQ bit to enable interrupt

- Especially, if it is the last TX BD, the upper WR bit must be set. EMAC will \"go back\" to the first TX BD after this BD is processed

- If there are multiple BDs to be sent, repeat the steps of setting BD to pad all the TX BDs

- If one packet is contained in only one BD, set its EOF bit to 1

- If one packet is sent in multiple BDs, just mark the last BD it occupies as the end of the packet by setting the EOF bit

- To enable the TX interrupt, configure the TX-related bits in the register EMAC_INT_MASK

- Configure the TXEN bit in the register EMAC_MODE to enable TX

- If an interrupt is enabled, in the TX interrupt, obtain the current BD through the TXBDNUM field in the register EMAC_TX_BD_NUM

- Complete processing based on the status word of the current BD

- For BDs whose data has been sent, the RD bit in its control field will be cleared by hardware and BDs will not be used for TX again. Only after new data is padded and RD is set, can this BD be used for TX again

Receive Data Frame
--------------------
- Configure data frame format and interval bit fields in the register EMAC_MODE

- Specify the number of TX BDs by setting the TXBDNUM field in the register EMAC_TX_BD_NUM, so that the rest is the number of RX BDs

- Prepare an area in memory for receiving data

- Fill in the address of data frames in the data pointer field (word 1) corresponding to the RX BDs

- Clear the status flag in the control and status fields (word 0) corresponding to the RX BDs, and set the control field (interrupt enable)

- Write the receivable data frame length, and set the E-bit field to inform EMAC that this BD is free and can receive data. If necessary, set the upper IRQ bit to enable the interrupt

- Especially, if it is the last valid RX BD, the upper WR bit must be set. EMAC will \"go back\" to the first RX BD after this BD is processed

- If there are multiple BDs for receiving data, repeat the steps of setting BD to pad all the BDs

- To enable the RX interrupt, configure the RX-related bits in the register EMAC_INT_MASK

- Configure the RXEN bit in the register EMAC_MODE to enable RX

- If an interrupt is enabled, in the RX interrupt, obtain the current BD through the RXBDNUM field in the register EMAC_TX_BD_NUM

- Complete processing based on the status word of the current BD

- For BDs whose data has been received, the E bit in its control field will be cleared by hardware and BDs will not be used for RX again. Only after you take out the data and set the E bit, can this BD be used for RX again

.. only:: html

   .. include:: emac_register.rst

.. raw:: latex

   \input{../../en/content/emac}