# FreeRTOS
Free, open-source Real Time Operating System
Maintained by Amazon Web Services
[Support for 40+ architectures](https://www.freertos.org/RTOS_ports.html)

## Threads
Threads have an Id (handle) and a set of OS attributes: 
```C 
/*Private variables*/

osThreadId_t customThreadHandle;

const osThreadAttr_t customThread_attributes = {
  .name = "customThread",
  .stack_size = 128 * 4,
  .priority = (osPriority_t) osPriorityNormal,
};
```
Threads must be instantiated:
```C
int main(void) 
{
	osKernelInitialize();
	
	customThreadHandle = osThreadNew(customThreadFunction, NULL, &customThread_attributes);
	
	osKernelStart();
}
```
Threads should then infinitely loop inside their custom functions: 
```C
void customThreadFunction(void *argument)
{
  for(;;)
  {
	  // do stuff
	  osDelay(1); // Wait 1ms
  }
  osThreadTerminate(NULL);  
}
```


## [[DMA]]
![[DMA#FreeRTOS implementation]]


## Mutexes
[Old video from 2018](https://youtu.be/Ds1xcyn-vEU) showing how to implement mutexes (or semaphores..?) using CubeIDE on an [[stm32]]:
At the top of `main.c` define a mutex: 
```C 
/*Private variables*/
osMutexId mutexHandle
```
Then inside a thread loop:
```C
for(;;) 
{
	xSemaphoreTake(mutexHandle, portMAX_DELAY);
	// Critical region (do thing, like write to UART)
	xSemaphoreGive(mutexHandle);
}
```

## Thread-to-thread Messages
The same video as above in [[#Mutexes]] also shows how to send messages between threads:
Inside the senders thread loop:
```C
for(;;) 
{
	xTaskNotify(threadHandle, // recipient thread handle
				0x01,         // "message" (bits to set/clear)
				eSetBits      // set "message" bits
				);
}
```
Then inside the receivers thread loop:
```C
uint32_t messageValue;
for(;;)
{
	xTaskNofityWait(pdFALSE, // don't clear flags on entry
					0xFF,    // when clear, clear all flags
					&messageValue, // destination for "message"
					portMAX_DELAY // wait max time for message
					);
	if((messageValue & 0x01) != 0x00)
	{
		// do thing if "message" is received
	}			
}
```
