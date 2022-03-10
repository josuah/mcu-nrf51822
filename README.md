Nordic nRF51822
===============
<https://infocenter.nordicsemi.com/pdf/nRF51_RM_v3.0.pdf>

Microcontroller implementing [ARM Cortex-M0+](/arm-cortex-m0-plus/)
ISA, used by the [micro:bit](/arm-microbit-v1.5/).

![Nordic nRF51822 in micro:bit v1.5](/image/nordic-nrf51822.jpg)

SoftDevice S110
---------------
<https://infocenter.nordicsemi.com/pdf/S110_SDS_v2.0.pdf>

Nordic provides compiled code with drivers and MBR+bootloader to
put at addresses specified in the `.hex` file.

It mostly provide the Bluetooth and radio stack, along with some
common low-level code for passing execution to the Application.

The specification is mostly about Bluetooth, see Chapter 10, 11 and
Appendix A for how SoftDevice works.

The SoftDevice is optional as it is built on top of
[CMSIS](/arm-cortex-m0-plus/) which might be used directly
instead.

Interrupts and booting
----------------------
The execution flow is entirely driven by interrupts, including for
starting the first bytes of the Application (aka, Operating System)
from bootup.

Each interrupt is caught by the System Vector Table at 0x00000000,
which eventually forward it to the Bootloader or SoftDevice, which
eventually forward it to the Application.

Forwarding goes through loading the address stored at the Interrupt's
position of the Vector Table, then branching at that address.

Booting means having the Reset Handler interrupt triggered, chaining
execution up to the application code ISR.

Memory Addresses
----------------
See the reference manual (link at the top) for complete
overview.

As usual everything is exposed on the same memory address space:

* Non Volatile Memory (NVM)
* Random Access Memory (RAM)
* Peripherals communication registers
* Special control registers.

All with some protection mode available for reading and/or writing.
The Serial Wire Debugger (SWD) always has access to everything.

### 0x00000000 - 0x200000000 - Code Memory

#### 0x00000000 - Master Boot Record (MBR) and its Vector Table

Receives interrupts from the hardware.

The MBR only update the bootloader and forward interrupts. It is
not changed by upgrades making the MPU safe from bricking.

Interrupts are either caught by the MBR, forwarded to the Bootloader
or forwarded to the SoftDevice.

#### 0x00001000 - SoftDevice and its Vector Table

Receives interrupts from the MBR.

The SoftDevice may transfer execution to the application code at
if setup with sd_softdevice_vector_table_base_set(APP_CODE_BASE).

Interrupts are either caught or forwarded to the Application.

#### BOOTLOADERADDR - Bootloader and its Vector Table

Receives interrupts from the MBR or SoftDeivce.

It may be absent, in which case interrupts are only sent to the
SoftDeivce.

Interrupts are either caught or forwarded to the Application.

#### 0x00018000 - APP_CODE_BASE - Application and its Vector Table

Receives interrupts from the Bootloader or SoftDevice.

"Application" means operating system here, as it is higher level
than hardware and SoftDevice.

The address 0x00018000 is the one showing in the SoftDevice
[specification](/arm-nordic-nrf51822/S110_SDS_v2.0.pdf)#Ch11.2.

#### 0x10000000 - Factory Information Configuration (FICR)

Registers with various internal states and information about this MCU.

#### 0x10001000 - User Information Configuration (UICR)

Registers for controlling various states of this MCU.

Programming the nRF81522
------------------------
Two APIs are available:

* The SoftDevice, through the
  [SDK](https://developer.nordicsemi.com/nRF5_SDK/doc/) that offers
  headers to control the SoftDevice:
  <https://infocenter.nordicsemi.com/?topic=/struct_sdk/struct/sdk_nrf5_latest.html>

* The [CMSIS](https://developer.arm.com/documentation/dui0662/latest/)
  specified by ARM and implemented by the Nordic MCUs.

Links
-----
* <https://os.mbed.com/questions/79603/Nordic-nRF-IRQ-Handlers-and-Bootloader/>
  forum discussion about the bootloader

* <https://hackaday.io/project/28180/logs>
  Tinkering of a sibling Nordic MCU.

* <https://github.com/hlnd/nrf51-uart-bootloader> small bootloader
  in C and assembly

