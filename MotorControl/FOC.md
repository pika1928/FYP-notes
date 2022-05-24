# FOC

## Field Oriented Control
The Field Oriented Control scheme keeps the magnetic field of the stator 90° ahead of the rotor's magnetic field. 
This maintains a maximum torque between the stator and the rotor, efficiently converting input power to the stator into output power in the rotor.

### FOC algorithm
A FOC program must: 
1. Measure the angular position of the rotor
2. Determine the desired stator field based upon the rotor position
3. Control the 3-phase motor currents to create the desired stator field

A good explanation of FOC as an algorithm: [MATLAB: Understanding Field-Oriented Control](https://youtu.be/YPD1_rcXBIE). 
(Part of [this series](https://youtube.com/playlist?list=PLn8PRpmsu08qL-EG3DRMtRyokpXQJyhp7) on BLDC/PMSM motor control)

#### Clarke & Park Theory
The stator's magnetic field vector can be decomposed into it's ==direct== and ==quadrature== components, where:
- ==direct==(blue) is in-line with the rotor field(grey)
- ==quadrature==(yellow) is perpendicular to the rotor field(grey)
![[foc-stator-field-d-and-q-components.png]]

To maintain a 90° lead on the rotor, the stator's quadrature component wants to be maximised, and the direct component minimised.

The real stator's magnetic field is made up of the 3 components from the 3 phases A, B and C:
![[foc-3phases-sum-to-stator-field.png]]

Controlling these AC phase signal with a PID controller would be difficult.
Instead of doing that, Clarke and Park transforms can be used to express the 3 AC phase signals as the magnitude of the direct and quadrature components of their combined field vector: 
A+B+C = stator field vector -> Clarke + Park transforms -> direct and quadrature components
![[foc-clarke-and-park-transform-to-q-and-d.png]]

Following the Clarke/Park transformation, PI control could be used to maximise the quadrature component and minimise the direct component:  
![[foc-PI-control-of-d-and-q-components.png]]

#### Space Vector Modulation Theory
Space Vector Modulation (SVM) is an implementation method for FOC that replaces the inverse Clarke transform when converting from the ideal stator field into the 3-phase signals to the motor.  
![[foc-svm-block-diagram.png]]

SVM uses PWM to interpolate between the 6 "basic" stator magnetic field vectors (V1-V6) and the 2 "null" stator magnetic field vectors (V0, V7):
![[foc-svm-basic-and-null-vectors.png]]

These vectors are created by the sum of the component A, B and C phases vectors in each of the 8 possible switch configurations: 
	![[foc-svm-basic-vector-abc-components.png]]
	![[foc-svm-basic-vector-abc-formation.png]]
The angle of the Space Vector is controlled by PWMing two adjacent basic vectors, and it's magnitude is controlled by switching between basic and null vectors (null vector V0 or V7 is chosen depending on which requires the least switches to get to, maximising efficiency by minimising switching losses). 




---
## Implementation
[This MathWorks page](https://uk.mathworks.com/solutions/power-electronics-control/field-oriented-control.html) includes a block diagram that shows all the processes involved in implementing FOC using SVM, with an outer current(torque) controller and optional outer velocity controller:
![[foc-implementation-mathworks-block-diagram.png]]

From this and other resources we can create a more detailed block diagram that shows how sensor/sensorless FOC can be implemented:
![[foc-implementation-block-diagram.png]]^foc-implementation


### Clarke and Park transforms
[Jantzen Lee video explaining the Clarke and Park transformations](https://youtu.be/mbJOxqxLkLE)

The ==Clarke transformation== is used to describe the 3-phase stator currents i_A, i_B and i_C as a current vector on the stators αβ plane:
![[foc-clarke-transform.png]]

The ==Park transform== is used to describe the stators current vector on the αβ plane in terms of the rotors DQ plane:
![[foc-park-transform.png]]
iα and iβ components of a current vector: 
![[foc-park-transform-ivector-αβ.png]]
iD and iQ components of the same vector: 
![[foc-park-transform-ivector-dq.png]]

### Space Vector Modulation
[This MathWorks page](https://uk.mathworks.com/solutions/power-electronics-control/space-vector-modulation.html?s_eid=PSM_15028) gives an overview of what a SVM implementation includes. 
[This video](https://youtu.be/oHEVdXucSJs?t=445) from Jantzen Lee explains concisely how to go from inputs to outputs. It is based upon [this lecture](https://www.youtube.com/watch?v=5eQyoVMz1dY&t=767s) from TI which includes a bit more information.
![[foc-svm-pwm-duty-cycles.png]]
![[foc-svm-pwm-t012-to-tabc.png]]
![[foc-svm-pwm-centre-aligned-outputs.png]]


### Observer algorithms for sensor-less position and velocity
Observers are closed loop controllers (error tracking integrators/eliminators) that also make use of a feed-forward loop to estimate the position/velocity/acceleration of a system by observing it's current position and it's commanded position. 
This eliminates the phase-delay of a typical 2nd/3rd order closed-loop controller. 

#### Sliding mode observer
By measuring the Back-EMF induced in the 3 phases, the rotor position can be estimated.
<u>Limitations:</u> SMO only works at medium+ speeds, since at low speeds there is insufficient BEMF to make a good estimation.

Resources:
[2-phase FOC SMO explanation](https://microchip-mplab-harmony.github.io/mc_apps_sam_e7x_s7x_v7x/apps/pmsm_foc_smo_sam_e70/readme.html) with good block diagrams and program flow. 


#### Low-speed Saliency Tracking Observer
By injecting a sinusoidal voltage signal into the direct axis, a sinusoidal current signal will be induced in the quadrature axis.
By measuring the amplitude of the induced current signal in the Q axis you can observe how well aligned the model/observed stator DQ axis is with the real/actual stator DQ axis. 
<u>Limitations:</u> only works for motors with a strong saliency signal, and only works at low speeds. 

Resources:
[TI lecture](https://www.youtube.com/watch?v=bZwLFpXhFbI&t=3760s). 

#### Flux observer


### Motor parameter sensing/estimation
Stator resistance Rs
d and q axis inductances Ld Lq
Back-EMF constant Ke
Motor inertia J
Friction constant F

### FOC tuning

### Field Weakening for over-speed
[MathWorks article on Field-Weakening](https://uk.mathworks.com/help/mcb/gs/field-weakening-control-mtpa-pmsm.html)
Increase maximum motor speed at the expense of torque (at those higher speeds). 
Maybe an extension goal, not necessary for the main project. 