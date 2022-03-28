# DMA
## Direct Memory Access
Stream data straight from a peripheral (eg ADC) to memory without having to pass it through the CPU. 
This removes the CPU bottleneck and allows for much faster continuous data streaming.


## [[stm32]]-HAL implementation
[Digi-Key youtube tutorial on stm32 ADCs with DMA](https://youtu.be/EsZLgqhqfO0)
In  the ==System Core== category, enable DMA. 
Add a `request` for the peripheral from which to stream data (eg ADC1), 
use Normal or Circular `Mode` to fill or loop-over the destination memory:
![[stm32-dma-configuration.png]]
In this case, `Increment Address` is enabled for memory since the destination memory will be multiple registers, but disabled for peripheral since the ADC has just 1 register. 
Half Word `Data Width` is used since the ADC is 12-bit and a Half-Word is 16-bits.
This will create a DMA handle:
```C
/* Private variables */
DMA_HandleTypeDef hdma_adc1;
```

In the peripheral, enable continuous mode and DMA, 
eg for ADC1 ==enable== `Continuous Conversion Mode` and `DMA Continuous Requests`. Also set the `Clock Prescaler` to as small a value as possible for minimal clock divisions (aka AFastAP):
![[stm32-dma-adc-configuration.png]]
We need a buffer for DMA to pipe ADC values into:
```C
/* Private define */
/* USER CODE BEGIN PD */
#define ADC_DMA_BUF_LEN 4096  // Buffer size for DMA ADC->MEM
⋮
/* Private variables */
/* USER CODE BEGIN PV */
uint16_t adc_dma_buf[ADC_DMA_BUF_LEN]; // buffer for ADC reading to be DMAd into
```
In `main()`, the DMA can be initiated for ADC -> MEM with the stm32-HAL:
```C
HAL_ADC_Start_DMA(&hadc1, (uint32_t*)adc_dma_buf, ADC_DMA_BUF_LEN);
```
The stm32-HAL will then call two functions, triggered when the buffer is half, and fully filled. This means the MEM buffer can be used as a [[DoubleBuffer(Ping-pongBuffer)]]
To use them, they must be defined in `main.c`:
```C
/* USER CODE BEGIN 4 */

// Called when buffer is half-filled
void HAL_ADC_ConvHalfCpltCallback(ADC_HandleTypeDef* hadc)
{
	// code to run when adc_dma_buf is half full
}

// Called when buffer is fully-filled
void HAL_ADC_ConvCpltCallback(ADC_HandleTypeDef* hadc)
{
	// code to run when adc_dma_buf is full
}
```


## [[FreeRTOS]] implementation


