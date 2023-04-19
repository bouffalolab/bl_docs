===========
ADC
===========

Overview
===========
The chip integrates one 12-bit SAR ADC, which supports 12 external analog input signals and several internal analog signals. ADC can work in single conversion mode and multi-channel scanning mode, and the conversion result is 12/14/16-bit LeftJustified mode. ADC has a FIFO with a depth of 32, which supports multiple interrupts and DMA. In addition to measuring ordinary analog signals, ADC can also measure power supply voltage, and detect temperature by measuring the internal/external diode voltage.

Features
===========

- High performance

    + Optional 12-bit/14-bit/16-bit conversion result for output
    + Single channel continuous conversion mode up to 2M samples
    + Other conversion modes up to 500K sample rate
    + Optional reference voltage of 2.0V/3.2V
    + Transfer of conversion result to memory through DMA
    + Supports single-channel single conversion, single-channel continuous conversion, multi-channel single conversion, multi-channel continuous conversion
    + Single-ended and differential input modes
    + Jitter compensation
    + User-defined offset value of conversion result

- Number of analog channels

    * Twelve external analog channels
    * Two internal DAC channels
    * One VBAT/2 channel
    * One TSEN channel

Functional Description
===========================

The block diagram of ADC module is shown as follows.

.. figure:: ../../picture/ADCBasicStruct.svg
   :align: center

   Block Diagram of ADC

The ADC module consists of front-end input channel selector, programmable amplifier, ADC sampling module, data processing module, and FIFO.

The input channel selector is used to select the channel to be sampled, including internal and external analog signals.

The programmable amplifier is used to further process the input signal, and it is set based on the characteristics of the input signal (DC/AC) to get a more accurate conversion value.

The ADC sampling module is the most important functional module, which gets the result of conversion from analog signal to digital signal by successive comparison.

The result of the conversion is 12/14/16 bit, and the data processing module further processes the conversion result, including adding channel information, etc.

The final data will be pushed to the back-end FIFO.

ADC Pins and Internal Signals
-------------------------------

.. table:: ADC internal signal 

    +-----------------+--------------+-------------------------------------------------------------------+
    | Internal Signal |    Type      |        Description                                                |
    +=================+==============+===================================================================+
    |   VBAT/2        |     Input    | Voltage signal divided from the power supply pin                  |
    +-----------------+--------------+-------------------------------------------------------------------+
    |   TSEN          |     Input    | Output voltage of internal temperature sensor                     |
    +-----------------+--------------+-------------------------------------------------------------------+
    |   VREF          |     Input    | Reference voltage of internal analog module                       |
    +-----------------+--------------+-------------------------------------------------------------------+
    | DACOUTA         |     Input    | DAC module output                                                 |
    +-----------------+--------------+-------------------------------------------------------------------+
    | DACOUTB         |     Input    | DAC module output                                                 |
    +-----------------+--------------+-------------------------------------------------------------------+


.. table:: ADC external pin 

    +--------------+-----------------+-------------------------------------------+
    | External Pin | Signal Type     |        Signal Description                 |
    +==============+=================+===========================================+
    |   VDDA       |     Input       | Positive supply voltage of analog module  |
    +--------------+-----------------+-------------------------------------------+
    |   VSSA       |     Input       | Analog module ground                      |
    +--------------+-----------------+-------------------------------------------+
    | ADC_CHX      |     Input       | Analog input pins, 12 channels            |
    +--------------+-----------------+-------------------------------------------+


ADC Channels
-------------
The selectable channels of ADC sampling include the input signals of external analog pins and the selectable signals inside the chip:

- ADC CH0
- ADC CH1
- ADC CH2
- ADC CH3
- ADC CH4
- ADC CH5
- ADC CH6
- ADC CH7
- ADC CH8
- ADC CH9
- ADC CH10
- ADC CH11
- DAC OUTA
- DAC OUTB
- VBAT/2
- TSEN
- VREF
- GND

It is worth noting that if VBAT/2 or TSEN is selected as the input signal to be sampled, gpadc_vbat_en or gpadc_ts_en shall be set. The ADC module supports single-ended input or differential input. For the former, GND shall be selected as the negative input channel.

ADC Clocks
-------------

The following figure illustrates the working clock source of the ADC module.

.. figure:: ../../picture/ADCClock.svg
   :align: center
   
   ADC Clocks

The clock source of ADC can be the 96M from PLL, XTAL or internal RC32M. The GLB module controls the setting of clock source selection and provides clock frequency division. By default, the clock source of ADC is 96M and the clock frequency division is 2. The clock reaching the ADC module is 32M. 

Inside the ADC module, clock frequency division (div=16 by default) is provided, so the default clock is 2M. Users can adjust the clock source and division factor of ADC according to the actual sampling needs. 

The gpadc_32m_clk_div frequency division register is 6 bits wide (max div=64). The frequency division formula is fout=fsource/(gpadc_32m_clk_div+1). The gpadc_clk_div_ratio frequency division register of 3 bits width is located inside the ADC module. Its frequency division values are defined as follows:

- 3'b000: div=1
- 3'b001: div=4
- 3'b010: div=8
- 3'b011: div=12
- 3'b100: div=16
- 3'b101: div=20
- 3'b110: div=24
- 3'b111: div=32

ADC Conversion Mode
------------------------

ADC supports single channel conversion and scanning conversion modes.

In single channel conversion mode, select the positive input channel via gpadc_pos_sel and the negative input channel via gpadc_neg_sel, set the gpadc_cont_conv_en control bit to 0 to indicate single channel conversion and then set the gpadc_conv_start control bit to start the conversion.

In the scanning conversion mode, the gpadc_cont_conv_en control bit is set to 1. ADC performs conversion one by one according to the number of conversion channels set by the gpadc_scan_length control bit and the channel sequence set by gpadc_reg_scn_posX (X = 1, 2) and gpadc_reg_scn_negX (X = 1, 2) register sets. The conversion result is automatically pushed into the ADC FIFO. The channels set by the gpadc_reg_scn_posX (X = 1, 2) and gpadc_reg_scn_negX (X = 1, 2) register sets can be the same, which means that the user can sample a channel multiple times for conversion.

The conversion result of ADC is usually pushed into FIFO. Users need to set the FIFO data receiving threshold interrupt according to the actual number of conversion channels, and this threshold interrupt is used as the ADC Conversion Complete Interrupt.

ADC Result
-------------
The register gpadc_raw_data stores the original result of ADC. In the single-ended mode, the valid bit of data is 12 bits, without sign bit. In the differential mode, the most significant bit (MSB) is the sign bit, and the remaining 11 bits represent the conversion result.

The register gpadc_data_out stores the ADC result, which contains the ADC result, sign bit, and channel information. The data format is as follows:

.. table:: Meaning of ADC Conversion Result

    +---------+----+-----+-----+-----+----+-----+-----+-----+----+----+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
    | BitS    | 25 | 24  | 23  | 22  | 21 | 20  | 19  | 18  | 17 | 16 |15|14|13|12|11|10|9 | 8| 7| 6| 5| 4| 3| 2| 1| 0|
    +=========+====+=====+=====+=====+====+=====+=====+=====+====+====+==+==+==+==+==+==+==+==+==+==+==+==+==+==+==+==+
    | meaning |  Positive channel number  |  Negative channel number  |           Conversion result                   |
    +---------+----+-----+-----+-----+----+-----+-----+-----+----+----+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+

In the above table, bit21–bit25 indicates the positive channel number, and bit16–bit20 indicates the negative channel number, while bit0–bit15 indicates the converted value.

The gpadc_res_sel control bit can set the bits of the conversion result to 12 bits, 14 bits, and 16 bits, of which 14 bits and 16 bits are the result of multiple sampling. The optional setting values are as follows:

- 3'b000    12bit 2MS/s, OSR=1 
- 3'b001    14bit 125kS/s, OSR=16
- 3'b010    14bit 31.25kS/s, OSR=64 
- 3'b011    16bit 15.625KS/s, OSR=128
- 3'b100    16bit 7.8125KS/s, OSR=256

In the left-justified ADC conversion result, when 12 bits are selected, bit15–bit4 of the conversion result is valid; when 14 bits are selected, bit15–bit2 is valid; and when 16 bits are selected, bit15–bit0 is valid. Similarly, in the differential mode, the MSB is the sign bit. Namely, when 14 bits are selected, bit15 is the sign bit and bit14 is the MSB, while bit14–bit2 is the conversion result. In the single-ended mode, there is no sign bit. Namely, when 12 bits are selected, bit15–bit4 is the conversion result and bit15 is the MSB.

In practice, ADC results are generally pushed into FIFO, which is especially important in the multi-channel scanning mode. Therefore, users usually obtain conversion results from ADC FIFO. The data format of ADC FIFO is the same as that in the register gpadc_data_out.

ADC Interrupt
-------------
- ADC conversion completion interrupt

  * When the ADC conversion is complete and the result is stored in the FIFO, the interrupt switch is set via gpadc_rdy_mask to select whether to trigger the ADC conversion completion interrupt.

- ADC positive (negative) sampling over-range interrupt

  * When the ADC is over-ranged for positive and negative sampling, set the interrupt switch via gpadc_pos_satur_mask, gpadc_neg_satur_mask to select whether to trigger an interrupt or not, and when an interrupt is generated, query the interrupt status via the gpadc_pos_satur and gpadc_neg_satur registers. The interrupt status can be cleared by setting gpadc_pos_satur_clr and gpadc_neg_satur_clr. This function can be used to determine if the input voltage is abnormal.

ADC FIFO
-------------
The ADC module has a FIFO with a depth of 32 and its data width is 26 bits. When the ADC completes conversion, the result will be automatically pushed into the FIFO. ADC FIFO has the following statuses and interrupt management functions:

- FIFO: full
- FIFO: non-empty
- FIFO Overrun interrupt
- FIFO Underrun interrupt

When an interrupt is generated, the interrupt flag can be cleared by the corresponding clear bit.

ADC FIFO enables users to obtain data through query, interrupt, and DMA modes.

**Query mode**

CPU polls the gpadc_rdy bit: When the control bit is set, it indicates that there are valid data in FIFO. CPU can get the amount of FIFO data through gpadc_fifo_data_count and read out these data from FIFO.

**Interrupt mode**

If gpadc_rdy_mask is set to 0 in CPU, ADC will generate an interrupt when data is pushed into FIFO. In the interrupt function, the user can get the amount of FIFO data through gpadc_fifo_data_count and read out these data from FIFO, and then set gpadc_rdy_clr to clear the interrupt.

**DMA mode**

If the user sets the gpadc_dma_en control bit, DMA can transfer the conversion data to the memory. In the DMA mode, after the data volume threshold is set for ADC FIFO to send DMA requests through gpadc_fifo_thl, when receiving a request, DMA will automatically transfer the specified number of results from FIFO to the corresponding memory according to the user defined parameters.

ADC Setup Process
------------------

**Set ADC clock**

According to the required ADC conversion speed, determine the working clock of ADC, set the ADC clock source and frequency division of the GLB module, and determine the final working clock frequency of the ADC module based on the gpadc_clk_div_ratio.

**Set GPIO according to the channel used**

According to the analog pin used, determine the channel number used, and initialize the corresponding GPIO to analog function. But note that when setting GPIO as analog input, set it as floating input instead of pull-up or pull-down.

**Set the channel to convert**

According to the used analog channel and conversion mode, set the corresponding channel register. For single channel conversion, set the conversion channel information in the registers gpadc_pos_sel and gpadc_neg_sel. In the multi-channel scanning mode, set gpadc_scan_length, gpadc_reg_scn_posX, and gpadc_reg_scn_negX according to the number of channels to be scanned and the scanning sequence.

**Set the data read method**

Based on the data reading mode introduced by ADC FIFO, select the mode used and set the register. If DMA is used, configure one channel of DMA, and assist ADC FIFO in transferring data.

**Start conversion**

Finally, set gpadc_res_sel to select the precision of data conversion result, and then set gpadc_global_en=1 and gpadc_conv_start=1 to start ADC conversion. When conversion is completed, to convert again, it is necessary to set gpadc_conv_start to 0 first and then to 1 to trigger the conversion again.

VBAT Measurement
------------------
The VBAT/2 here measures the voltage of the chip VDD33, not the external lithium battery voltage. If you need to measure the voltage of power supply sources like lithium battery, you can divide the voltage and input it into the GPIO analog channel of ADC. Measuring the voltage of VDD33 can reduce the use of GPIO.

The VBAT/2 voltage measured by the ADC module is divided, and the actual voltage input to the ADC module is half of that of VDD33, namely VBAT/2=VDD33/2. As the voltage is divided, to ensure a high accuracy, it is suggested to select 2.0 V reference voltage for ADC, and enable the single-ended mode. The positive input voltage is set to VBAT/2 and the negative input voltage is set to GND. Gpadc_vbat_en is set to 1. After conversion starts, the corresponding conversion result can be multiplied by 2 to get the voltage of VDD33.

TSEN Measurement
------------------
ADC can measure the voltage of internal or external diode. As the voltage difference of diode is related to temperature, the ambient temperature can be calculated from the measured voltage of diode. So it is called temperature sensor (TSEN).

The measurement principle of TSEN is the curve fitted by the voltage difference (ΔV) with the change of temperature, where ΔV is produced by measuring two different currents on a diode. No matter an external or internal diode is measured, the final output value is related to temperature and can be expressed as Δ(ADC_out)=7.753T+X. As long as we know the voltage, we know the temperature T. X indicates an offset value, which can be used as a standard value. Before actual use, X shall be determined first. The chip manufacturer will measure Δ(ADC_out) at a standard temperature, such as 25°C room temperature, before the chip leaves the factory, to obtain X. In actual use, the user can get the temperature T according to the formula T=[Δ(ADC_out)X]/7.753.

When TSEN is used, it is recommended to set ADC to the 16bits mode, reduce error through multiple sampling, select 2.0 V reference voltage to improve accuracy, and set gpadc_ts_en to 1 to enable the TSEN function.

If internal diode is selected, gpadc_tsext_sel=0. If external diode is selected, gpadc_tsext_sel=1.

The positive input channel is selected according to actual needs, namely TSEN channel for the internal diode or the analog GPIO channel for the external diode. The negative input channel is set to GND.

After finishing the above settings, set gpadc_tsvbe_low=0 to start measurement, and get the measurement result V0. Then set gpadc_tsvbe_low=1 to start measurement, and get the measurement result V1, Δ(ADC_out)=V1-V0. The temperature T is calculated based on the formula T=[Δ(ADC_out)X]/7.753.

.. only:: html

   .. include:: adc_register.rst

.. raw:: latex

   \input{../../en/content/adc}

