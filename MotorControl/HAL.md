# HAL
## Hardware Abstraction Layer
A Hardware Abstraction Layer is a library that is used to mask the underlying hardware being used to perform a task/algorithm.
This means code written using a HAL can run on any MCU that implements that HAL, making it highly portable.

## stm32-HAL
Each series of STM32 microprocessors (eg stm32f4) have a HAL that enables program-compatibility across the whole series. 
Additionally, the HAL functions are largely compatible between series', enabling high inter-compatibility between them.
This makes the stm32 platform appealing on a cost-to-future-expandability basis. 

Documentation for the stm32f4 series of microcontrollers:
! [[stm32f4-hal-and-ll-drivers-documentation.pdf]]
