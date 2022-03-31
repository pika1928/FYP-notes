# FYP
## Final Year Project
Title: EV Motor Control using FreeRTOS on an Embedded CPU

<u>Objective1:</u> [[#Portable FreeRTOS FOC program]]. 
<u>Description:</u> Produce a program that implements Field Oriented Control of any BLDC/PMSM motor that can run on any sufficiently capable embedded-CPU running FreeRTOS. 

<u>Objective2:</u> [[#Example ESC PCB]]. 
<u>Description:</u> Produce an exemplar implementation of the project demonstrating an embedded-CPU running FreeRTOS running the FOC program, controlling a model-scale BLDC/PMSM motor.  

## Portable [[FreeRTOS]] [[FOC]] program
Developed using an [[stm32]] nucleo-f446re development board and the STM32CubeIDE.

### Program structure


#### FOC
The FOC algorithm part of the program will be implemented using Space Vector Modulation: 
![[FOC#Space Vector Modulation]]

## Example [[ESC]] PCB
The model-scale example PCB will aim to deliver power on a level suitable for hobby RC vehicles (such as RC cars) or hobby e-vehicles (such as e-bikes, e-scooters, e-skateboards).
This will hopefully serve as a safe, low-power analogy for EVs. 
![[ESC#Model scale ESC]]
