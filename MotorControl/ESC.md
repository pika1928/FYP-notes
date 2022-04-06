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
switching buck regulator 
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