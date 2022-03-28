# FOC
## Field Oriented Control
The Field Oriented Control scheme keeps the magnetic field of the stator 90° ahead of the rotor's magnetic field. 
This maintains a maximum torque between the stator and the rotor, efficiently converting input power to the stator into output power in the rotor.

### FOC algorithm
A FOC program must: 
1. Measure the angular position of the rotor
2. Compute the desired stator field based upon the rotor position
3. Control the 3-phase motor currents to create the desired stator field

A good explanation of FOC as an algorithm: [MATLAB: Understanding Field-Oriented Control](https://youtu.be/YPD1_rcXBIE). 
(Part of [this series](https://youtube.com/playlist?list=PLn8PRpmsu08qL-EG3DRMtRyokpXQJyhp7) on BLDC/PMSM motor control)

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

