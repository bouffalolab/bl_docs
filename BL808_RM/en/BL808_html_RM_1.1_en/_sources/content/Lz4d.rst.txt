===========
LZ4D
===========

Overview
=====
LZ4 is a lossless data compression algorithm that features extremely fast decompression speed and average compression ratio. LZ4D is a functional module for decompression of data blocks compressed using LZ4. LZ4D contains a status and control register set used to set the start address of the input compressed data, the output address of the decompressed data, and the decompression status and optional interrupt configuration.

Features
=========
- Supports data blocks compressed using LZ4

- Supports decompression of data blocks compressed at different levels

- Supports decompression of multiple compressed data blocks

- Event flags for decompression completion and errors

- Generates a corresponding interrupt when an event occurs

Functional Description
===========

The module gets the compressed data blocks from the source address through bus access, decompresses them using the internal algorithm, and writes the decompressed data to the specified destination address. During decompression, the module updates relevant registers and sets flags for events including successful decompression and decompression errors. In turn, under these event statuses, the module can request interrupts from the CPU.

Clock
============
LZ4D Module requires one work clock.

Programming Flow
===========

Data Decompression
-----------
- Turn on the module's clock

- Disable the module, write the address of the raw data to be decompressed (excluding compressed file header) to the start address register for source data

- Prepare a sufficient contiguous memory to hold the decompressed data, with a recommended length aligned on 4-byte boundary. Write the address of this memory block to the start address register for destination data

- If interrupts are required, install an interrupt handler function and set the corresponding interrupt enable bit

- In the control register, the module will start decompression once enabled

- If a "complete interrupt" is enabled, then wait for the "complete interrupt"

- If no interrupt is enabled, read the status register and then wait for the completion of decompression

.. only:: html

   .. include:: lz4_register.rst

.. raw:: latex

   \input{../../en/content/lz4}