## Watchdog

### Overview
The watchdog is used to transmit a reset signal to reset the entire system within a period after an exception occurs in the system.

### Features
The watchdog has the following features:

-  Provides a 32-bit internal down counter. The count clock source is configurable.
-  Supports the configurable timeout interval, namely, initial count value.
-  Locks registers to avoid any modification to them.
-  Supports the generation of timeout interrupts.
-  Supports the generation of reset signals.
-  Supports the debugging mode.

### Function Principle
The watchdog works based on a 32-bit down counter. 

The initial value is loaded by **WDG_LOAD**. 

When the watchdog clock is enabled, the count value is decremented by 1 on the rising edge of each count clock. When the count value reaches 0, the watchdog generates an interrupt. On the next rising edge of the count clock, the counter reloads the initial value from **WDG_LOAD** and continues to count in decremental mode.

If the count value of the counter reaches 0 for the second time but the CPU does not clear the watchdog interrupt, the watchdog transmits the reset signal **WDG_RSTN** and the counter stops counting.

You can enable or disable the watchdog by configuring **WDG_CONTROL** as required. That is, you can control the watchdog whether to generate interrupts and reset signals.

-  When the interrupt generation function is disabled, the watchdog counter stops counting.
-  When the interrupt generation function is enabled again, the watchdog counter counts starting from the preset value of **WDG_LOAD** instead of the last count value. Before an interrupt is generated, the initial value can be reloaded.

The count clock of the watchdog can be a crystal oscillator clock or a bus clock so different count time ranges are available.

By configuring **WDG_LOCK**, you can disable the operation of writing to the internal registers of the watchdog.

-  Writing ```0x1ACC_E551``` to **WDG_LOCK** to enable the write permission for all the registers of the watchdog.
-  Writing any other values to **WDG_LOCK** to disable the write permission for all the registers of the watchdog except **WDG_LOCK**.

This feature avoids modifications to the watchdog registers by software. Therefore, the watchdog operation is not terminated by mistake by software when an exception occurs.

In debugging mode, the watchdog is disabled automatically to avoid the intervention to the normal debugging.


### Configuring the Frequency of the Count Clock

The system supports two types of watchdog count clocks: 3 MHz crystal oscillator clock and bus clock. The two clocks are selected by configuring SC_CTRL[wdogenov].

The count time **Twdg** of the watchdog is calculated as follows:

**Twdg** = **WDG_LOAD_Value** * ( 1 / **Fclk**)

where:

-  **Twdg** indicates the count time of the watchdog.
-  **WDG_LOAD_Value** indicates the initial count value of the watchdog.
-  **Fclk** indicates the frequency of the watchdog count clock.

The ranges of the count time of the watchdog under different clocks are as follows:

-  When a 3 MHz crystal oscillator clock is selected, the count time ranges from 0s to 1400s.
-  When a bus clock (such as a 100 MHz clock) is selected, the count time ranges from 0s to 42s.

### Initializing the System

The watchdog counter stops counting after the system power-on reset. Before the system is initialized, the watchdog must be initialized and enabled. To initialize the watchdog, perform the following steps:

-  **Step 1:** Write to **WDG_LOAD** to set the initial count value.
-  **Step 2:** Write to **WDG_CONTROL** to enable the interrupt mask function and start the watchdog counter.
-  **Step 3:** Write to **WDG_LOCK** to lock the watchdog to avoid the watchdog settings being modified by the software by mistake.

**----END**

### Processing an Interrupt

After an interrupt is received from the watchdog, the interrupt must be cleared in time and the
initial count value must be reloaded to the watchdog to restart counting. A watchdog interrupt is processed as follows:

-  **Step 1:** Write ```0x1ACC_E551``` to **WDG_LOCK** to unlock the watchdog.
-  **Step 2:** Write to **WDG_INTCLR** to clear the watchdog interrupt and load the initial count value to the watchdog to restart counting.
-  **Step 3:** Write any other values rather than ```0x1ACC_E551``` to **WDG_LOCK** to lock the watchdog.

**----END**

### Disabling the Watchdog
You can control the status of the watchdog by writing ```0``` or ```1``` to **WDG_CONTROL[inten]**.

-  ```0```: disabled
-  ```1```: enabled


## Summary of watchdog registers (base address: ```0x2004_0000```)

| Offset Address       | Register    | Description                      | Total Reset Value       | ACCESS |
|----------------------|-------------|----------------------------------|:-----------------------:|:------:|
| ```0x0000```         | WDG_LOAD    | Initial count value register     | ```0xFFFF_FFFF```       | RW     |
| ```0x0004```         | WDG_VALUE   | Current count value register     | ```0xFFFF_FFFF```       | RO     |
| ```0x0008```         | WDG_CONTROL | Control register                 | ```0x0000_0000```       | RW     |
| ```0x000C```         | WDG_INTCLR  | Interrupt clear register         | -                       | WO     |
| ```0x0010```         | WDG_RIS     | Raw interrupt register           | ```0x0000_0000```       | RO     |
| ```0x0014```         | WDG_MIS     | Masked interrupt status register | ```0x0000_0000```       | RO     |
| ```0x0018–0x0BFC```  | RESERVED    | Reserved                         | -                       |        |
| ```0x0C00```         | WDG_LOCK    | Lock register                    | ```0x0000_0000```       | RW     |


## Register Description

-  **WDG_LOAD** is an initial count value register. It is used to configure the initial count value of the internal counter of the watchdog.
**example:** ```himm 0x20040000 0x0000000F``` (set count value to 15 ticks)
***
-  **WDG_VALUE** is a current count value register It is used to read the current count value of the internal counter of the watchdog.
**example:** ```himm 0x20040004``` (read count value ticks)
***
-  **WDG_CONTROL** is a control register. It is used to enable or disable the watchdog and control the interrupt and reset functions of the watchdog.
	-  (resen) Output enable of the watchdog reset signal.
		-  ```0```: disabled
		-  ```1```: enabled
	-  (inten) Output enable of the watchdog interrupt signal.
		-  ```0```: The counter stops counting, the current count value remains unchanged, and the watchdog is disabled.
		-  ```1```: The counter, interrupt and watchdog are enabled.

    | resen | inten | value           |       ---------------      | resen | inten | value           |
    |:-----:|:-----:|:---------------:|----------------------------|:-----:|:-----:|:---------------:|
    | 0     | 0     | ```0x00```      |       ---------------      | 1     | 0     | ```0x02```      |
    | 0     | 1     | ```0x01```      |       ---------------      | 1     | 1     | ```0x03```      |
    
    **example:** ```himm 0x20040008 0x03``` (reset signal output and watchdog are enabled)


***
-  **WDG_INTCLR** is an interrupt clear register. It is used to clear watchdog interrupts so the watchdog can reload an initial value for counting. This register is write-only. When a value is written to this register, the watchdog interrupts are cleared. No written value is recorded in this register and no default reset value is defined.
**example:** ```himm 0x2004000C 0x01``` (clear interrupt)
***
-  **WDG_RIS** is a raw interrupt status register. It shows the raw interrupt status of the watchdog.
	Status of the raw interrupts of the watchdog. When the count value reaches 0, this bit is set to 1.
		- ```0```: No interrupt is generated.
		- ```1```: An interrupt is generated.
**example:** ```himm 0x20040010``` (get status of interrupt)
***
-  **WDG_MIS** is a masked interrupt status register. It shows the masked interrupt status of the
watchdog.
    Status of the masked interrupts of the watchdog.
		- 0: No interrupt is generated or the interrupt is masked.
		- 1: An interrupt is generated.
**example:** ```himm 0x20040014``` (get masked status of interrupt)
***
-  **WDG_LOCK** is a lock register. It is used to control the write and read permissions for the watchdog registers.
	-  Writing ```0x1ACC_E551``` to this register enables the write permission for all the registers.
	-  Writing other values disables the write permission for all the registers.

    **example:** ```himm 0x20040C00 0x1ACC_E551``` (unlock watchdog)
    **example:** ```himm 0x20040C00 0x01``` (lock watchdog)

	-  When this register is read, the lock status rather than the written value of this register is returned.
		-  ```0x0000_0000```: The write permission is available (unlocked).
		-  ```0x0000_0001```: The write permission is unavailable (locked).

    **example:** ```himm 0x20040C00``` (read lock status)
