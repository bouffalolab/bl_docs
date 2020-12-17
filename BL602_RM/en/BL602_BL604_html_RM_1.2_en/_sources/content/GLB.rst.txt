===========
GLB
===========

Introduction
==================
GLB (Global Register) is a chip's general global setting module, which mainly includes 
functions such as clock management, reset management, bus management, memory management, 
and GPIO management.

GLB function description
============================
Clock
-------------
The clock management function is mainly used to set the clock of the processor, bus, 
and various peripherals. This module can set the clock source, clock frequency 
division, etc. of the module's work, and can also achieve the gate control of 
the module's clock to achieve the purpose of low power consumption of the system.

For detailed settings, please refer to the relevant chapter of the system clock.

Reset 
--------------------
Provide individual reset function for each peripheral and chip reset function. 

The chip reset includes:

- CPU reset: just reset the CPU module, the program will run again, and the peripherals will not be reset

- System reset: each peripheral and CPU will be reset, but the related registers of the AON domain will not be reset

- Power-on reset: the entire system including the AON domain related registers will be reset

The application can choose to use the corresponding reset method as required.

Bus 
-----------------
Provide bus arbitration settings and bus error settings. You can set whether to 
generate an interrupt when a bus error occurs, and provide error bus address 
information to facilitate user debugging procedures.

Memory 
----------------------
Provides the power management of each memory module in the low-power mode 
of the chip system, including two setting modes:

- retention mode：In this mode, the data on the memory can be saved, but cannot be read or written until exiting the low power mode.

- sleep mode：In this mode, the data in the memory will be lost and is only used to reduce system power consumption.

GPIO overview
----------------
The GPIO management function provides GPIO control registers to realize the 
configuration of GPIO attributes by software, so that users can conveniently operate 
GPIO. Each GPIO can be configured as three modes of input, output and optional 
function. In each mode (except for analog optional functions), it provides three 
port states: pull-up, pull-down, and floating. In addition, GPIO also provides 
interrupt functions, which can be configured as rising edge trigger, falling edge 
trigger, or High/low level trigger.

GPIO main features
----------------------

- It can be configured as a normal input / output function. In this mode, pull-up, pull-down or floating input/output can be set.

- It can be configured as an optional function and used with peripheral functions. In this mode, pull-up and pull-down can also be set. When using the analog function, it must be set to floating.

- The drive capability can be set to provide greater output current.

- Schmitt trigger function can be set to provide simple hardware anti-shake function.

GPIO function description
-------------------------------
Each GPIO can be configured by software as:

- Floating input
- Pull-up input
- Pull down input
- Pull-up interrupt input
- Pull-down interrupt input
- Floating interrupt input
- Pull-up output
- Pull-down output
- Floating output
- Analog input optional function
- Analog output optional function
- Digital optional functions

The basic block diagram of the GPIO module is shown below:

.. figure:: ../../picture/GPIOBasicStruct.png
   :align: center

   GPIO Basic Struct

GPIO function 
------------------

The function of GPIO is set through the GPIO_CFGCTL register group. The main setting items include:

- func_sel: select GPIO function
- pu: choose whether to pull up
- pd: choose whether to pull down
- drv: set the driving capability
- smt: select whether to enable Schmitt trigger
- ie: set input enable
- oe: set output enable

The functions that GPIO can set include:

- Flash/QSPI：set GPIO as QSPI function, can be connected to Flash as program storage / run medium
- SPI: set GPIO as SPI function
- I2C: set GPIO to I2C function
- UART: set GPIO as UART function
- PWM: set GPIO to PWM function
- ANA: set GPIO to Analog function
- SWGPIO: set GPIO as general IO function
- JTAG: set GPIO to JTAG function

In order to meet the needs of customers as much as possible, each of the 
GPIOs can basically select the above optional functions. When selecting an 
optional function, the GPIO and corresponding function signals are shown 
in the following table:

.. table:: GPIO function table 

    +--------+------------+------------+------------+------------+------------+------------+-------------+------------+------------+
    | GPIO   |    SDIO    |    FLASH   |    SPI     |    I2C     |    UART    |    PWM     |    Analog   |    SWGPIO  |    JTAG    |
    +--------+------------+------------+------------+------------+------------+------------+-------------+------------+------------+
    | GPIO0  |      CLK   |    D1      |     MISO   |     SCL    |      SIG0  |     CH0    |             | SWGPIO0    |     TMS    |
    +--------+------------+------------+------------+------------+------------+------------+-------------+------------+------------+
    | GPIO1  |      CMD   |    D2      |     MOSI   |     SDA    |      SIG1  |     CH1    |             | SWGPIO1    |     TDI    |
    +--------+------------+------------+------------+------------+------------+------------+-------------+------------+------------+
    | GPIO2  |      DAT0  |    D2      |     SS     |     SCL    |      SIG2  |     CH2    |             | SWGPIO2    |     TCK    |
    +--------+------------+------------+------------+------------+------------+------------+-------------+------------+------------+
    | GPIO3  |      DAT1  |    D3      |     SCLK   |     SDA    |      SIG3  |     CH3    |             | SWGPIO3    |     TDO    |
    +--------+------------+------------+------------+------------+------------+------------+-------------+------------+------------+
    | GPIO4  |      DAT2  |            |     MISO   |     SCL    |      SIG4  |     CH4    |   CH1       | SWGPIO4    |     TMS    |
    +--------+------------+------------+------------+------------+------------+------------+-------------+------------+------------+
    | GPIO5  |      DAT3  |            |     MOSI   |     SDA    |      SIG5  |     CH0    |   CH4       | SWGPIO5    |     TDI    |
    +--------+------------+------------+------------+------------+------------+------------+-------------+------------+------------+
    | GPIO6  |            |            |     SS     |    SCL     |      SIG6  |     CH1    |   CH5       | SWGPIO6    |     TCK    |
    +--------+------------+------------+------------+------------+------------+------------+-------------+------------+------------+
    | GPIO7  |            |            |     SCLK   |     SDA    |      SIG7  |     CH2    |             | SWGPIO7    |     TDO    |
    +--------+------------+------------+------------+------------+------------+------------+-------------+------------+------------+
    | GPIO8  |            |            |     MISO   |     SCL    |      SIG0  |     CH3    |             | SWGPIO8    |     TMS    |
    +--------+------------+------------+------------+------------+------------+------------+-------------+------------+------------+
    | GPIO9  |            |            |     MOSI   |     SDA    |      SIG1  |     CH4    |  CH6/7      | SWGPIO9    |     TDI    |
    +--------+------------+------------+------------+------------+------------+------------+-------------+------------+------------+
    | GPIO10 |            |            |     SS     |     SCL    |      SIG2  |     CH0    |MICBIAS/CH8/9| SWGPIO10   |     TCK    |
    +--------+------------+------------+------------+------------+------------+------------+-------------+------------+------------+
    | GPIO11 |            |            |     SCLK   |     SDA    |      SIG3  |     CH1    |IROUT/CH10   | SWGPIO11   |     TDO    |
    +--------+------------+------------+------------+------------+------------+------------+-------------+------------+------------+
    | GPIO12 |            |            |     MISO   |     SCL    |      SIG4  |     CH2    |ADC_VREF/CH0 | SWGPIO12   |     TMS    |
    +--------+------------+------------+------------+------------+------------+------------+-------------+------------+------------+
    | GPIO13 |            |            |     MOSI   |     SDA    |      SIG5  |     CH3    |    CH3      | SWGPIO13   |     TDI    |
    +--------+------------+------------+------------+------------+------------+------------+-------------+------------+------------+
    | GPIO14 |            |            |     SS     |     SCL    |      SIG6  |     CH4    |    CH2      | SWGPIO14   |     TCK    |
    +--------+------------+------------+------------+------------+------------+------------+-------------+------------+------------+
    | GPIO15 |            |            |     SCLK   |     SDA    |      SIG7  |     CH0    |PSWIROUT/CH11| SWGPIO15   |     TDO    |
    +--------+------------+------------+------------+------------+------------+------------+-------------+------------+------------+
    | GPIO16 |            |            |     MISO   |     SCL    |      SIG0  |     CH1    |             | SWGPIO16   |     TMS    |
    +--------+------------+------------+------------+------------+------------+------------+-------------+------------+------------+
    | GPIO17 |            |    D3      |     MOSI   |     SDA    |      SIG1  |     CH2    |DC_TP_OUT    | SWGPIO17   |     TDI    |
    +--------+------------+------------+------------+------------+------------+------------+-------------+------------+------------+
    | GPIO18 |            |    D2      |     SS     |     SCL    |      SIG2  |     CH3    |             | SWGPIO18   |     TCK    |
    +--------+------------+------------+------------+------------+------------+------------+-------------+------------+------------+
    | GPIO19 |            |    D1      |     SCLK   |     SDA    |      SIG3  |     CH4    |             | SWGPIO19   |     TDO    |
    +--------+------------+------------+------------+------------+------------+------------+-------------+------------+------------+
    | GPIO20 |            |    D0      |     MISO   |     SCL    |      SIG4  |     CH0    |             | SWGPIO20   |     TMS    |
    +--------+------------+------------+------------+------------+------------+------------+-------------+------------+------------+
    | GPIO21 |            |    CS      |     MOSI   |     SDA    |      SIG5  |     CH1    |             | SWGPIO21   |     TDI    |
    +--------+------------+------------+------------+------------+------------+------------+-------------+------------+------------+
    | GPIO22 |            |    CLK_OUT |    SS      |     SCL    |      SIG6  |     CH2    |             | SWGPIO22   |     TCK    |
    +--------+------------+------------+------------+------------+------------+------------+-------------+------------+------------+

In the above table, when the UART function is selected, only one signal of 
the UART is selected, and the specific function of the pin is not specified 
(such as UART TX or UART RX). It is also necessary to use 
UART_SIGX_SEL(X = 0-7) to select specific UART signals and corresponding functions.

The signals that can be selected for each UART_SIGX_SEL include:

- 0 : UART0_RTS
- 1 : UART0_CTS
- 2 : UART0_TXD
- 3 : UART0_RXD
- 4 : UART1_RTS
- 5 : UART1_CTS
- 6 : UART1_TXD
- 7 : UART1_RXD

Take GPIO0 as an example, when fun_sel selects UART, GPIO0 selects UART_SIG0. 
By default, the value of UART_SIG0_SEL is 0, which is UART0_RTS, that is, 
GPIO is UART0_RTS function. If the application wants to use GPIO as UART1_TXD, 
as long as UART_SIG0_SEL is set to 6, then the function of GPIO0 is UART1_TXD.

GPIO output 
----------------

By setting func_sel to SWGPIO, GPIO can be used as the input / output of ordinary GPIO. 
Setting ie to 0 and oe to 1 can configure GPIO as an output function. The output value 
is set through the GPIO_O register group.

When the corresponding bit of GPIO_O is set to 0, the GPIO output is low, and when the 
corresponding bit of GPIO_O is set to 1, the GPIO output is high. The output capability 
can be set via the DRV control bit.

GPIO input 
--------------

Set func_sel to SWGPIO, set ie to 1, and oe to 0. The user can configure the GPIO as an 
input function, set whether to enable the Schmitt trigger through the smt control bit, 
and set the pull-down property through the pd, pu control bit .

The value of the external input can be obtained by reading the corresponding bit of 
the GPIO_I register.

GPIO optional function 
---------------------------

Setting func_sel as the corresponding peripheral function can realize the connection 
between GPIO and peripherals, and realize the input and output of peripherals. 
As can be seen from the basic functional block diagram of GPIO, when selecting 
optional functions, it is necessary to set ie to 1, oe Set to 0, that is to disconnect 
the output control function of ordinary GPIO.

In this way, for peripherals with fixed input functions, the OE signal of the peripheral 
is always 0 to implement the input function; for peripherals with fixed output, the OE 
signal is always 1 so that the output is controlled by the peripheral. At this time, 
The input signal is the output signal, but it will not be collected by the output peripheral. 
When the peripheral needs both input and output, the input and output can be realized by 
controlling the peripheral OE signal.

GPIO interrupt 
------------------

To use the GPIO interrupt function, the user needs to set the GPIO to the input mode first, 
and the interrupt trigger mode is set through the GPIO_INT_MODE_SET register group. 
The interrupt modes that can be set include:

- Interrupt on rising edge
- Interrupt on falling edge
- High level trigger interrupt
- Low level trigger interrupt

Each GPIO can be set as an interrupt function. Whether to enable a GPIO interrupt can 
be set through the GPIO_INT_MASK register. When an interrupt occurs, the GPIO pin number 
that generated the interrupt can be obtained through the GPIO_INT_STAT register in the 
interrupt function. Clear the corresponding interrupt signal through GPIO_INT_CLR.


Register description
==========================

+-----------------------+------------------------------------+
| Name                  | Description                        |
+-----------------------+------------------------------------+
| `clk_cfg0`_           | Clock configuration-processor, bus |
+-----------------------+------------------------------------+
| `clk_cfg2`_           | Clock configuration-UART,Flash     |
+-----------------------+------------------------------------+
| `clk_cfg3`_           | Clock configuration-I2C,SPI        |
+-----------------------+------------------------------------+
| `GPADC_32M_SRC_CTRL`_ | Clock configuration-GPADC          |
+-----------------------+------------------------------------+
| `GPIO_CFGCTL0`_       | GPIO0, GPIO1 configuration         |
+-----------------------+------------------------------------+
| `GPIO_CFGCTL1`_       | GPIO2, GPIO3 configuration         |
+-----------------------+------------------------------------+
| `GPIO_CFGCTL2`_       | GPIO4, GPIO5 configuration         |
+-----------------------+------------------------------------+
| `GPIO_CFGCTL3`_       | GPIO6, GPIO7 configuration         |
+-----------------------+------------------------------------+
| `GPIO_CFGCTL4`_       | GPIO8, GPIO9 configuration         |
+-----------------------+------------------------------------+
| `GPIO_CFGCTL5`_       | GPIO10, GPIO11 configuration       |
+-----------------------+------------------------------------+
| `GPIO_CFGCTL6`_       | GPIO12, GPIO13 configuration       |
+-----------------------+------------------------------------+
| `GPIO_CFGCTL7`_       | GPIO14, GPIO15 configuration       |
+-----------------------+------------------------------------+
| `GPIO_CFGCTL8`_       | GPIO16, GPIO17 configuration       |
+-----------------------+------------------------------------+
| `GPIO_CFGCTL9`_       | GPIO18, GPIO19 configuration       |
+-----------------------+------------------------------------+
| `GPIO_CFGCTL10`_      | GPIO20, GPIO21 configuration       |
+-----------------------+------------------------------------+
| `GPIO_CFGCTL11`_      | GPIO22, GPIO23 configuration       |
+-----------------------+------------------------------------+
| `GPIO_CFGCTL12`_      | GPIO24, GPIO25 configuration       |
+-----------------------+------------------------------------+
| `GPIO_CFGCTL13`_      | GPIO26, GPIO27 configuration       |
+-----------------------+------------------------------------+
| `GPIO_CFGCTL14`_      | GPIO28 configuration               |
+-----------------------+------------------------------------+

clk_cfg0
----------
 
**Address：**  0x40000000
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| GLBID                                         | RSVD                                          | BCLKDIV                                                                                       |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| HCLKDIV                                                                                       | RCSEL                 | PLLSEL                | RSVD                                          |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+-------------------------------------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                                                             |
+----------+----------+--------+-------------+-------------------------------------------------------------------------+
| 31:28    | GLBID    | R      | 4'H6        |                                                                         |
+----------+----------+--------+-------------+-------------------------------------------------------------------------+
| 27:24    | RSVD     |        |             |                                                                         |
+----------+----------+--------+-------------+-------------------------------------------------------------------------+
| 23:16    | BCLKDIV  | R/W    | 0           | bclk divide from hclk                                                   |
+----------+----------+--------+-------------+-------------------------------------------------------------------------+
| 15:8     | HCLKDIV  | R/W    | 0           | hclk divide from root clock (clock source selected by hbn_root_clk_sel) |
+----------+----------+--------+-------------+-------------------------------------------------------------------------+
| 7:6      | RCSEL    | R      | 0           | root clock selection from HBN (0: RC32M 1: XTAL  2/3: PLL others)       |
+----------+----------+--------+-------------+-------------------------------------------------------------------------+
| 5:4      | PLLSEL   | R/W    | 0           | pll clock selection (0: 48MHz 1: 120MHz  2: 160MHz  3: 192MHz)          |
+----------+----------+--------+-------------+-------------------------------------------------------------------------+
| 3:0      | RSVD     |        |             |                                                                         |
+----------+----------+--------+-------------+-------------------------------------------------------------------------+

clk_cfg2
----------
 
**Address：**  0x40000008
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| DMAEN                                                                                         | RSVD                                                                                          |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                  | SFSEL                 | SFEN      | SFDIV                             | HUCSEL    | RSVD                  | UARTEN    | RSVD      | UARTDIV                           |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                                                                               |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------+
| 31:24    | DMAEN    | R/W    | 8'HFF       | CH0, 1, 2, AHBm, AHBs, Rqs                                                                |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------+
| 23:14    | RSVD     |        |             |                                                                                           |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------+
| 13:12    | SFSEL    | R/W    | 2'D2        | Flash Clock Select (0: 120M, 1:80M, 2:HCLK, 3:96M)                                        |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------+
| 11       | SFEN     | R/W    | 1           | Flash Clock Enable                                                                        |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------+
| 10:8     | SFDIV    | R/W    | 3'D3        | Flash Clock Divider (Selected Flash Clock)/(N+1)                                          |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------+
| 7        | HUCSEL   | R      | 0           | uart clock selection from HBN (0: root clock 1: PLL 160M)                                 |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------+
| 6:5      | RSVD     |        |             |                                                                                           |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------+
| 4        | UARTEN   | R/W    | 1           | UART Clock Enable                                                                         |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------+
| 3        | RSVD     |        |             |                                                                                           |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------+
| 2:0      | UARTDIV  | R/W    | 3'D7        | UART Clock Divider (root clock or 160M)/(N+1) (clock source selected by hbn_uart_clk_sel) |
+----------+----------+--------+-------------+-------------------------------------------------------------------------------------------+

clk_cfg3
----------
 
**Address：**  0x4000000c
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                              | I2CEN     | I2CDIV                                                                                        |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                              | SPIEN     | RSVD                              | SPIDIV                                                    |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+-------------------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                                           |
+----------+----------+--------+-------------+-------------------------------------------------------+
| 31:25    | RSVD     |        |             |                                                       |
+----------+----------+--------+-------------+-------------------------------------------------------+
| 24       | I2CEN    | R/W    | 1           | I2C Master Clock Out Enable                           |
+----------+----------+--------+-------------+-------------------------------------------------------+
| 23:16    | I2CDIV   | R/W    | 8'D255      | I2C Master Clock Out Divider (Freq_of_BCLK/(N+1))     |
+----------+----------+--------+-------------+-------------------------------------------------------+
| 15:9     | RSVD     |        |             |                                                       |
+----------+----------+--------+-------------+-------------------------------------------------------+
| 8        | SPIEN    | R/W    | 1           | SPI Clock Enable (Default : Enable)                   |
+----------+----------+--------+-------------+-------------------------------------------------------+
| 7:5      | RSVD     |        |             |                                                       |
+----------+----------+--------+-------------+-------------------------------------------------------+
| 4:0      | SPIDIV   | R/W    | 5'D3        | SPI Clock Divider (BUS_CLK/(N+1)),  default BUS_CLK/4 |
+----------+----------+--------+-------------+-------------------------------------------------------+

GPADC_32M_SRC_CTRL
--------------------
 
**Address：**  0x400000a4
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                                                                                                                          |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                              | GADCDIV   | GADCSEL   | RSVD      | GADCDIV                                                               |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+-------------------------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                                                 |
+----------+----------+--------+-------------+-------------------------------------------------------------+
| 31:9     | RSVD     |        |             |                                                             |
+----------+----------+--------+-------------+-------------------------------------------------------------+
| 8        | GADCDIV  | R/W    | 1           | GPADC 32M Clock Dvider Enable                               |
+----------+----------+--------+-------------+-------------------------------------------------------------+
| 7        | GADCSEL  | R/W    | 0           | GPADC Clock Source Select.  0: 96MHz,  1: xclk              |
+----------+----------+--------+-------------+-------------------------------------------------------------+
| 6        | RSVD     |        |             |                                                             |
+----------+----------+--------+-------------+-------------------------------------------------------------+
| 5:0      | GADCDIV  | R/W    | 6'D2        | GPADC 32M Clock Divider (96M)/(N+1) , default : 96M/3 = 32M |
+----------+----------+--------+-------------+-------------------------------------------------------------+

GPIO_CFGCTL0
--------------
 
**Address：**  0x40000100
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                          | GP1FUNC                                       | RSVD                  | GP1PD     | GP1PU     | GP1DRV                | GP1SMT    | GP1IE     |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                          | GP0FUNC                                       | RSVD                  | GP0PD     | GP0PU     | GP0DRV                | GP0SMT    | GP0IE     |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+---------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                           |
+----------+----------+--------+-------------+---------------------------------------+
| 31:28    | RSVD     |        |             |                                       |
+----------+----------+--------+-------------+---------------------------------------+
| 27:24    | GP1FUNC  | R/W    | 4'H1        | GPIO Function Select (Default : SDIO) |
+----------+----------+--------+-------------+---------------------------------------+
| 23:22    | RSVD     |        |             |                                       |
+----------+----------+--------+-------------+---------------------------------------+
| 21       | GP1PD    | R/W    | 0           | GPIO Pull Down Control                |
+----------+----------+--------+-------------+---------------------------------------+
| 20       | GP1PU    | R/W    | 0           | GPIO Pull Up Control                  |
+----------+----------+--------+-------------+---------------------------------------+
| 19:18    | GP1DRV   | R/W    | 0           | GPIO Driving Control                  |
+----------+----------+--------+-------------+---------------------------------------+
| 17       | GP1SMT   | R/W    | 1           | GPIO SMT Control                      |
+----------+----------+--------+-------------+---------------------------------------+
| 16       | GP1IE    | R/W    | 1           | GPIO Input Enable                     |
+----------+----------+--------+-------------+---------------------------------------+
| 15:12    | RSVD     |        |             |                                       |
+----------+----------+--------+-------------+---------------------------------------+
| 11:8     | GP0FUNC  | R/W    | 4'H1        | GPIO Function Select (Default : SDIO) |
+----------+----------+--------+-------------+---------------------------------------+
| 7:6      | RSVD     |        |             |                                       |
+----------+----------+--------+-------------+---------------------------------------+
| 5        | GP0PD    | R/W    | 0           | GPIO Pull Down Control                |
+----------+----------+--------+-------------+---------------------------------------+
| 4        | GP0PU    | R/W    | 0           | GPIO Pull Up Control                  |
+----------+----------+--------+-------------+---------------------------------------+
| 3:2      | GP0DRV   | R/W    | 0           | GPIO Driving Control                  |
+----------+----------+--------+-------------+---------------------------------------+
| 1        | GP0SMT   | R/W    | 1           | GPIO SMT Control                      |
+----------+----------+--------+-------------+---------------------------------------+
| 0        | GP0IE    | R/W    | 1           | GPIO Input Enable                     |
+----------+----------+--------+-------------+---------------------------------------+

GPIO_CFGCTL1
--------------
 
**Address：**  0x40000104
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                          | GP3FUNC                                       | RSVD                  | GP3PD     | GP3PU     | GP3DRV                | GP3SMT    | GP3IE     |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                          | GP2FUNC                                       | RSVD                  | GP2PD     | GP2PU     | GP2DRV                | GP2SMT    | GP2IE     |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+---------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                           |
+----------+----------+--------+-------------+---------------------------------------+
| 31:28    | RSVD     |        |             |                                       |
+----------+----------+--------+-------------+---------------------------------------+
| 27:24    | GP3FUNC  | R/W    | 4'H1        | GPIO Function Select (Default : SDIO) |
+----------+----------+--------+-------------+---------------------------------------+
| 23:22    | RSVD     |        |             |                                       |
+----------+----------+--------+-------------+---------------------------------------+
| 21       | GP3PD    | R/W    | 0           | GPIO Pull Down Control                |
+----------+----------+--------+-------------+---------------------------------------+
| 20       | GP3PU    | R/W    | 0           | GPIO Pull Up Control                  |
+----------+----------+--------+-------------+---------------------------------------+
| 19:18    | GP3DRV   | R/W    | 0           | GPIO Driving Control                  |
+----------+----------+--------+-------------+---------------------------------------+
| 17       | GP3SMT   | R/W    | 1           | GPIO SMT Control                      |
+----------+----------+--------+-------------+---------------------------------------+
| 16       | GP3IE    | R/W    | 1           | GPIO Input Enable                     |
+----------+----------+--------+-------------+---------------------------------------+
| 15:12    | RSVD     |        |             |                                       |
+----------+----------+--------+-------------+---------------------------------------+
| 11:8     | GP2FUNC  | R/W    | 4'H1        | GPIO Function Select (Default : SDIO) |
+----------+----------+--------+-------------+---------------------------------------+
| 7:6      | RSVD     |        |             |                                       |
+----------+----------+--------+-------------+---------------------------------------+
| 5        | GP2PD    | R/W    | 0           | GPIO Pull Down Control                |
+----------+----------+--------+-------------+---------------------------------------+
| 4        | GP2PU    | R/W    | 0           | GPIO Pull Up Control                  |
+----------+----------+--------+-------------+---------------------------------------+
| 3:2      | GP2DRV   | R/W    | 0           | GPIO Driving Control                  |
+----------+----------+--------+-------------+---------------------------------------+
| 1        | GP2SMT   | R/W    | 1           | GPIO SMT Control                      |
+----------+----------+--------+-------------+---------------------------------------+
| 0        | GP2IE    | R/W    | 1           | GPIO Input Enable                     |
+----------+----------+--------+-------------+---------------------------------------+

GPIO_CFGCTL2
--------------
 
**Address：**  0x40000108
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                          | GP5FUNC                                       | RSVD                  | GP5PD     | GP5PU     | GP5DRV                | GP5SMT    | GP5IE     |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                          | GP4FUNC                                       | RSVD                  | GP4PD     | GP4PU     | GP4DRV                | GP4SMT    | GP4IE     |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+---------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                           |
+----------+----------+--------+-------------+---------------------------------------+
| 31:28    | RSVD     |        |             |                                       |
+----------+----------+--------+-------------+---------------------------------------+
| 27:24    | GP5FUNC  | R/W    | 4'H1        | GPIO Function Select (Default : SDIO) |
+----------+----------+--------+-------------+---------------------------------------+
| 23:22    | RSVD     |        |             |                                       |
+----------+----------+--------+-------------+---------------------------------------+
| 21       | GP5PD    | R/W    | 0           | GPIO Pull Down Control                |
+----------+----------+--------+-------------+---------------------------------------+
| 20       | GP5PU    | R/W    | 0           | GPIO Pull Up Control                  |
+----------+----------+--------+-------------+---------------------------------------+
| 19:18    | GP5DRV   | R/W    | 0           | GPIO Driving Control                  |
+----------+----------+--------+-------------+---------------------------------------+
| 17       | GP5SMT   | R/W    | 1           | GPIO SMT Control                      |
+----------+----------+--------+-------------+---------------------------------------+
| 16       | GP5IE    | R/W    | 1           | GPIO Input Enable                     |
+----------+----------+--------+-------------+---------------------------------------+
| 15:12    | RSVD     |        |             |                                       |
+----------+----------+--------+-------------+---------------------------------------+
| 11:8     | GP4FUNC  | R/W    | 4'H1        | GPIO Function Select (Default : SDIO) |
+----------+----------+--------+-------------+---------------------------------------+
| 7:6      | RSVD     |        |             |                                       |
+----------+----------+--------+-------------+---------------------------------------+
| 5        | GP4PD    | R/W    | 0           | GPIO Pull Down Control                |
+----------+----------+--------+-------------+---------------------------------------+
| 4        | GP4PU    | R/W    | 0           | GPIO Pull Up Control                  |
+----------+----------+--------+-------------+---------------------------------------+
| 3:2      | GP4DRV   | R/W    | 0           | GPIO Driving Control                  |
+----------+----------+--------+-------------+---------------------------------------+
| 1        | GP4SMT   | R/W    | 1           | GPIO SMT Control                      |
+----------+----------+--------+-------------+---------------------------------------+
| 0        | GP4IE    | R/W    | 1           | GPIO Input Enable                     |
+----------+----------+--------+-------------+---------------------------------------+

GPIO_CFGCTL3
--------------
 
**Address：**  0x4000010c
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                          | GP7FUNC                                       | RSVD                  | GP7PD     | GP7PU     | GP7DRV                | GP7SMT    | GP7IE     |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                          | GP6FUNC                                       | RSVD                  | GP6PD     | GP6PU     | GP6DRV                | GP6SMT    | GP6IE     |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                              |
+----------+----------+--------+-------------+------------------------------------------+
| 31:28    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 27:24    | GP7FUNC  | R/W    | 4'HB        | GPIO Function Select (Default : SWGPIO ) |
+----------+----------+--------+-------------+------------------------------------------+
| 23:22    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 21       | GP7PD    | R/W    | 0           | GPIO Pull Down Control                   |
+----------+----------+--------+-------------+------------------------------------------+
| 20       | GP7PU    | R/W    | 0           | GPIO Pull Up Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 19:18    | GP7DRV   | R/W    | 0           | GPIO Driving Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 17       | GP7SMT   | R/W    | 1           | GPIO SMT Control                         |
+----------+----------+--------+-------------+------------------------------------------+
| 16       | GP7IE    | R/W    | 1           | GPIO Input Enable                        |
+----------+----------+--------+-------------+------------------------------------------+
| 15:12    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 11:8     | GP6FUNC  | R/W    | 4'HB        | GPIO Function Select (Default : SWGPIO ) |
+----------+----------+--------+-------------+------------------------------------------+
| 7:6      | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 5        | GP6PD    | R/W    | 0           | GPIO Pull Down Control                   |
+----------+----------+--------+-------------+------------------------------------------+
| 4        | GP6PU    | R/W    | 0           | GPIO Pull Up Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 3:2      | GP6DRV   | R/W    | 0           | GPIO Driving Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 1        | GP6SMT   | R/W    | 1           | GPIO SMT Control                         |
+----------+----------+--------+-------------+------------------------------------------+
| 0        | GP6IE    | R/W    | 1           | GPIO Input Enable                        |
+----------+----------+--------+-------------+------------------------------------------+

GPIO_CFGCTL4
--------------
 
**Address：**  0x40000110
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                          | GP9FUNC                                       | RSVD                  | GP9PD     | GP9PU     | GP9DRV                | GP9SMT    | GP9IE     |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                          | GP8FUNC                                       | RSVD                  | GP8PD     | GP8PU     | GP8DRV                | GP8SMT    | GP8IE     |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                              |
+----------+----------+--------+-------------+------------------------------------------+
| 31:28    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 27:24    | GP9FUNC  | R/W    | 4'HB        | GPIO Function Select (Default : SWGPIO ) |
+----------+----------+--------+-------------+------------------------------------------+
| 23:22    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 21       | GP9PD    | R/W    | 0           | GPIO Pull Down Control                   |
+----------+----------+--------+-------------+------------------------------------------+
| 20       | GP9PU    | R/W    | 0           | GPIO Pull Up Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 19:18    | GP9DRV   | R/W    | 0           | GPIO Driving Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 17       | GP9SMT   | R/W    | 1           | GPIO SMT Control                         |
+----------+----------+--------+-------------+------------------------------------------+
| 16       | GP9IE    | R/W    | 1           | GPIO Input Enable                        |
+----------+----------+--------+-------------+------------------------------------------+
| 15:12    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 11:8     | GP8FUNC  | R/W    | 4'HB        | GPIO Function Select (Default : SWGPIO ) |
+----------+----------+--------+-------------+------------------------------------------+
| 7:6      | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 5        | GP8PD    | R/W    | 0           | GPIO Pull Down Control                   |
+----------+----------+--------+-------------+------------------------------------------+
| 4        | GP8PU    | R/W    | 0           | GPIO Pull Up Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 3:2      | GP8DRV   | R/W    | 0           | GPIO Driving Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 1        | GP8SMT   | R/W    | 1           | GPIO SMT Control                         |
+----------+----------+--------+-------------+------------------------------------------+
| 0        | GP8IE    | R/W    | 1           | GPIO Input Enable                        |
+----------+----------+--------+-------------+------------------------------------------+

GPIO_CFGCTL5
--------------
 
**Address：**  0x40000114
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                          | GP11FUNC                                      | RSVD                  | GP11PD    | GP11PU    | GP11DRV               | GP11SMT   | GP11IE    |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                          | GP10FUNC                                      | RSVD                  | GP10PD    | GP10PU    | GP10DRV               | GP10SMT   | GP10IE    |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                              |
+----------+----------+--------+-------------+------------------------------------------+
| 31:28    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 27:24    | GP11FUNC | R/W    | 4'HE        | GPIO Function Select (Default : JTAG )   |
+----------+----------+--------+-------------+------------------------------------------+
| 23:22    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 21       | GP11PD   | R/W    | 0           | GPIO Pull Down Control                   |
+----------+----------+--------+-------------+------------------------------------------+
| 20       | GP11PU   | R/W    | 0           | GPIO Pull Up Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 19:18    | GP11DRV  | R/W    | 0           | GPIO Driving Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 17       | GP11SMT  | R/W    | 1           | GPIO SMT Control                         |
+----------+----------+--------+-------------+------------------------------------------+
| 16       | GP11IE   | R/W    | 1           | GPIO Input Enable                        |
+----------+----------+--------+-------------+------------------------------------------+
| 15:12    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 11:8     | GP10FUNC | R/W    | 4'HB        | GPIO Function Select (Default : SWGPIO ) |
+----------+----------+--------+-------------+------------------------------------------+
| 7:6      | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 5        | GP10PD   | R/W    | 0           | GPIO Pull Down Control                   |
+----------+----------+--------+-------------+------------------------------------------+
| 4        | GP10PU   | R/W    | 0           | GPIO Pull Up Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 3:2      | GP10DRV  | R/W    | 0           | GPIO Driving Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 1        | GP10SMT  | R/W    | 1           | GPIO SMT Control                         |
+----------+----------+--------+-------------+------------------------------------------+
| 0        | GP10IE   | R/W    | 1           | GPIO Input Enable                        |
+----------+----------+--------+-------------+------------------------------------------+

GPIO_CFGCTL6
--------------
 
**Address：**  0x40000118
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                          | GP13FUNC                                      | RSVD                  | GP13PD    | GP13PU    | GP13DRV               | GP13SMT   | GP13IE    |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                          | GP12FUNC                                      | RSVD                  | GP12PD    | GP12PU    | GP12DRV               | GP12SMT   | GP12IE    |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                              |
+----------+----------+--------+-------------+------------------------------------------+
| 31:28    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 27:24    | GP13FUNC | R/W    | 4'HB        | GPIO Function Select (Default : SWGPIO ) |
+----------+----------+--------+-------------+------------------------------------------+
| 23:22    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 21       | GP13PD   | R/W    | 0           | GPIO Pull Down Control                   |
+----------+----------+--------+-------------+------------------------------------------+
| 20       | GP13PU   | R/W    | 0           | GPIO Pull Up Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 19:18    | GP13DRV  | R/W    | 0           | GPIO Driving Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 17       | GP13SMT  | R/W    | 1           | GPIO SMT Control                         |
+----------+----------+--------+-------------+------------------------------------------+
| 16       | GP13IE   | R/W    | 1           | GPIO Input Enable                        |
+----------+----------+--------+-------------+------------------------------------------+
| 15:12    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 11:8     | GP12FUNC | R/W    | 4'HE        | GPIO Function Select (Default : JTAG )   |
+----------+----------+--------+-------------+------------------------------------------+
| 7:6      | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 5        | GP12PD   | R/W    | 0           | GPIO Pull Down Control                   |
+----------+----------+--------+-------------+------------------------------------------+
| 4        | GP12PU   | R/W    | 0           | GPIO Pull Up Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 3:2      | GP12DRV  | R/W    | 0           | GPIO Driving Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 1        | GP12SMT  | R/W    | 1           | GPIO SMT Control                         |
+----------+----------+--------+-------------+------------------------------------------+
| 0        | GP12IE   | R/W    | 1           | GPIO Input Enable                        |
+----------+----------+--------+-------------+------------------------------------------+

GPIO_CFGCTL7
--------------
 
**Address：**  0x4000011c
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                          | GP15FUNC                                      | RSVD                  | GP15PD    | GP15PU    | GP15DRV               | GP15SMT   | GP15IE    |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                          | GP14FUNC                                      | RSVD                  | GP14PD    | GP14PU    | GP14DRV               | GP14SMT   | GP14IE    |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                              |
+----------+----------+--------+-------------+------------------------------------------+
| 31:28    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 27:24    | GP15FUNC | R/W    | 4'HB        | GPIO Function Select (Default : SWGPIO ) |
+----------+----------+--------+-------------+------------------------------------------+
| 23:22    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 21       | GP15PD   | R/W    | 0           | GPIO Pull Down Control                   |
+----------+----------+--------+-------------+------------------------------------------+
| 20       | GP15PU   | R/W    | 0           | GPIO Pull Up Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 19:18    | GP15DRV  | R/W    | 0           | GPIO Driving Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 17       | GP15SMT  | R/W    | 1           | GPIO SMT Control                         |
+----------+----------+--------+-------------+------------------------------------------+
| 16       | GP15IE   | R/W    | 1           | GPIO Input Enable                        |
+----------+----------+--------+-------------+------------------------------------------+
| 15:12    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 11:8     | GP14FUNC | R/W    | 4'HE        | GPIO Function Select (Default : JTAG )   |
+----------+----------+--------+-------------+------------------------------------------+
| 7:6      | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 5        | GP14PD   | R/W    | 0           | GPIO Pull Down Control                   |
+----------+----------+--------+-------------+------------------------------------------+
| 4        | GP14PU   | R/W    | 0           | GPIO Pull Up Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 3:2      | GP14DRV  | R/W    | 0           | GPIO Driving Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 1        | GP14SMT  | R/W    | 1           | GPIO SMT Control                         |
+----------+----------+--------+-------------+------------------------------------------+
| 0        | GP14IE   | R/W    | 1           | GPIO Input Enable                        |
+----------+----------+--------+-------------+------------------------------------------+

GPIO_CFGCTL8
--------------
 
**Address：**  0x40000120
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                          | GP17FUNC                                      | RSVD                  | GP17PD    | GP17PU    | GP17DRV               | GP17SMT   | GP17IE    |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                          | GP16FUNC                                      | RSVD                  | GP16PD    | GP16PU    | GP16DRV               | GP16SMT   | GP16IE    |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                              |
+----------+----------+--------+-------------+------------------------------------------+
| 31:28    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 27:24    | GP17FUNC | R/W    | 4'HE        | GPIO Function Select (Default : JTAG )   |
+----------+----------+--------+-------------+------------------------------------------+
| 23:22    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 21       | GP17PD   | R/W    | 0           | GPIO Pull Down Control                   |
+----------+----------+--------+-------------+------------------------------------------+
| 20       | GP17PU   | R/W    | 0           | GPIO Pull Up Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 19:18    | GP17DRV  | R/W    | 0           | GPIO Driving Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 17       | GP17SMT  | R/W    | 1           | GPIO SMT Control                         |
+----------+----------+--------+-------------+------------------------------------------+
| 16       | GP17IE   | R/W    | 1           | GPIO Input Enable                        |
+----------+----------+--------+-------------+------------------------------------------+
| 15:12    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 11:8     | GP16FUNC | R/W    | 4'HB        | GPIO Function Select (Default : SWGPIO ) |
+----------+----------+--------+-------------+------------------------------------------+
| 7:6      | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 5        | GP16PD   | R/W    | 0           | GPIO Pull Down Control                   |
+----------+----------+--------+-------------+------------------------------------------+
| 4        | GP16PU   | R/W    | 0           | GPIO Pull Up Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 3:2      | GP16DRV  | R/W    | 0           | GPIO Driving Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 1        | GP16SMT  | R/W    | 1           | GPIO SMT Control                         |
+----------+----------+--------+-------------+------------------------------------------+
| 0        | GP16IE   | R/W    | 1           | GPIO Input Enable                        |
+----------+----------+--------+-------------+------------------------------------------+

GPIO_CFGCTL9
--------------
 
**Address：**  0x40000124
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                          | GP19FUNC                                      | RSVD                  | GP19PD    | GP19PU    | GP19DRV               | GP19SMT   | GP19IE    |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                          | GP18FUNC                                      | RSVD                  | GP18PD    | GP18PU    | GP18DRV               | GP18SMT   | GP18IE    |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                              |
+----------+----------+--------+-------------+------------------------------------------+
| 31:28    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 27:24    | GP19FUNC | R/W    | 4'HB        | GPIO Function Select (Default : SWGPIO ) |
+----------+----------+--------+-------------+------------------------------------------+
| 23:22    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 21       | GP19PD   | R/W    | 0           | GPIO Pull Down Control                   |
+----------+----------+--------+-------------+------------------------------------------+
| 20       | GP19PU   | R/W    | 0           | GPIO Pull Up Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 19:18    | GP19DRV  | R/W    | 0           | GPIO Driving Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 17       | GP19SMT  | R/W    | 1           | GPIO SMT Control                         |
+----------+----------+--------+-------------+------------------------------------------+
| 16       | GP19IE   | R/W    | 1           | GPIO Input Enable                        |
+----------+----------+--------+-------------+------------------------------------------+
| 15:12    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 11:8     | GP18FUNC | R/W    | 4'HB        | GPIO Function Select (Default : SWGPIO ) |
+----------+----------+--------+-------------+------------------------------------------+
| 7:6      | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 5        | GP18PD   | R/W    | 0           | GPIO Pull Down Control                   |
+----------+----------+--------+-------------+------------------------------------------+
| 4        | GP18PU   | R/W    | 0           | GPIO Pull Up Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 3:2      | GP18DRV  | R/W    | 0           | GPIO Driving Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 1        | GP18SMT  | R/W    | 1           | GPIO SMT Control                         |
+----------+----------+--------+-------------+------------------------------------------+
| 0        | GP18IE   | R/W    | 1           | GPIO Input Enable                        |
+----------+----------+--------+-------------+------------------------------------------+

GPIO_CFGCTL10
---------------
 
**Address：**  0x40000128
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                          | GP21FUNC                                      | RSVD                  | GP21PD    | GP21PU    | GP21DRV               | GP21SMT   | GP21IE    |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                          | GP20FUNC                                      | RSVD                  | GP20PD    | GP20PU    | GP20DRV               | GP20SMT   | GP20IE    |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                              |
+----------+----------+--------+-------------+------------------------------------------+
| 31:28    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 27:24    | GP21FUNC | R/W    | 4'HB        | GPIO Function Select (Default : SWGPIO ) |
+----------+----------+--------+-------------+------------------------------------------+
| 23:22    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 21       | GP21PD   | R/W    | 0           | GPIO Pull Down Control                   |
+----------+----------+--------+-------------+------------------------------------------+
| 20       | GP21PU   | R/W    | 0           | GPIO Pull Up Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 19:18    | GP21DRV  | R/W    | 0           | GPIO Driving Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 17       | GP21SMT  | R/W    | 1           | GPIO SMT Control                         |
+----------+----------+--------+-------------+------------------------------------------+
| 16       | GP21IE   | R/W    | 1           | GPIO Input Enable                        |
+----------+----------+--------+-------------+------------------------------------------+
| 15:12    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 11:8     | GP20FUNC | R/W    | 4'HB        | GPIO Function Select (Default : SWGPIO ) |
+----------+----------+--------+-------------+------------------------------------------+
| 7:6      | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 5        | GP20PD   | R/W    | 0           | GPIO Pull Down Control                   |
+----------+----------+--------+-------------+------------------------------------------+
| 4        | GP20PU   | R/W    | 0           | GPIO Pull Up Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 3:2      | GP20DRV  | R/W    | 0           | GPIO Driving Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 1        | GP20SMT  | R/W    | 1           | GPIO SMT Control                         |
+----------+----------+--------+-------------+------------------------------------------+
| 0        | GP20IE   | R/W    | 1           | GPIO Input Enable                        |
+----------+----------+--------+-------------+------------------------------------------+

GPIO_CFGCTL11
---------------
 
**Address：**  0x4000012c
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                          | GP23FUNC                                      | RSVD                  | GP23PD    | GP23PU    | GP23DRV               | GP23SMT   | GP23IE    |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                          | GP22FUNC                                      | RSVD                  | GP22PD    | GP22PU    | GP22DRV               | GP22SMT   | GP22IE    |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                              |
+----------+----------+--------+-------------+------------------------------------------+
| 31:28    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 27:24    | GP23FUNC | R/W    | 4'HB        | GPIO Function Select (Default : SWGPIO ) |
+----------+----------+--------+-------------+------------------------------------------+
| 23:22    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 21       | GP23PD   | R/W    | 0           | GPIO Pull Down Control                   |
+----------+----------+--------+-------------+------------------------------------------+
| 20       | GP23PU   | R/W    | 0           | GPIO Pull Up Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 19:18    | GP23DRV  | R/W    | 0           | GPIO Driving Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 17       | GP23SMT  | R/W    | 1           | GPIO SMT Control                         |
+----------+----------+--------+-------------+------------------------------------------+
| 16       | GP23IE   | R/W    | 1           | GPIO Input Enable                        |
+----------+----------+--------+-------------+------------------------------------------+
| 15:12    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 11:8     | GP22FUNC | R/W    | 4'HB        | GPIO Function Select (Default : SWGPIO ) |
+----------+----------+--------+-------------+------------------------------------------+
| 7:6      | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 5        | GP22PD   | R/W    | 0           | GPIO Pull Down Control                   |
+----------+----------+--------+-------------+------------------------------------------+
| 4        | GP22PU   | R/W    | 0           | GPIO Pull Up Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 3:2      | GP22DRV  | R/W    | 0           | GPIO Driving Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 1        | GP22SMT  | R/W    | 1           | GPIO SMT Control                         |
+----------+----------+--------+-------------+------------------------------------------+
| 0        | GP22IE   | R/W    | 1           | GPIO Input Enable                        |
+----------+----------+--------+-------------+------------------------------------------+

GPIO_CFGCTL12
---------------
 
**Address：**  0x40000130
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                          | GP25FUNC                                      | RSVD                  | GP25PD    | GP25PU    | GP25DRV               | GP25SMT   | GP25IE    |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                          | GP24FUNC                                      | RSVD                  | GP24PD    | GP24PU    | GP24DRV               | GP24SMT   | GP24IE    |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                              |
+----------+----------+--------+-------------+------------------------------------------+
| 31:28    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 27:24    | GP25FUNC | R/W    | 4'HB        | GPIO Function Select (Default : SWGPIO ) |
+----------+----------+--------+-------------+------------------------------------------+
| 23:22    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 21       | GP25PD   | R/W    | 0           | GPIO Pull Down Control                   |
+----------+----------+--------+-------------+------------------------------------------+
| 20       | GP25PU   | R/W    | 0           | GPIO Pull Up Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 19:18    | GP25DRV  | R/W    | 0           | GPIO Driving Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 17       | GP25SMT  | R/W    | 1           | GPIO SMT Control                         |
+----------+----------+--------+-------------+------------------------------------------+
| 16       | GP25IE   | R/W    | 1           | GPIO Input Enable                        |
+----------+----------+--------+-------------+------------------------------------------+
| 15:12    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 11:8     | GP24FUNC | R/W    | 4'HB        | GPIO Function Select (Default : SWGPIO ) |
+----------+----------+--------+-------------+------------------------------------------+
| 7:6      | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 5        | GP24PD   | R/W    | 1           | GPIO Pull Down Control                   |
+----------+----------+--------+-------------+------------------------------------------+
| 4        | GP24PU   | R/W    | 0           | GPIO Pull Up Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 3:2      | GP24DRV  | R/W    | 0           | GPIO Driving Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 1        | GP24SMT  | R/W    | 1           | GPIO SMT Control                         |
+----------+----------+--------+-------------+------------------------------------------+
| 0        | GP24IE   | R/W    | 1           | GPIO Input Enable                        |
+----------+----------+--------+-------------+------------------------------------------+

GPIO_CFGCTL13
---------------
 
**Address：**  0x40000134
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                          | GP27FUNC                                      | RSVD                  | GP27PD    | GP27PU    | GP27DRV               | GP27SMT   | GP27IE    |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                          | GP26FUNC                                      | RSVD                  | GP26PD    | GP26PU    | GP26DRV               | GP26SMT   | GP26IE    |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+------------------------------------------+
| Bit      | Name     |Type    | Reset       | Description                              |
+----------+----------+--------+-------------+------------------------------------------+
| 31:28    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 27:24    | GP27FUNC | R/W    | 4'HB        | GPIO Function Select (Default : SWGPIO ) |
+----------+----------+--------+-------------+------------------------------------------+
| 23:22    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 21       | GP27PD   | R/W    | 0           | GPIO Pull Down Control                   |
+----------+----------+--------+-------------+------------------------------------------+
| 20       | GP27PU   | R/W    | 0           | GPIO Pull Up Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 19:18    | GP27DRV  | R/W    | 0           | GPIO Driving Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 17       | GP27SMT  | R/W    | 1           | GPIO SMT Control                         |
+----------+----------+--------+-------------+------------------------------------------+
| 16       | GP27IE   | R/W    | 1           | GPIO Input Enable                        |
+----------+----------+--------+-------------+------------------------------------------+
| 15:12    | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 11:8     | GP26FUNC | R/W    | 4'HB        | GPIO Function Select (Default : SWGPIO ) |
+----------+----------+--------+-------------+------------------------------------------+
| 7:6      | RSVD     |        |             |                                          |
+----------+----------+--------+-------------+------------------------------------------+
| 5        | GP26PD   | R/W    | 0           | GPIO Pull Down Control                   |
+----------+----------+--------+-------------+------------------------------------------+
| 4        | GP26PU   | R/W    | 0           | GPIO Pull Up Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 3:2      | GP26DRV  | R/W    | 0           | GPIO Driving Control                     |
+----------+----------+--------+-------------+------------------------------------------+
| 1        | GP26SMT  | R/W    | 1           | GPIO SMT Control                         |
+----------+----------+--------+-------------+------------------------------------------+
| 0        | GP26IE   | R/W    | 1           | GPIO Input Enable                        |
+----------+----------+--------+-------------+------------------------------------------+

GPIO_CFGCTL14
---------------
 
**Address：**  0x40000138
 

+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 31        | 30        | 29        | 28        | 27        | 26        | 25        | 24        | 23        | 22        | 21        | 20        | 19        | 18        | 17        | 16        | 
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                                                                                                                          |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| 15        | 14        | 13        | 12        | 11        | 10        | 9         | 8         | 7         | 6         | 5         | 4         | 3         | 2         | 1         | 0         |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 
| RSVD                                                                                                                  | GP28PD    | GP28PU    | GP28DRV               | GP28SMT   | GP28IE    |
+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+ 

+----------+----------+--------+-------------+------------------------+
| Bit      | Name     |Type    | Reset       | Description            |
+----------+----------+--------+-------------+------------------------+
| 31:6     | RSVD     |        |             |                        |
+----------+----------+--------+-------------+------------------------+
| 5        | GP28PD   | R/W    | 0           | GPIO Pull Down Control |
+----------+----------+--------+-------------+------------------------+
| 4        | GP28PU   | R/W    | 0           | GPIO Pull Up Control   |
+----------+----------+--------+-------------+------------------------+
| 3:2      | GP28DRV  | R/W    | 0           | GPIO Driving Control   |
+----------+----------+--------+-------------+------------------------+
| 1        | GP28SMT  | R/W    | 1           | GPIO SMT Control       |
+----------+----------+--------+-------------+------------------------+
| 0        | GP28IE   | R/W    | 1           | GPIO Input Enable      |
+----------+----------+--------+-------------+------------------------+

