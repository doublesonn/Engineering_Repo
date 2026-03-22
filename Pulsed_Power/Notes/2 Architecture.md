
![[PulsedPowerDetailedBlockDiagramV1.drawio 2.pdf]]
The plan is to use a flyback charger stage to charge the cap bank to 500V
using the LT3751

An MCU will be used to monitor the charge voltage, and once we're at 500V, it will disable the charger circuit and isolate the cap bank from the charger using an SCR

The MCU will then activate an SCR between the Cap bank and the coil to dump the cap bank energy onto the coil and produce a magnetic field.

An internal, concentric coil will be used to measure the magnetic field produced by the primary coil, as well as simulate a fusion event that pushed an opposing magnetic field back onto the primary coil. 

A recovery circuit will direct this energy into a secondary capacitive bank to be measured. 