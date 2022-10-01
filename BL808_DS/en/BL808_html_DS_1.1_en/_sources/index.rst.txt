
Features
============
.. raw:: latex

   \section*{Features}
   \begin{multicols}{2}

- Wireless (Tier-1 RF Performance)
  
  * 2.4 GHz RF transceiver
  * Wi-Fi 802.11 b/g/n
  * Bluetooth® 5.x Dual-mode (BT+BLE)
  * Zigbee / IEEE 802.15.4
  * Wi-Fi/Bluetooth/Zigbee Coexistence
  * Integrated balun, PA/LNA
  * Support External PA/LNA
  
- Microcontroller
  
  * Multi-Core RISC-V CPUs (Max Freq 480MHz)
  * RTC timer up to one year
  * General purpose timers
  * DMA channels
  * JTAG development support
  * XIP QSPI flash support
  
- Audio Codec

  * ADC*2 (Mic*2 or Mic*1+Line-in)
  * DAC*1 (Speaker)
  * Sample Rate 8~192KHz, 24bit


- Video/Image/Display

  * MJPEG , H264 (Baseline/Main)
  * Maximum resolution:2M(1920x1080)
  * Video encoding format:

     + MJPEG and H264 (Baseline/Main)
     + 1920x1080 @ 30fps + 640x480 @ 30fps
     + Up to 8-ROI(region-of-interest)
  * Camera Sensor interface: DVP and MIPI-CSI
  * Display interface: SPI、DBI、DPI(RGB)、MIPI-DSI


- AI NN general HW Accelerator

  * NPU BLAI-100 (Bouffalo Lab AI engine)
    For Video/Audio detection/recognition, etc.

- Memory

  * Embedded 32/64MB DRAM
  * Support max 128MB SPI-Nor Flash
  * Support max 256MB SPI-NAND Flash

- Security
  
  * Secure boot ; Secure debug
  * XIP QSPI On-The-Fly AES Decryption (OTFAD)
  * Support sensitive SW isolation (TrustZone)
  * AES-CBC/CCM/GCM/XTS modes
  * MD5, SHA-1/224/256/384/512
  * TRNG (True Random Number Generator)
  * PKA (Public Key Accelerator) for RSA/ECC


- Peripherals

  * USB 2.0 HS OTG
  * Ethernet RMII Interface
  * SD-card interface
  * Four UART interface
  * Two SPI interface
  * Four I2C interface
  * Eight PWM channels
  * I2S interface
  * PDM interface
  * General-Purpose ADC
  * General-Purpose DAC
  * General analog comparators (ACOMP)
  * PIR (Passive Infra-Red) detection
  * IR remote HW accelerator
  * Support 12-Channel Touch
  * Flexible 36 or 40 GPIOs

- Power Modes (Ultra-low Power modes)

  * Off(~1uA) ; Hibernate 
  * Power Down Sleep (flexible)

- Clock

  * Support XTAL 24/26/32/38.4/40 MHz
  * Support XTAL 32/32.768 KHz
  * Internal RC 32KHz/32MHz oscillator
  * Internal System PLLs

- Package Type

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
