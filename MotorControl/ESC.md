# ESC
# Electronic Speed Controller
A device that controls the speed of an electric motor.

In a motor control system, the ESC sits between the DC power source and the electric motor. It generates and controls the signals sent to the motor in order to achieve a desired speed as specified by the user/another system. 

# FYP Model-scale ESC
Where "model scale" means targeting up to 60V 100A, i.e. 6kW.

## Block diagram
The block diagram for a model scale might looks something like this:
![[esc-block-diagram.png]]

The FOC program that such an ESC would run is detailed in [[FOC#Implementation]]: 
![[FOC#^foc-implementation]]

### Power source
#### MCU
Requires 3.3V, regulated either from the batteries/PSU supply the motor or from the USB port when a source is connected. 

##### Switching Buck Regulator from batteries
LM5008A switching buck regulator integrated inside [[DRV8353RS]] gate driver.
Recommended: a 10-47uF bulk capacitor near the input. 
###### LM5008A Component Values using datasheet 
Assuming:
	12-75V input voltage range. 
	150mA output current. 

###### R_FB1 & R_FB2
Vout=2.5*(RFB1+RFB2)/RFB1;
	therefore, for 3.3V out, ==RFB1:RFB2 must be 3.125:1==
###### R_T
Fmax = Vout/(Vinmax\*400ns)
		 = 3.3/(75\*400n) 
		 = 110kHz
RT = Vout/(1.385e-10\*Fmax)
	 = 3.3/(1.385e-10\*110k) 
	 = 216kOhm
Nearest E12 = ==220kOhm==, giving ==Fmax=108kHz==, ==Vinmax=76.1V==
###### L1

###### C3

###### C2 & R3

R3 must be sized so that with the ESR of C2 there results a >25mVp-p ripple at the FB pin. Typically this means 0.5-3Ohms.

###### R_CL

###### D1

###### C1

###### C4

###### C5

###### LM5008A Component Values using WEBENCH
Assuming:
	12-75V input voltage range. 
	3.3V @ 150mA output current. 
	30°C ambient temp. 

WEBENCH generates: 
![[esc-WEBENCH-generated-LM5008A-schematic.svg]]
And simulates the output voltage for load currents of 50mA (sim:3) and 150mA (sim:4):
![[esc-WEBENCH-Vout-50-150mA.png]]
For which both ripple ~35mA between 3.283V and 3.317V. 

##### Low Drop-Out regulator from USB power
<u>AMS1117-3.3:</u> Advanced Monolithic Systems fixed 3.3V LDO:
![[esc-psu-3v3-ldo.png]]
The P-MOS FET acts as reverse polarity protection, which isn't needed for USB, _however_
by using a **pull-down resistor** and input from the batteries: if battery voltage (12-75V) is present, then the P-MOS will be reverse-biased, Vgs will be -ve, and so it wont conduct. 
This way, in the presence of a battery connection, only the LM5008A switching buck regulator will provide 3.3V to the MCU and so no back-feeding from the batteries to the USB cable/supply will occur. 
But in the absence of battery power, the USB cable can be used to power the MCU (for e.g. programming) without powering the DRV8353 gate driver, motor switches, or motor. 

**Cost:** The **pull-down resistor** will parasitically drain 0.12-0.75mA from the batteries. 

##### 3.3VA
Additional filtering for the analogue voltage source. 


#### FETs
large input lead capacitor
trace copper weight (trace tinning?)

### IO
#### STM32 pinout
Configured using CubeMX in CubeIDE:
![[esc-stm32-pinout.png]]

#### USB
[Dubious Creations article tutorial on implementing USB C 2.0](https://dubiouscreations.com/2021/04/06/designing-with-usb-c-lessons-learned/) 
Controlled impedance:

#### CAN bus
[Texas Instruments article on CAN bus standard vs split termination](https://e2e.ti.com/blogs_/b/industrial_strength/posts/the-importance-of-termination-networks-in-can-transceivers)

### Crystal oscillator
Application note from STM about how to select surrounding circuitry for a crystal oscillator for stm32 MCUs: 
[[stm32-oscillator-design-guide-stmicroelectronics.pdf]]

## PCB Signal Integrity considerations
<u>Minimising signal-loop (return path) area</u>
Stack-up:    Signal - GND - GND - Signal
GND between signal layers reduces coupling fields & therefore crosstalk.
Adjacent GND vias by signal vias to provide minimal signal-loop area. 

Power layers close to HF ICs to minimise parasitic inductance (aka better opwer delivery).
Stich equi-potential layers (ie GND layers) together often. 
