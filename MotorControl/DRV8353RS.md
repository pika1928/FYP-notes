# DRV8353RS
## Datasheet
![[drv835x_datasheet.pdf]]

## (DRV835)**3RS** Block diagram
![[drv835x_datasheet.pdf#page=29]]

## Notes
### PWM control modes
<u>6x PWM mode:</u> 2 PWM signals for each half-bridge, 1 for H, 1 for L, if both H & L are high the output switches are made low ie high-Z (rather than high as commanded).   
<u>3x PWM mode:</u> 1 PWM signal for each half-bridge. "Simplified" control scheme that doesn't look too useful. 
<u>1x PWM mode:</u> for dumb controllers, drv835x just cycles through trapezoidal states with PWM signal controlling frequency (speed). 
<u>Independent PWM mode:</u> individual control signals for each switch, 1 for H, 1 for L. This bypasses the drv835x's dead-time handshake meaning it is up to the controller to prevent shoot-through (short circuit from VDD->H->L->GND). 

### Device interface modes (SPI vs HW)
#### SPI
Pins: SCLK (clock), SDI (data in), SDO (data out), and nSCS (chip select: active-low).

#### HW (Hardware)
Pins: 
- GAIN (current shunt amplifier gain), 
- IDRIVE (gate drive-current strength), 
- MODE (PWM control mode), 
- VDS (switch drain-source over-voltage threshold)

### Supporting circuitry
#### VCP Voltage Charge Pump for H gates
p34: X5R or X7R 1uF 16V capacitor between VDRAIN and VCP pins, 
and X5R or X7R 47nF (VDRAIN-rated)V capacitor between CPH and CPL pins. 


