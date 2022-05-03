# STM32
Family of arm-based microprocessors that all run the stm32-[[HAL]] 

This means you can write C/C++ programs using the stm32-HAL and they will be portable to the whole family of MCUs.
Can then pick an appropriately powerful, cost-effective MCU for implementation. 
It is likely that for a given stm32-MCU there will be a drop-in replacement with higher-specs in an equivalent package. 

## Programming
---
### Read ADC
#### Single conversion
[Digi-Key youtube tutorial on stm32 ADCs](https://youtu.be/EsZLgqhqfO0)
In Pinout & Configuration set pin (eg `PA0`) to `ADC_X`, 
under Analog, make sure `ADC_X` has the channel enabled corresponding to that pin
![[stm32-read-adc-pinout&configuration.png]]
This will create a handle to the ADC:
```C
/* Private variables */
 ADC_HandleTypeDef hadc1; 
 ```
We need a variable to store ADC readings in `main()`:
```C
int main(void)
{
	/* USER CODE BEGIN 1 */
	uint16_t data; 
```
Then inside the `while()` loop the ADC value can be read using the stm32-HAL:
```C
while (1)
  {
	  // Read ADC1
	  HAL_ADC_Start(&hadc1); 
	  HAL_ADC_PollForConversion(&hadc1, HAL_MAX_DELAY); // Hang MCU while waiting for ADC read
	  data = HAL_ADC_GetValue(&hadc1); // get ADC1 value from ADC1 register
```
#### To print over UART:
Define a buffer in `main`: 
```C
char msg[10];  // buffer for UART transmission
```
In the while `loop`: 
```C
// Print ADC1 value to UART
	  sprintf(msg, "%hu\r\n", data); // format `data` into a string into msg (h=short, u=unsigned decimal int)
	  HAL_UART_Transmit(&huart2, (uint8_t*)msg, strlen(msg), HAL_MAX_DELAY);

	  HAL_Delay(100); // wait 100ms
```
![[stm32-read-adc-uart-output.png]]

#### [[DMA]] continuous stream
![[DMA#stm32 -HAL implementation]]

---
### Timers as counters
[Digi-Key youtube tutorial on stm32 Timers and Timer-Interrupts](https://youtu.be/VfbW6nfG4kw)
#### Pre-scalers
Timer Pre-scalers just divide the CPU Clock `HCLK` by an amount (NB in STM32CubeIDE prescaler is 0-indexed)
![[mcu-timer-prescalers.png]]
Check the chip datasheet to see what timers are available and their functions, eg for the stm32f446 in the nucleo-f446re: 
! [[stm32f446re_datasheet.pdf#page=30]]

#### [[HAL]] clock
In Pinout & Configuration, **System Core > SYS**, `Timebase Source` determines which clock is used for HAL functions (such as `HAL_Delay()`). 

#### Timer hardware functions
In Pinout & Configuration, **Timers > TIMX**, `Channel`s can be activated and configured in various modes:
- ==Input Capture== mode stores the clock value when an even happens on a specified pin.
- ==Output Compare== toggles a pin whenever the timer reaches a certain value. 
- ==PWM Generation== toggles a pin with a specified duty cycle based on the timers value. 
_These modes are carried out in hardware, therefore do not tax the CPU._

#### Reading the timer value
In `main()` CubeIDE will have initialised the timer, but it must still be started:
```C
MX_TIMXX_Init();             // initialise timer
HAL_TIM_Base_Start(&htimXX); // start timer
```
the value of the timer can then be read with
```C
timer_value = __HAL_TIM_GET_COUNTER(&htimXX);
```

#### Triggering an interrupt 
In Pinout & Configuration, **Timers > TIMX > Configuration > Parameter Settings**, `Counter Period` sets the value the timer counts to. 
Along with `Prescaler`, this allows the timer to be set to count a specific amount of time. 

For interrupts to work, in **Timers > TIMX > Configuration > NVIC Settings**, the `update and global interrupt` flag must be ==enabled==. 

In `main()` CubeIDE will have initialised the timer, but it must still be started *with interrupts*:
```C
MX_TIMXX_Init();                // initialise timer
HAL_TIM_Base_Start_IT(&htimXX); // start timer with interrupt
```

The stm32-HAL functions pertaining to timer interrupts can be found in the stm32 HAL documentation for a given MCU series, eg for the stm32f4 line:
! [[stm32f4-hal-and-ll-drivers-documentation.pdf#page=995]]

These call-back functions can be implemented in `main.c`:
```C
/* USER CODE BEGIN 4 */

// Callback function for when timer resets
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
	// Check which timer reset and triggered this callback
	if (htim == &htimXX)
	{
		// code to run when timerXX elapses goes here
	}
}
```

---
### Timers as PWM generators
[STM youtube tutorial on PWM, complementary outputs and deadtime insertion.](https://youtu.be/rDaC2N-33Oo) 

---
## Peripheral connections
---
### SPI 
[Digi-Key youtube tutorial on stm32 SPI](https://youtu.be/eFKeNPJq50g)
In Pinout & Configuration, under ==Connectivity==, select an SPI peripheral interface. 
Full-Duplex `Mode` allows for simultaneous R/W on the MOSI & MISO lines. 
If a hardware chip-select is needed, a nearby `GPIO_Output` pin can be assigned and used manually in code. 
In the ==SPI== `Parameter Settings`, `Data Size` can be set to 8 or 16 bits and under `Clock Parameters`: `Prescaler` can be adjusted to lower the communication speed:
![[stm32-spi-configuration.png]]
A user-friendly label for the "GPIO_Output" pin, such as "SPI1_nSCS", can be set by 'right-clicking' > 'Enter User Label'. 

In `main()` a buffer for SPI can be created:
```C
int main(void)
{
  /* USER CODE BEGIN 1 */
	char spi_buf[20];
```
and the nSCS pin can be set to it's default high state after GPIO initialisation:
```C
int main(void)
{
  ⋮
  /* Initialize all configured peripherals */
  MX_GPIO_Init();
  ⋮
  MX_SPI1_Init();
  /* USER CODE BEGIN 2 */
    // nSCS pin should be default: high
    HAL_GPIO_WritePin(GPIOC, SPI1_nSCS, GPIO_PIN_SET);
```

SPI communication can then be carried out as follows:
```C
	const uint16_t SPI_COMMAND = 0b0000101011110000;
	  
	HAL_GPIO_WritePin(GPIOC, SPI1_nSCS, GPIO_PIN_RESET);
	HAL_SPI_Transmit(&hspi1, (uint8_t *)&SPI_COMMAND, 2, 100);
	HAL_SPI_Receive(&hspi1, (uint8_t *)spi_buf, 2, 100);
	HAL_GPIO_WritePin(GPIOC, SPI1_nSCS, GPIO_PIN_SET);

	// !!!Check return status of SPI_Transmit
```
where `HAL_SPI_Transmit()` requires: 
-	a handle to the SPI peripheral, 
-	the data to be sent, 
-	the number of bytes, 
-	and a timeout duration (ms). 
and `HAL_SPI_Receive()` requires similar except a pointer to a buffer to fill with data read instead of a pointer to a buffer of data to send. 

#### Async SPI with call-back interrupts
The `HAL_SPI_Transmit_IT()` and `HAL_SPI_Receive_IT()` functions can be used instead and the `HAL_SPI_TxCpltCallback()` and `HAL_SPI_RxCpltCallback()` functions implemented to be executed upon SPI transmission/receiving completing. 

---
### SWD / JTAG
In **System Core > SYS**, enable SWD with/without 'Trace' (SWO pin) by selecting `Trace Asynchronous Sw` or `Serial Wire`:
![[stm32-swd-configuration.png]]

---
### USB
#### USB 1.0 "Full Speed"  12Mbps
In **Connectivity**, enable **USB_OTG_FS** as `Host`, `Device`, or `Dual Role`:
![[stm32-usb-fs-configuration.png]]
Then in **Middleware > USB_DEVICE**, a USB class can be configured such as `Communication Device` so that the USB appears as a Virtual Com Port:
![[stm32-usb-fs-device-class.png]]

#### USB 2.0 "High Speed" 480Mbps

---
### CAN bus
2.0A and B up to 1Mbps
Requires external CAN transceiver to modulate signals from 0-3.3V to CANL/CANH that go from 0-VCC/2 and VCC/2-VCC. 

---
### Encoder
[DeepBlue embedded article on using encoders with stm32](https://deepbluembedded.com/stm32-timer-encoder-mode-stm32-rotary-encoder-interfacing/) 
In **Timers > TIMx**, select a timer capable of handling encoder signals (STM32F4xx: TIM 2, 3, 4 & 5) and set the `Combined Channels` setting to `Encoder Mode`: 
![[stm32-encoder-configuration.png]]
This will require a HSE clock/crystal.
CubeIDE will then initialise the timer:
```C
int main(void) {

  /* Initialize all configured peripherals */
  MX_TIM2_Init();
}
```
The encoder must be started:
```C
	HAL_TIM_Encoder_Start(&htim2, TIM_CHANNEL_ALL);
```
Then the timer count will increment/decrement depending upon which direction the encoder turns, and can be read using deconstruction: 
```C
	count = (TIMx->CNT);
```

