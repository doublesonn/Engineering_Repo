
# High Voltage Circuits



### Capacitor Charger Circuit 

LT3751 capacitor charger
Datasheet: https://www.analog.com/media/en/technical-documentation/data-sheets/lt3751.pdf

Initial Calculations
- Vtran, Vcc: 24V
- Vout: 500V
- C_out: 4.7uF
- N: 1:10
- Estimated efficiency: 75%
- Peak Charge Current: 25A
Charge time:
$$ t_{charge} = \frac{2E}{\eta V_{trans}I_{Peak}} = \frac{2*0.5}{0.75*24V*25A} = 2.2mS
$$

Primary Inductance: 
$$
L_{Primary} = \frac{Vout}{I_{peak}N} = \frac{500}{25 * 0.75} = ~6\mu H
$$

Transformer Selection
Coilcraft GA3459-BL
https://www.coilcraft.com/getmedia/eb987d97-6bee-4e60-8a64-a66d4e6e447b/ga3459.pdf

The GA3459-BL has an inductance of 5uH, and peak current limit of 20A. 

Typical magnetics derating - 70-80% of saturation current to leave margin for saturation current and temperature drift.

Using this rule of thumb, I will limit the peak to 15A. 

Charge time becomes:

$$ t_{charge} = \frac{2E}{\eta V_{trans}I_{Peak}} = \frac{2*0.5}{0.75*24V*15A} = 3.7mS
$$

To limit my peak current, the Rsense resistor will be sized as

$$
R_{sense} = \frac{0.107V}{I_{peak}} = \frac{0.107}{15A} = 7.13m\Omega
$$

MOSFET selection
Need to identify
- Vds rating
- Id rating
- Gate charge requirement for LT3751 internal driver

Vds Rating
Flyback voltage seen at the drain will be roughly
$$
Vd = V_{in}+\frac{(V_{out})}{N} = 24V + \frac{500}{10} = 74V
$$
Using 60% derating factor to find a minimum Vds rating
$$
V_{ds_{derated}} = \frac{74V}{0.6} = 123.3V
$$
The closest commonly available Vds rating is around 200V

Id rating
Peak current target: 15A

**Gate charge** 
The LT3751 has an integrated driver. To ensure fast turn-on, the gate charge of the selected MOSFET needs to be considered. From the LT3751 data sheet, the gate rise time & fall time are characterized with a 3.3nF capacitive load. 

This puts gate charge required to meet datasheet spec at:

$$
Q_{GateCharge} = C_{Gate}V_{GS} = 3.3nF*10V = 33nC
$$

Infineon - BSC320N20NS3GATMA1
https://www.mouser.com/datasheet/3/70/1/Infineon-BSC320N20NS3%20G-DataSheet-v02_04-EN.pdf

Vds: 200V
Id: 36A
Qgate: 29nC max
Rdson @ 10V: 27-32mOhm

Diode Selection
GB02SLT12-214 
https://www.genesicsemi.com/sic-schottky-mps/GB02SLT12-214/GB02SLT12-214.pdf


![[Screenshot 2026-03-21 at 1.49.30 PM 1.png]]


### Safety Discharge Circuit

To safely discharge the high voltage capacitor as quickly as possible, an MCU controllable discharge circuit will be placed in parallel with the capacitor bank.

The strategy selected to accomplish this will be to use a MOSFET driven by an MCU control signal through an isolated gate driver. 

**MOSFET selection**
The drain to source breakdown voltage should be high enough to handle the 500V capacitor bank voltage. I will use a 60% derating factor to select a Vds rating with margin to ensure safe operating range. 

Minimum Vds rating: 833V

IPB95R310PFD7
datasheet: https://www.mouser.com/datasheet/3/70/1/Infineon_IPB95R310PFD7_DataSheet_v02_01_EN.pdf
Vds breakdown: 950V
I_pulse: 62A

A current limiting resistor will need to keep the discharge current under 62A
I will derate the drain current by 60% and use this as an operating max value.
I_pulse_operating = 37.2A

A minimum current limiting resistor value of 13.4ohms will limit the peak discharge to 37.2A, I will select a real component value of 15ohms. 

The capacitor bank discharge current will be 33.3A with a discharge time near
$$
t_{discharge} = 5*RC = 5*15*4.7{\mu F} = 352.5{\mu S}
$$

Isolated Gate Driver Selection: Si823H9BC-IS

![[Screenshot 2026-03-21 at 2.03.54 PM.png]]

![[Screenshot 2026-03-21 at 2.05.47 PM.png]]


### Pulsed Energy Stage

To activate the high power pulse stage, I will trigger a silicon controlled rectifier using a control signal from the MCU through an isolated gate driver. 

SCR selection
The calculate 


For simplicity, I will recycle the gate driver selected for the capacitor discharge circuit. The Si823H9BC-IS is capable of sourcing 4A. 

More detail regarding this stage is written up in [[3 Pulsed Energy Stage Design]]



Complete High Voltage Schematic 

![[Screenshot 2026-03-21 at 2.13.35 PM.png]]

# Low Voltage Circuits

The low-voltage subsystem is responsible for providing stable auxiliary power to sensing, control, and signal-conditioning circuits throughout the system. Both isolated and non-isolated domains will be implemented, each incorporating regulated 12 V and 5 V supply rails to support mixed-signal and digital components.

In addition to power delivery, this subsystem will enable system observability through integrated sensing. Voltage, current, and magnetic field measurements will be implemented to support system characterization, diagnostics, and performance evaluation during development and testing.



