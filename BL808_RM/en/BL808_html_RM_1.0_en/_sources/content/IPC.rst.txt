===========
IPC
===========

Overview
=====
The Interprocessor Communication (IPC) is a communication mechanism between multi-core processors.
BL808 has three heterogeneous cores, and any two cores can communicate with each other through IPC.

Features
=========
- Each core has an independent 32-bit IPC channel
- Any two cores can communicate with each other

Functional Description
===========
Basic Principle
-----------
Each core has six IPC registers: IPCx\_TRI, IPCx\_STS, IPCx\_ACK, IPCx\_IEN, IPCx\_IDIS and IPCx\_ISTS. The length of each register is 32-bit, and each bit corresponds to one channel of IPC. The cores M0, LP and D0 correspond to IPC0, IPC1 and IPC2 respectively. When one core needs to issue a notification to another, it only needs to write "1" to the corresponding channel of IPCx\_TRI of the receiving core. At this time, the corresponding channel of IPCx\_STS of the receiving core is set to "1". If the interrupt of the corresponding IPC channel of the receiving core is also enabled, an interrupt will be received, indicating that the receiving core receives the notification from the other core.

Operating Process
---------
For example, the core x sends a notification to the core Y, and the channel z is used. The operation flow is as follows:

- The core y enables the interrupt of channel z. That is, the core y performs operation IPCy\_IEN = 1\<\<z
- The core x writes 1 to the bit z in the register IPCy\_TRI. That is, core x performs operation IPCy\_TRI = 1\<\<z
- Upon receiving the interrupt, the core y responds to the core x. That is, the core y performs operation IPCy\_ACK = 1\<\<z
- The core y performs the notified application layer related operation

The operation flow is as follows:

.. figure:: ../../picture/IPCFlow.svg
   :align: center

   IPC Flow

