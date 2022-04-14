# ESC
## Electronic Speed Controller
A device that controls the speed of an electric motor.

In a motor control system, the ESC sits between the DC power source and the electric motor. It generates and controls the signals sent to the motor in order to achieve a desired speed as specified by the user/another system. 

## FYP Model-scale ESC
Where "model scale" means targeting up to 60V 100A, i.e. 6kW.

### Block diagram
The block diagram for a model scale might looks something like this:
![[esc-block-diagram.png]]

The FOC program that such an ESC would run is detailed in [[FOC#Implementation]]: 
![[FOC#^foc-implementation]]

#### Power source
##### MCU
LM5008A switching buck regulator integrated inside [[DRV8353RS]] gate driver.

analogue voltage source filtering

##### FETs
large input lead capacitor
trace copper weight (trace tinning?)

#### IO
##### USB
electrostatic protection


#### Crystal oscillator
Application note from STM about how to select surrounding circuitry for a crystal oscillator for stm32 MCUs: 
[[stm32-oscillator-design-guide-stmicroelectronics.pdf]]

### PCB Signal Integrity considerations
<u>Minimising signal-loop (return path) area</u>
Stack-up:    Signal - GND - GND - Signal
GND between signal layers reduces coupling fields & therefore crosstalk.
Adjacent GND vias by signal vias to provide minimal signal-loop area. 

Power layers close to HF ICs to minimise parasitic inductance (aka better opwer delivery).
Stich equi-potential layers (ie GND layers) together often. 
