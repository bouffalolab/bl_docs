==========
KeyScan
==========

KYS introduction
=======================
The KYS (Key Scan) module is used to scan the matrix keyboard to obtain key values. It periodically scans the keyboard connection pins and generates an interrupt as soon as it finds a key press.

KYS main features
=====================
- Configurable number of rows and columns, up to 8 rows*20 columns of matrix keyboard
- Store up to four key values
- Support key interrupt

KYS function description
==============================
Configurable number of rows and columns
-------------------------------------------
The number of rows and columns of the matrix keyboard can be configured through the bits <ROW_NUM> and <COL_NUM> of the register KS_CTRL, and the configuration value is the actual value minus one. The maximum number of rows supports 8 rows, and the maximum number of columns supports 20 columns.

GPIO selection
-----------------
Since the button scan is fixed from the GPIO pins with the functions of ROW_0 and COL_0, the row pins need to be selected sequentially from the GPIO with the function of ROW_0, that is, from any one of GPIO0/GPIO8/GPIO16/GPIO24. The column pins need to be selected in order from the GPIO with the function of COL_0, that is, from any one of GPIO0/GPIO20.

Key value
-----------
The key value will be stored in the register KEYCODE_VALUE, every 8 bits is a key value, and the first key value will be stored in the lowest 8 bits. When the bit corresponding to the serial number in the register KEYCODE_CLR is set to one, the corresponding key value and interrupt flag bit will be cleared. The row number corresponding to the key value = key value% total number of rows, the column number corresponding to the key value = key value / total number of rows.

Interrupt
------------
When a key press is detected, the interrupt flag bit will be set and an interrupt will be generated at the same time.

.. only:: html

   .. include:: kys_register.rst

.. raw:: latex

   \input{../../en/content/kys}