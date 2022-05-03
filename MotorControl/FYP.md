# FYP
## Final Year Project
Title: EV Motor Control using FreeRTOS on an Embedded CPU

<u>Objective1:</u> [[#Portable FreeRTOS FOC program]]. 
<u>Description:</u> Produce a program that implements Field Oriented Control of any BLDC/PMSM motor that can run on any sufficiently capable embedded-CPU running FreeRTOS. 

<u>Objective2:</u> [[#Example ESC PCB]]. 
<u>Description:</u> Produce an exemplar implementation of the project demonstrating an embedded-CPU running FreeRTOS running the FOC program, controlling a model-scale BLDC/PMSM motor.  


## To Do
##### ESC PCB
- [x] DRV8353 schematic wiring 
- [x] ==STM32 output configuration== 
- [x] USB connector 
	- [x] Power MCU over USB without batteries? 
- [x] CANBUS connector 
- [x] Encoder connector 
	- [ ] ~~Hall-effect sensor connector?~~ 
- [x] Output pads/holes/terminals for motor phase connections
- [ ] Double check DRV schematic 
	- [ ] VREF reference voltage for current sense 

- [ ] MOSFET selection 
- [ ] feedback resistor selection 
- [ ] LM5008A component selection 
- [ ] DRV8353 component selection 
- [ ] Remaining components selection 
- [ ] PCB layout 
- [ ] PCB order file generation 
- [ ] PCB file checking 
- [ ] PCB ordering 

##### FOC Program
- [ ] Configure FreeRTOS on stm32 
- [ ] Outline files & functions according to plan [[FOC#Implementation]] 
- [ ] Write code for each file & function 
- [ ] Test o.o 


---
## Portable [[FreeRTOS]] [[FOC]] program
---
Developed using an [[stm32]] nucleo-f446re development board and the STM32CubeIDE.

### Program structure

#### FOC
The FOC algorithm part of the program will be implemented using Space Vector Modulation: 
![[FOC#Space Vector Modulation]]


---
## Example [[ESC]] PCB
---
The model-scale example PCB will aim to deliver power on a level suitable for hobby RC vehicles (such as RC cars) or hobby e-vehicles (such as e-bikes, e-scooters, e-skateboards).
This will hopefully serve as a safe, low-power analogy for EVs. 
![[ESC#FYP Model-scale ESC]]

### Component selection for ESC PCB
Components for the PCB will be selected based upon their ability to complete a given function, their availability and their price.
The following is a list of key components and the reasons they were chosen:

`Component:` MCU
`Part chosen:` stm32f405rgt6
`Reasons:` very similar feature set to the stm32f446ret6 in the nucleo-f446re used for [[FOC]] program development but unlike the f446, the f405 is more widely stocked and therefore available for order both individually and through SMD-assembly services at PCB prototype facilities. 

`Component:` PSU for MCU
<u>Switched DC-DC converter for power from batteries</u>
`Part chosen:` integrated LM5008A inside DRV8353RS gate driver
`Reasons:` reduces PCB footprint and complexity.  
<u>Low Dropout Regulator for power from USB</u>
`Part chosen:` Advanced Monolithic Systems AMS1117-3.3
`Reasons:` Small footprint, simple, power efficiency not a concern over USB, cheap, very well stocked. 

`Component:` 3.3V Fuse 
`Part chosen:`  BOURNS MF-MSMF010-2
`Reasons:` Pretty in gold. 300mA as 3.3V line (MCU) shouldn't draw half of that yet can survive that. 

`Component:` Reverse polarity protection 
`Part chosen:` 
`Reasons:` 

`Component:` Ferrite bead
`Part chosen:` Sunlord GZ2012D601TF
`Reasons:` 500mA therefore shouldn't blow before fuse trips. 600ohm @ 100MHz: datasheet page10 row2 col2.  

`Component:` High Speed External Crystal Oscillator for MCU 
`Part chosen:` Yangxing Tech X322516MLB4SI
`Reasons:` 16MHz, super inexpensive, very well stocked. 

`Component:` Status LED
`Part chosen:` Hubei KENTO Elec C2290
`Reasons:` 0603 "White" LED, 3V drop, super cheap, very well stocked. 

`Component:` Power LED
`Part chosen:` Hubei KENTO Elec KT-0603R
`Reasons:` 0603 Red LED, 2V drop, super cheap, very well stocked. 

`Component:` Gate driver for MOSFETs
`Part chosen:` Texas Instruments [[DRV8353RS]]
`Reasons:` The TI drv835x is a MOSFET gate driver aimed at 3-phase motor control applications. It can be spec'd to include three current shunt amplifiers (drv835**3**rs) to measure the current on each phase as well as to include a DC-DC buck converter (drv8353**r**s) that can source 350mA @ 2.5-75V, ideal for powering the MCU.
These optional integrated additional features remove the need for additional ICs and circuitry, greatly reducing the complexity of the ESC PCB design. This decreases the time necessary for design, validation and testing and increasing the likelihood of success. 

`Component:` USB C receptacle
`Part chosen:`  Korean Hroparts Elec TYPE-C-31-M-12
`Reasons:` SMD solder-able by JLCPCB, USB 2.0 spec so simpler to implement, cheap and very well stocked. 

`Component:` ESD IO protection for MCU
`Part chosen:` STM USBLC6-2SC6
`Reasons:` Designed specifically for USB ESD protection, very well stocked. 

`Component:` CAN Bus transceiver 
`Part chosen:` Maxim Integrated MAX33041EASA+
`Reasons:` Small SOIC-8 package, 3.3V, in stock. 


`Component:`  Vgs protection diodes
`Part chosen:` 
`Reasons:` LL-34 package is aka Mini-MELF: [KiCad forum](https://forum.kicad.info/t/ll-34-footprint/13146/2). 

`Component:` 
`Part chosen:` 
`Reasons:` 

`Component:` 
`Part chosen:` 
`Reasons:` 

`Component:` 
`Part chosen:` 
`Reasons:` 