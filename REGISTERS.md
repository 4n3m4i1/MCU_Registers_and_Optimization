# Registers
While working with microcontrollers registers come up often. Whether it is interacting with a peripheral (UART, SPI, i2C/twi, DMA, etc..), other memory spaces (internal EEPROM, ), GPIO, or anything outside of the pure arithmetic/logic of a microprocessor we access these registers to issue controls, set parameters, load/send data, etc..

#### What is a Register
Depending on the frame of reference, the word register may conjure a variety of images in one's head. From the assembly programmer, a register is likely to be first associated with the primary register file within the microprocessor/CPU. For those unfamiliar with programming or computer architectures, a cash register may come to mind.

This document pertains predominantly to the C programmer, who may be working with what they consider registers without knowing what is necessarily under the hood. Specifically I/O registers, perhaps what controls the state of a GPIO pin, or what contains the clock divider value for a peripheral bus clock. These are often memory locations that are tightly coupled to the processor, in the standard SRAM address space (may be different on your platform), and with more coherent processor designs can have special instructions that can optimize register manipulation.

Example: Using a register to set the state of a GPIO pin (AVR)
`DDRD |= (1 << 2);`

Later on an analysis of compiler optimization options will be conducted specifically to see how and when these direct register manipulation instructions are implemented.

#### Header Files
When one first begins to work with microcontrollers they may notice a header file is typically included in their C (or asm) file. This is a device or manufacturer specific header, that may contain the family name of the part being used:

STM32 Example: `#include "stm32f446xx.h"`

or perhaps an architecture specific header:

Atmel (now Microchip) Example: `#include <avr/io.h>`

Regardless of the header style, eventually a specific device header will be utilized by the preprocessor to eventually copy-paste any mention of a recognized register with the dereferenced pointer expression, with the value predefined in the header file.

##Pointers! (and formatting)
Within the aforementioned header files there are often dozens to hundreds of register references, typically with some base address defined, and register names associated with an offset from the defined base address. This may be done as discrete values, or in more modern implementations with struct based approaches.

####STM32 Example:
Taken from `stm32f446xx.h`
- Define a base address, in this case the peripheral address base
`#define PERIPH_BASE           0x40000000UL`
- Define base address of a specific peripheral bus, based on earlier definition
`#define APB2PERIPH_BASE       (PERIPH_BASE + 0x00010000UL)`
- Define base address of a specific peripheral, based on earlier definition
`#define USART1_BASE           (APB2PERIPH_BASE + 0x1000UL)`
- Define the name of peripheral control struct, associate this name with a specific location in memory
`#define USART1              ((USART_TypeDef *) USART1_BASE)`


####AVR Example:
Taken from `iom328p.h` and `avr/sfr_defs.h`
- Define the base address within a wrapper function
`#define DDRD _SFR_IO8(0x0A)`
- We should be interested what `_SFR_IO8()` actually does
`#if __AVR_ARCH__ >= 100`  
`#define __SFR_OFFSET 0x00`
`#else`
`#define __SFR_OFFSET 0x20`
`#endif`
`#define _SFR_IO8(addr) _MMIO_BYTE((addr) + __SFR_OFFSET)`
- Once again, we should look deeper
`#define _MMIO_BYTE(addr)  (*(volatile uint8_t *)(addr)`

#AVR Analysis
Through several layers of abstraction we can very clearly see what is going on, and even better we can easily draw parallels to AVR assembly such that disasseblies can actually be read.
Shown in the example:
- `DDRD` is defined as `(*(volatile uint8_t *)(0x0A + 0x20))`

Breaking this down from the inside out:
- `0x0A + 0x20` Is Equivalent to `Register Address Offset + Peripheral Base Address`
- `(volatile uint8_t *)` Casts the numeric value to an unsigned volatile 8 bit pointer
- `*()` The leading `*` dereferences the pointer, allowing us to assign or read values stored at that pointer's address

####iMX.RT Example:
(will do later)


