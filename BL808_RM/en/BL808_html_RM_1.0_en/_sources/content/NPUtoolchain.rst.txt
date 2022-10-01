===============
NPU toolchain
===============

GUI Flow chart
=====

.. image:: ../../picture/NPUGUIflowchart.png
   :width: 400
   :align: center

Network Structure
=====
| To display model structure via Netron on GUI, you have to choose a platform 
| and a config file that you want to display.

| Platform: 
| Darknet, Tensorflow, Caffe, Mxnet, ONNX


.. image:: ../../picture/NPUnetworkstructure.png
   :width: 400
   :align: center

ONNX Converter
===============
To convert your platform into ONNX, you have to choose a platform and name the ONNX file.

| Platform: 
| Tensorflow, Caffe, Mxnet

| (DarkNet, Tensorflow Lite is supported directly.)


.. image:: ../../picture/NPUONNXconverter.png
   :width: 400
   :align: center

| After you press “convert” on ONNX Converter, the screen will show two more buttons
| which you can choose to “display” it on GUI or “download” it. 


.. image:: ../../picture/NPUONNXconverter2.png
   :width: 400
   :align: center

Optimizer
==========
The optimizer section allows you to quantize float to fixed points.

:strong:`“Optimize”`: spend more time, high accuracy

:strong:`“Fast Optimize”`: spend less time, more accuracy loss

| Platform: 
| Darknet, Tensorflow, ONNX

| (BLAI NPU also support optimizer format on Tensorflow Lite)

.. image:: ../../picture/NPUoptimizer.png
   :width: 400
   :align: center

Generator & Simulator
=====
:strong:`“Sim&Gen”` will predict the image with 8 bits quantized weights after you have done the Optimizer section, and generate BLAI NPU instruction

:strong:`“Float Sim”` will predict the image with 32 bits unquantized weights.
 
| Platform: 
| Darknet, Tensorflow, ONNX

| (BLAI NPU also support optimizer format on Tensorflow Lite)

.. image:: ../../picture/NPUgensim.png
   :width: 400
   :align: center   