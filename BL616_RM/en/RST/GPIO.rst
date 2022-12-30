===========
GPIO
===========

Overview
=========
Users can connect General Purpose I/O Ports (GPIO) with external hardware devices to control these devices.

Features
=========
- Up to 35 I/O pins
- Each I/O pin supports up to 25 functions
- Each I/O pin can be configured in pull-up, pull-down, or floating mode
- Each I/O pin can be configured as input, output or Hi-Z state mode
- The output mode of each I/O pin has 4 optional drive capabilities
- The input mode of each I/O pin can be set to enable/disable the Schmitt trigger
- Each I/O pin supports 9 external interrupt modes
- FIFO depth: 128 halfwords
- Data can be transferred through DMA from RAM to I/O pins for output


GPIO function list
==================
The <reg_gpio_xx_func_sel> bit of the GPIO_CFGxx (where xx represents the GPIO pin number) register is used to set the multiplexing function of GPIO. The multiplexing function number is shown in the following table:

.. table:: GPIO function list has_header

    +---------------+---------------+
    |    Number     |  Function     |
    +---------------+---------------+
    | 0             | SDH           |
    +---------------+---------------+
    | 1             | SPI0          |
    +---------------+---------------+
    | 2             | FLASH         |
    +---------------+---------------+
    | 3             | I2S0          |
    +---------------+---------------+
    | 4             | PDM           |
    +---------------+---------------+
    | 5             | I2C0          |
    +---------------+---------------+
    | 6             | I2C1          |
    +---------------+---------------+
    | 7             | UART0         |
    +---------------+---------------+
    | 8             | EMAC          |
    +---------------+---------------+
    | 9             | CAM           |
    +---------------+---------------+
    | 10            | ANALOG        |
    +---------------+---------------+
    | 11            | GPIO          |
    +---------------+---------------+
    | 12            | SDIO          |
    +---------------+---------------+
    | 16            | PWM0          |
    +---------------+---------------+
    | 17            | JTAG          |
    +---------------+---------------+
    | 18            | UART1         |
    +---------------+---------------+
    | 19            | PWM1          |
    +---------------+---------------+
    | 20            | SPI1          |
    +---------------+---------------+
    | 21            | I2S1          |
    +---------------+---------------+
    | 22            | DBI_B         |
    +---------------+---------------+
    | 23            | DBI_C         |
    +---------------+---------------+
    | 24            | QSPI          |
    +---------------+---------------+
    | 25            | AUPWM         |
    +---------------+---------------+
    | 31            | CLOCK_OUT     |
    +---------------+---------------+

.. note::
   If the GPIO is set as a peripheral function, just set the <reg_gpio_xx_func_sel> bit of the GPIO_CFGxx register to the corresponding number of the peripheral.
   
GPIO Input Settings
=======================
Set the register GPIO_CFGxx to configure the general interface to the input mode as described below (xx denotes the GPIO pin number):

- Set <reg_gpio_xx_ie> to 1 to enable the GPIO input mode
- Set <reg_gpio_xx_func_sel> to 11 to enter the SWGPIO mode
- In the SWGPIO mode, set to enable/disable the Schmitt trigger through <reg_gpio_xx_smt> for waveform shaping
- Set to enable/disable the internal pull-up and pull-down functions through <reg_gpio_xx_pu> and <reg_gpio_xx_pd>
- Set the type of external interrupt through <reg_gpio_xx_int_mode_set>, and then read the level value of I/O pin through <reg_gpio_xx_i>

GPIO Output Settings
======================
The following four output modes of GPIO can be set through the register GPIO_CFGxx.

Normal Output Mode
-------------
- Set <reg_gpio_xx_oe> to 1 to enable the GPIO output mode
- Set <reg_gpio_xx_func_sel> to 11 to enter the SWGPIO mode
- Set <reg_gpio_xx_mode> to 0 to enable the normal output function of I/O
- Set to enable/disable the internal pull-up and pull-down functions through <reg_gpio_xx_pu> and <reg_gpio_xx_pd>, and then set the level of I/O pin through <reg_gpio_xx_o>

Set/Clear Output Mode
------------------
- Set <reg_gpio_xx_oe> to 1 to enable the GPIO output mode
- Set <reg_gpio_xx_func_sel> to 11 to enter the SWGPIO mode
- Set <reg_gpio_xx_mode> to 1 to enable the Set/Clear output function of I/O
- Set to enable/disable the internal pull-up and pull-down functions through <reg_gpio_xx_pu> and <reg_gpio_xx_pd>

In the Set/Clear output mode, you can set <reg_gpio_xx_set> to 1 to keep the I/O pin at the high level, or set <reg_gpio_xx_clr> to 1 to keep the I/O pin at the low level. If both <reg_gpio_xx_set> and <reg_gpio_xx_clr> are set to 1, the I/O pin is kept at the high level. If both of them are set to 0, the setting does not work.


I/O FIFO
=================
The depth of I/O FIFO is 128 halfwords. The <gpio_tx_fifo_cnt> bit in the register GPIO_CFG143 indicates the current available space of FIFO (128 by default). Every time a value is written into the GPIO_CFG144 register, the value of <gpio_tx_fifo_cnt> will decrease by 1. After it decreases to 0, if a value is continuously written to the register GPIO_CFG144 and <cr_gpio_tx_fer_en> is 1, the error interrupt will be enabled and this interrupt will occur.

When the <cr_gpio_tx_en> bit in the GPIO_CFG142 register is 1, the data of I/O FIFO will be sent to I/O pins one by one, and the value of <gpio_tx_fifo_cnt> will increment. When it is incremented to greater than <cr_gpio_tx_fifo_th> and <cr_gpio_tx_fifo_en> is 1, the FIFO interrupt is enabled and this interrupt will occur.

If the <cr_gpio_dma_tx_en> bit in the register CR_GPIO_CFG143 is 1, DMA is enabled to send data. If <cr_gpio_tx_fifo_th> is less than <gpio_tx_fifo_cnt>, DMA will transfer the data from the preset RAM to the buffer, whereas the interrupt flag <r_gpio_tx_fifo_int> will be cleared automatically.

I/O Interrupt
================
I/O supports various external interrupts. Setting the <reg_gpio_xx_int_mask> in the register GPIO_CFGxx to 0 can enable the external interrupt of the corresponding pin. <reg_gpio_xx_int_mode_set> is used to set the external interrupt type of that pin.

The supported interrupt types are as follows:

- Synchronous Falling Edge Interrupt

  * Based on the f32k_clk clock, the input pin level is sampled once on each rising edge of the clock. If a high level is followed by two low levels, a synchronous falling edge interrupt will be generated at this time
  
- Synchronous Rising Edge Interrupt

  * Based on the f32k_clk clock, the input pin level is sampled once on each rising edge of the clock. If a low level is followed by two high levels, a synchronous rising edge interrupt will be generated at this time

- Synchronous Low Level Interrupt

  * Based on the f32k_clk clock, after detecting a low level, a synchronous low-level interrupt is generated at the rising edge of the third clock
  
- Synchronous High Level Interrupt

  * Based on the f32k_clk clock, after detecting a high level, a synchronous high-level interrupt is generated at the rising edge of the third clock
  
- Synchronous Double Edge Interrupt

  * Based on the f32k_clk clock, if a high level transition to low level (low level transition to high level) is detected, a falling edge (rising edge) event will be generated. After the event is generated, at the third rising edge of the clock, synchronous double edge interrupt will be generated

- Asynchronous Falling Edge Interrupt

  * When a high-to-low transition is detected, an asynchronous falling edge interrupt is triggered immediately

- Asynchronous Rising Edge Interrupt

  * When a low-to-high transition is detected, an asynchronous rising edge interrupt is triggered immediately

- Asynchronous Low Level Interrupt
  
  * Based on the f32k_clk clock, the input pin level is sampled once on each rising edge of the clock. If it is low for 3 consecutive times, an asynchronous low-level interrupt is triggered
  
- Asynchronous High Level Interrupt
  
  * Based on the f32k_clk clock, the input pin level is sampled once on each rising edge of the clock. If it is high for 3 consecutive times, an asynchronous high-level interrupt will be triggered

In the interrupt function, you can obtain the interrupt-generating GPIO state through the <gpio_xx_int_stat> of the register GPIO_CFGxx, and clear the interrupt flag through <reg_gpio_xx_int_clr>.

.. only:: html

   .. include:: glb_register.rst

.. raw:: latex

   \input{../../en/content/glb}