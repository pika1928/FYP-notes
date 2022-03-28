# STM32
Family of arm-based microprocessors that all run the stm32-[[HAL]] 

This means you can write C/C++ programs using the stm32-HAL and they will be portable to the whole family of MCUs.
Can then pick an appropriately powerful, cost-effective MCU for implementation. 
It is likely that for a given stm32-MCU there will be a drop-in replacement with higher-specs in an equivalent package. 

## Programming
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
