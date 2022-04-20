# DRV8353RS
## Datasheet
![[drv835x_datasheet.pdf]]

## (DRV835)**3RS** Block diagram
![[drv835x_datasheet.pdf#page=29]]

---
## Notes
### PWM control modes
<u>6x PWM mode:</u> 2 PWM signals for each half-bridge, 1 for H, 1 for L, if both H & L are high the output switches are made low ie high-Z (rather than high as commanded).   
<u>3x PWM mode:</u> 1 PWM signal for each half-bridge. "Simplified" control scheme that doesn't look too useful. 
<u>1x PWM mode:</u> for dumb controllers, drv835x just cycles through trapezoidal states with PWM signal controlling frequency (speed). 
<u>Independent PWM mode:</u> individual control signals for each switch, 1 for H, 1 for L. This bypasses the drv835x's dead-time handshake meaning it is up to the controller to prevent shoot-through (short circuit from VDD->H->L->GND). 

### Device interface modes (SPI vs HW)
#### SPI
Pins: SCLK (clock), SDI (data in), SDO (data out), and nSCS (chip select: active-low).
![[#Programming over SPI]]

#### HW (Hardware)
Pins: 
- GAIN (current shunt amplifier gain), 
- IDRIVE (gate drive-current strength), 
- MODE (PWM control mode), 
- VDS (switch drain-source over-voltage threshold)

### Supporting circuitry
For summary see Layout Guidelines [[drv835x_datasheet.pdf#page=78]].

#### VCP Voltage Charge Pump for H gates
p34: X5R or X7R 1uF 16V capacitor between VDRAIN and VCP pins, 
and X5R or X7R 47nF (VDRAIN-rated)V capacitor between CPH and CPL pins. 

#### VGLS Voltage Linear Regulator for L gates
p34: X5R or X7R 1uF 16V capacitor between VGLS and GND.

#### VM Gate Driver Power Supply
![[#^VMgatedriverpowersupply]]
p77: 0.1uF VM-rated capacitor between VM and GND. 
		+ bulk-capacitor 
![[#^VMBulk]]

#### DVDD 5V 10mA linear regulator
p38: X5R or X7R 1uF 6.3V capacitor between DVDD and DGND/GND. ^DVDDlinearregulatorsupply

#### LM5008A Buck Switching Regulator
p46: points to LM5008A datasheet.
p48: 0.47uF capacitor between VCC and GND.
In HV applications a 7.5-14V voltage can be applied to VCC (using a diode) to turn off the internal regulator, reducing internal power dissipation.
p49: R_T between V_IN and R_T/SD pins must be sized according to `R_T ≥ Vin * Ton / 0.1385n`
![[#^RTon]]
![[#^RCLToff]]
p50: 0.01uF capacitor between BST and SW pins. 

### Implementation details 
#### VM Gate Driver Power Supply
p34: can be supplied from HV motor supply voltage *or from a more efficient LV 'gate driver supply' switching/linear regulator to improve device efficiency.* 
The integrated LM5008A buck regulator in the **R** skews can perform this task.  ^VMgatedriverpowersupply

#### IDRIVE and TDRIVE
##### IDRIVE
Adjustable gate-drive current to control switches' V_DS slew rates which are critical in EMI, dV/dt gate turn-on (important for shoot-through), and parasitics in the half-bridges (induced by switching voltage transients).
By changing the gate current in the Q_GD / Miller charging region, the slew rate of the external MOSFETs can be altered.
IDRIVE can be configured for 50mA-1A for source and 100mA-2A for sink gate currents. 
After turn-on/turn-off, IHOLD of 25mA is then used for the gate current, increasing efficiency. 

##### TDRIVE 
Automatic gate-drive state machine that:
- inserts dead-time to prevent shoot-through, 
- prevents gate turn-on due to parasitic dV/dt, 
- and detects MOSFET gate faults.
By measuring MOSFET V_GS, the switching between H/L can be automatically done without a fixed dead-time, this mean dead-time is automatically and effectively adjusted for temperature drift and MOSFET parameter variation. 
In addition, `t_DEAD` can be added on top of the automatic dead-time. 

#### MOSFET Over-Current Protection / Short Circuit
If the MOSFET voltage V_DS is under V_VDS_OCP for t_OCP then the OCP protection trips. MOSFET V_DS is measured as depicted on page 38 figure 35.

#### DVDD 5V 10mA linear regulator
![[#^DVDDlinearregulatorsupply]]
If load current exceeds 10mA, DVDD behaves as a 10mA constant-current source (voltage drops). 
**Power Dissipated** by DRV835x DVDD:  P = (V_VM-V_DVDD)\*I_DVDD.

#### Current-Shunt Amplifiers (drv835**3**xx)
p41: Bi-directional current
  Equation for current through R_SENSE resistors.
  Gain of 5, 10, 20 or 40. 
p43: Uni-directional current

Auto Calibration occurs on boot for which 50us should be allowed before measurements taken. 
<u>N.B. Auto Calibration sets the amplifiers to max gain.</u>

p45: MOSFET V_DS sense mode: can be used by setting CSA_FET via SPI and can be used to measure the Voltage drop across the Low-side MOSFET's R_DS(on), allowing the <u>half-bridge current to be calculated</u>. 

#### LM5008A Buck DC-DC Regulator
Requires a minimum output voltage ripple of 25-50mV in order to function.
If this is unacceptable, the output can be taken from between the output resistor and capacitor, but load regulation will be "slightly degraded".

The output voltage is determined by the resistor potential divider, the feedback pin FB, is compared against an internal 2.5V.
Over-Voltage is triggered when FB rises above 2.875V.

R_T determines the On-Time of the buck switching regulator, at maximum Vin, R_T must give a T_ON of >400ns. ^RTon

The LM5008A can be disabled by connecting the R_T/SD pin to GND.

R_CL determines the off-time if an Over-Current occurs (Iout > 0.51A). ^RCLToff

125°C max Junction temperature. 
At 165°C the LM5008A resets in low-power mode where the buck-switch is disabled. 

#### Gate Driver Protection Circuits
##### Under-Voltage Lock-Out (UVLO)
V_M and V_DRAIN UVLO disables the MOSFETs.
V_CP and V_GLS UVLO also disables the MOSFETs, but this behaviour can be disabled by setting DIS_GDUV via SPI. 

##### Over-Current Protection (OCP)
###### MOSFETs:
V_DS OCP threshold set through VDS_LVL SPI register, 
t_OCP_DEG deglitch set through OCP_DEG SPI register, 
SPI register OCP_MODE can operate either: latched shutdown, automatic retry, report only, disabled (see page 51). 
In Cycle-By-Cycle mode (default), MOSEF OCP faults will be cleared on every next rising edge PWM input. 
OCP_ACT SPI register changes OCP actions between effecting relevant half-briges (0: individual) or all half-bridges (1: linked).

###### V_SENSE:
If V_SP > V_SEN_OCP for > t_OCP_DEG,  SEN_OCP event occurs and OCP_MODE action is taken.
V_SEN_OCP threshold set through SEN_LVL SPI register,
t_OCP_DEG deglitch set through OCP_DEG SPI register,, 
SPI register OCP_MODE can operate either: latched shutdown, automatic retry, report only, disabled (see page 51-52). 

##### Gate Driver Fault
If GHx or GLx do not change after t_DRIVE of being driven, all MOSFETs are disabled and a fault is reported (FAULT, GDF and VGS bits corresponding to GHx and GLx). Operation resumes following clearing the CLR_FLT bit via SPI.
GDF can be disabled by setting DIS_GDF_UVLO via SPI. 
GDFs are indicative of too low I_DRIVE or t_DRIVE settings, or a Gate-Source short. 

##### OCP Soft Shutdown
I_DRIVEN is reduced 7 settings / to minimum value in the event of OCP to prevent large V spikes across MOSFET Drain-Source.

##### Over-Temperature (Thermal) Warning (OTW)
OTW bit set if die temp > T_OTW. OTW cleared if temp falls < hysteresis point.

##### Over-Temperature (Thermal) Shutdown (OTSD)
T_OTSD, if exceeded MOSFETs disabled, charge pump disabled, FAULT reorted, TSD bit set. 
Send CLR_FLT over SPI to clear TSD.

##### FAULT Table on p53
[[drv835x_datasheet.pdf#page=53]]

#### Device Functional Modes
##### Gate Driver
<u>Sleep Mode:</u>  ENABLE pin low for > t_SLEEP.  To wake ENABLE high for > t_WAKE. 
<u>Operating Mode:</u>  ENABLE high and V_M > V_UVLO. 
<u>Fault Reset:</u>  set CLR_FLT via SPI or issue reset pulse to ENABLE bit (t_RST). 

##### Buck Regulator
<u>Shutdown Mode:</u> V_SD < 0.7V (LDO and switching regulator turn off). 
<u>Active Mode:</u> VCC > UV threshold. 
	Discontinuous Conduction Mode (DCM) when load current < half peak-to-peak inductor ripple

#### Programming over SPI
[[drv835x_datasheet.pdf#page=54]]
SPI Data Input: 16 bit word: 5bit command (1b R/W ,4b address) + 11bits data. 
SPI Data Output: 16 bit: 5bit null, 11bits register-data. 

##### Register Map
[[drv835x_datasheet.pdf#page=56]]
p57: Fault Status Register 1 (read only)
p58: Fault Status Register 2 (read only)
p59: Driver Control Register (read/write)
p60: Gate Drive HS Register (read/write)  (High Side)
p61: Gate Drive LS Register (read/write)   (Low Side)
p62: OCP Control Register (read/write)     (Over Current Protection)
p63: CSA Control Register (read/write)     (Current Sense Amplification)
p64: CSA Control Register Cont. (read/write)

### Application and Implementation
Single supply, 3-phase BLDC motor drive with per-half-bridge current sense:
![[drv835x_datasheet.pdf#page=66]]


### Power Supply Recommendations
p78: Bulk Capacitor between V_M and GND of at least 10uF. ^VMBulk

To minimise parasitic inductance and allow large current delivery from the bulk capacitor use: 
- Thick traces on the high-current path around MOSFETs. 
- Many vias on traces transferring across PCB layers. 

### Layout Recommendations
Use dedicated traces to connect the VDRAIN and SLx pins to the Drains and Sources of respective MOSFETs to improve V_DS sensing and OCP detection. 

Minimise gate driver loop area 
	High Side: GHx -> HS-Gate -> HS-Source -> SHx
	Low Side: GLx -> LS-Gate ->LS-Source -> SPx/SLx

Buck-Regulator Layout Guidelines (p78):
	![[drv835x_datasheet.pdf#page=78]]

