# Intel Flex FPGA Evaluation Guide [Draft 1]

This document is designed for beginners in FPGA board evaluation and related works. It covers some basic topics that need to be noticed when getting into implementation in Flex lab on FPGA. 

## Possible Applications of FPGA in this Lab:
1. **Quick Evaluation**
   
   Since many FPGA boards themselves are connected with CPU and GPU, the convension between hardware language at the ports connecting silicon and host is necessary when processing GPU indibidually. Just for this purpose, a FPGA could be applied to convert the **cxl** bus to **PCIe** one to send to the host, then some simple functions could be evaluated on the host. 

    An important gap here is that most already-made RTL system Verilog, which is designed for GPU, cannot be synthesized directly. While the FPGA need those can be synthesized, we must convert the written code manually. 

    The meaning of this application is to simply evaluate some RTL designs right after they are produced by programming the FPGA. It could also present some problems that cannot be found through simulation, before the solid product of ASIC are made, which usuallly take up plenty of time. This helps start the development of related softwares after the hardware design, contributing a lot to synchronizing the development of both software and hardware.  

2. For some network cards, those fixed functions that has relatively lower performance requirement could be realized by FPGA instead of ASIC. 
3. Some acceleration stack could be used to verify the attempt to shift certain techniques from GPU to ASIC. 

## Quick Start Fundementals

### Basic Linux Commands
> Our server is based on ubuntu so here are some commands that are particularly useful in this environment. 

|  Command   | Usage  |
|  ----  | ----  |
| find [directory]  | search the directory for the tag |
| lsusb  | check current usb ports status |
| ps | check the PID of background processes |
| jobs | check the current running jobs |
| unzip | extract zips |
| tar | extract tar.gz files |
| lspci | check the status of current PCIe ports |


It's more delighting if you are familiar with vim. Try "vimtutor". 

### Connect to the Server
1. Download Linux Environmentr as SSH server [MobaXterm](https://mobaxterm.mobatek.net/download.html) 
2. Register with your IP(given)
3. Make several softwares available on your Linux, such as Quartus, VCS...


### Basic Bus Types
> The transactions between te host and FPGA is case-dependent and is crucial in efficiency, so here briefly listed several bus types that are usually applied in our designs. 

#### Low Speed Bus

1.  SPI -- serial peripheral interface

    > SPI, sometimes called a four-wire serial bus, is a synchronous serial communication interface for short distance communication. It's in full-duplex mode which allows communication in both directions at the same time, and a master-slave architecture. 
    + It specifies four logic signals:
   
        **SCLK**: Serial Clock (output from master); 
   
        **MOSI**: Master Out Slave In (data output from master); 
   
        **MISO**: Master In Slave Out (data output from slave); 
   
        **CS/SS**: Chip/Slave Select (often active low, output from master to indicate that data is being sent)
    + It can operate with a single master device and with one or more slave devices. Most slave devices have tri-state output so those don't have should use an external tri-state buffer. 
    + pros: 
      1. push-pull drivers provides good signal integrity and high speed
      2. Not limited to any maximum clock speed, enabling potentially high speed
      3. Complete protocol flexibility for the bits transferred
      4. Extremely simple hardware interfacing
      5. only 4 pins o IC packages
    + cons:
      1. more pins required than $I^2C$ or three-wire varants
      2. No hardware flow control by the slave 
      3. Typically supports only one master device (depends on device's hardware implementation)
      4. No error-checking protocol is defined
      5. SPI does not support hot swapping (dynamically adding nodes).
      6. Interrupts must either be implemented with out-of-band signals or be faked by using periodic polling similarly to USB 1.1 and 2.0.
    + Applications:
      - The board **real estate** savings compared to a parallel I/O bus are significant, and have earned SPI a solid role in **embedded systems**. (imaging how effect it is when some important data to be transmitted get lost!)
      - Simple and efficient especially in single master/single slave mode.
      - USed to talk to a variety of peripherals: Sensors, Control devices, Communications, Memory......
    + Difference with JTAG
      
       > The SPI bus is intended for **high speed, on board initialization** of device peripherals, while the JTAG protocol is intended to **provide reliable test access to the I/O pins** from an off board controller with less precise signal delay and skew parameters. While not strictly a level sensitive interface, the JTAG protocol supports the recovery of both setup and hold violations between JTAG devices by reducing the clock rate or changing the clock's duty cycles. Consequently, the JTAG interface is not intended to support **extremely high data rates**. They are not interchangeable. 

  
2.  $I^2C$ -- Inter-Integrated Circuit
    > It's a synchronous, multi-controller/target, serial communication bus, which is widely used for attaching **lower-speed** peripheral ICs to processors and microcontrollers in **short-distance, intra-board** communication. SMBus is a derived interface from $I^2C$ with  stronger robustness and interoperability. 
    + Design
      - $I^2C$ uses only two bidirectional open-collector or open-drain lines: serial data line **(SDA)** and serial clock line **(SCL)**, pulled up with resistors. Typical voltages used are +5 V or +3.3 V, although systems with other voltages are permitted. (the FPGA I programmed used 1.8 V)
      - The $I^2C$ reference design has a **7-bit address space**, with a rarely used 10-bit extension. Note that the bit rates are quoted for the transfers between controller and target without clock stretching or other hardware overhead. Thus the actual transfer rate of user data is lower than those peak bit rates alone would imply. 
    + Message protocols
      - $I^2C$ defines basic types of transactions, each of which begins with a *START* and ends with a *STOP*. 
      - Pure $I^2C$ systems support arbitrary message structures. However, SMBus is restricted to nine of those structures. PMBus extends SMBus with a Group protocol, allowing multiple such SMBus transactions to be sent in one combined message. The terminating *STOP* indicates when those grouped actions should **take effect**. 
    + Physical layer
      
       At the physical layer, both SCL and SDA lines are an open-drain (MOSFET) or open-collector (BJT) bus design, thus a pull-up resistor is needed for each line， meaning they're active-high. High-speed systems (and some others) may use a current source instead of a resistor to pull-up only SCL or both SCL and SDA, to accommodate higher bus capacitance and enable faster rise times. 
       
       An important consequence of this is that multiple nodes may be driving the lines simultaneously. If any node is driving the line low, it will be low. Nodes that are trying to transmit a logical one (i.e. letting the line float high) can detect this and conclude that another node is active at the same time.
    + Buffering and multiplexing(related to time domain problems)
      
      When there are many $I^2C$ devices in a system, there can be a need to include bus buffers or multiplexers to split large bus segments into smaller ones. Buffers can be used to isolate capacitance on one segment from another and/or allow $I^2C$ to be sent over longer cables or traces. 
    + Applications
      - $I^2C$ is appropriate for peripherals where simplicity and low manufacturing cost are more important than speed.
      - A particular strength of $I^2C$ is the capability of a microcontroller to control a network of device chips with just **two** general-purpose I/O pins and software. 
      - Enable **plug and play** operation via small ROM in SPD, VGA, HDMI, etc.
      - System Management for PC systems through SMBus, such as in PCI and PCIe.
      - Access real-time clock + low speed DACs and ADCs.
      - Changing and read settings in monitors through Display Data Channel. 
      - Comtrolling small LCD or OLED displays. 
      - Turning on and off the power supply of system componets.
    + Limitations
      - **Address Management:** The assignment of target addresses is one weakness of $I^2C$. Seven bits is too few to prevent address collisions between the many thousands of available devices. 
      - **Speed:** $I^2C$ supports a limited range of speeds. 
      - Devices are allowed to **stretch clock cycles** to suit their particular needs, which can starve bandwidth needed by faster devices and increase latencies when talking to other device addresses. 
      - **Potential Fault:** Because $I^2C$ is a shared bus, there is the potential for any device to have a fault and **hang the entire bus**. 

3. UART -- Universal asynchronous receiver-transmitter
    > UART is a computer hardware device for **asynchrinous serial communication** wit configurabkle data format and transmission speeds. Sending data bits one-by-one from LSB to MSB, it frames data by start and stop bits. It's usually an individual (or part of an) integrated circuit (IC) used for serial communications over a computer or peripheral device serial port. See what [Asynchronous serial communication](https://en.wikipedia.org/wiki/Asynchronous_serial_communication) is. 
   + Working Pattern
      > It takes bytes of data and transmits the individual bits in a sequential fashion. The destination end UART re-assenble the bits into complete bytes, where each UART contains a shift register, so that it cna convert between serial and parallel forms. The serial transmission of bits via a single wire saves cost than parallel transmission via multiple wires. 
     - **Data framing:** The idle, no data state is high-voltage, or powered. Each character is framed as a logic low start bit, data bits, possibly a parity bit and one or more stop bits. In most applications the least significant data bit (the one on the left in this diagram) is transmitted first. If the line is held in the logic low condition for longer than a character time, this is a break condition that can be detected by the UART.
     - All operations of the UART hardware are controlled by an internal clock signal which runs at a multiple of the data rate, typically **8 or 16 times** the bit rate. (Sampling happens approximately every 8/16 clock pulses.)
     - Communicating UARTs have no shared timing system apart from the communication signal. Typically, UARTs **resynchronize** their internal clocks on each change of the data line that is not considered a spurious pulse. 
     - **Double Buffering**It is a standard feature for a UART to store the most recent character while receiving the next. Many UARTs have a small first-in, first-out (FIFO) buffer memory between the receiver shift register and the host system interface. This allows the host processor even more time to handle an interrupt from the UART and prevents loss of received data at high rates.
     - Transmitter: Simpler but requires **a transmit FIFO buffer** to deposit multiple characters in a burst into the FIFO rather than have to deposit one character at a time into the shift register. 
   + Application
     - Transmitting and receiving UARTs must be set for **the same bit speed, character length, parity, and stop bits** for proper operation. The receiving UART may detect some mismatched settings and set a "framing error" flag bit for the host system. 
     - Typical serial ports used with personal computers connected to modems use eight data bits, no parity, and one stop bit; for this configuration, the number of ASCII characters per second equals the bit rate divided by 10. (Don't forget the start bit)
     - Bit-bang in low-cost home computers or embedded systems.
     - Also useful in Modems. (Converting digital signals into analog ones.)


4. JTAG -- named after the **Joint Test Action Group** which codified it
    > It is an industry standard for verifying designs and testing printed circuit boards after manufacture. JTAG implements standards for on-chip instrumentation in electronic design automation (EDA) as a complementary tool to digital simulation. It specifies the use of a dedicated **DEBUG** port implementing a serial communications interface without requiring direct external access to the system address and data buses. 

    > What we used in normal programming is the four-wire JTAG communications protocol of JTAG. 
 
   + A JTAG interface is a special interface added to a chip. Depending on the version of JTAG, two, four, or five pins are added. In either case a test probe need only connect to a single "JTAG port" to have access to all chips on a circuit board.
   + These four signals, collectively known as the Test Access Port or TAP, are part of IEEE Std. 1149.1. This standard was developed to provide a technology for testing Printed Circuit Board Assemblies (PCBAs) without needing the level of physical access required for bed-of-nails testing or the amount of custom development needed for functional test. The TAP was designed to interact with new registers that were added to devices to implement this method of testing. Including
    
    **TDI:** Test Data In

    **TDO:** Test Data Out

    **TCK:** Test Clock
    > Since only one data line is available, the protocol is serial. As with any clocked signal, data presented to TDI must be valid for some chip-specific *Setup* time before and *Hold* time after the relevant (here, rising) clock edge. TDO data is valid for some chip-specific time after the falling edge of TCK.
    **TMS:** Test Mode Select

    **TRST:** Test Reset[Optional]
    > If the pin is not available, the test logic can be reset by switching to the reset state synchronously, using TCK and TMS. 

    Very quickly however silicon manufacturers recognised the benefits of using the TAP to access registers offering other functionalities such as debug and programming. The main register added to a device specifically for JTAG testing is called the Boundary Scan Register (BSR). 

   + JTAG Boundary scan: 
  
        Boundary scan cells (see above) can operate in two modes. In their functional mode they have no effect on the operation of the device – this is the mode in which they operate when the board is running normally. In their test mode they disconnect the functional core of the device from the pins. By putting boundary scan cells into test mode they can be used to control the values being driven from an enabled device onto a net and also be used to monitor the value of that net.

        There are two main ways that this boundary scan capability can be used to test a board. The first way, connection testing (see next section) gives good test coverage, particularly for short circuit faults. It is based purely on the JTAG device capabilities, the connections and nets on the board and – in the case of XJTAG – the logic functionality on a board. The second way extends this coverage by using the JTAG enabled devices on a board to communicate with non-JTAG peripheral devices such as DDR RAM and flash.

     1. **A JTAG connection test:** will check that the connections around the JTAG enabled devices on a board are the same as those specified in the design. Where enabled pins are not meant to be connected they are tested for short circuit faults by driving one pin and checking that these values are not read on the other pins. Missing pull resistors and ‘stuck-at’ faults can also be found by a connection test as well as faults involving logic devices whose behaviour can be described in a truth table.
     2. **Non-JTAG enabled device:**  DDR, SDRAM, SRAM, flash, MDIO controlled Ethernet PHYs, SPI and $I^2C$ temperature sensors, real time clocks, ADCs and DACs are just some examples of such devices. The connection test will still provide excellent coverage for short circuit faults on the nets linking these non-JTAG devices to JTAG enabled devices; however it cannot check for open circuit faults at either the JTAG device or the non-JTAG device.

   + Registers: 
     > There are two types of registers associated with boundary scan. Each compliant device has one instruction register and two or more data registers.
     1. **Instruction Register:** The instruction register holds the current instruction. Its content is used by the TAP controller to decide what to do with signals that are received. Most commonly, the content of the instruction register will define to which of the data registers signals should be passed.
     2. **Data Registers:** There are three primary data registers, the Boundary Scan Register (BSR), the BYPASS register and the IDCODES register. Other data registers may be present, but they are not required as part of the JTAG standard.
         - BSR: this is the main testing data register. It is used to move data to and from the I/O pins of a device.
         - BYPASS: this is a single-bit register that passes information from TDI to TDO. It allows other devices in a circuit to be tested with minimal overhead. 
         - IDCODES: this register contains the ID code and revision number for the device. This information allows the device to be linked to its Boundary Scan Description Language (BSDL) file. The file contains details of the Boundary Scan configuration for the device.

   + **TAP Controller:** 
     > The TAP controller, a state machine whose transitions are controlled by the TMS signal, controls the behaviour of the JTAG system. 
     > 
     > All states have two exits, so all transitions can be controlled by the single TMS signal sampled on TCK. The two main paths(exactly the same) allow for setting or retrieving information from either a data register or the instruction register of the device. The data register operated on (e.g. BSR, IDCODES, BYPASS) depends on the value loaded into the instruction register.

    Reference: See figures on first three chapters of [XJTAG](https://www.xjtag.com/about-jtag/what-is-jtag/). 


-----------------------
#### High Speed Bus

1. DDR -- Double data rate
   + In computing, a computer bus operating with double data rate (DDR) transfers data on **both the rising and falling edges** of the clock signal. DDR SDRAM uses double-data-rate signalling only on the data lines. Address and control signals are still sent to the DRAM once per clock cycle (to be precise, on the rising edge of the clock), and timing parameters such as CAS latency are specified in clock cycles. 
   + DDR is used in conjunction with microprocessors to carry data between the central processing unit (CPU) and the north bridge, which is one of the two chips in the core logic chipset. This pathway is called the front-side bus.

2. PCIe -- Peripheral Component Interconnect Express
    > It is a high-speed serial computer expansion bus standard, a high-speed serial replacement if PCI/PCI-X bus. The main difference between them is the bus topology: PCI uses a shared oarallel bus where the host and all devices share a common set of address. While the PCIe is based on a point-to-point topology with separate serial linkc connecting every device to the **root complex** [see figure]. 
    > 
    > ![Example of PCIe topology](https://upload.wikimedia.org/wikipedia/commons/thumb/1/1c/Example_PCI_Express_Topology.svg/420px-Example_PCI_Express_Topology.svg.png)
    > 
    > Furthermore, the older PCI clocking scheme limits the bus clock to the slowest peripheral on the bus (regardless of the devices involved in the bus transaction). In contrast, a PCI Express bus link supports full-duplex communication between any two endpoints, with no inherent limitation on concurrent access across multiple endpoints.

   + Main Feature:

     - Interconnect: 
     
       PCI Express devices communicate via a logical connection called an interconnect or link. A link is a point-to-point communication channel between two PCI Express ports allowing both of them to send and receive ordinary PCI requests (configuration, I/O or memory read/write) and interrupts. 

    - Lane -> width
    
        A lane is composed of two differential signaling pairs, with one pair for receiving data and the other for transmitting. Thus, each lane is composed of four wires or signal traces. Each lane is used as a full-duplex byte stream, transporting data packets in eight-bit "byte" format simultaneously in both directions between endpoints of a link. Denoted as "x" prefix. Eg, x8, x16 in common use. 

     - Serial Bus -- pro
     
       A serial interface does not exhibit timing skew because there is only one differential signal in each direction within each lane, and there is no external clock signal since clocking information is embedded within the serial signal itself. **Multichannel serial design increases flexibility with its ability to allocate fewer lanes for slower devices.**

   + **Hardware Protocol:**
     > The terms are borrowed from the [IEEE 802](https://en.wikipedia.org/wiki/IEEE_802) networking protocol model.
     
     Physical layer(PHY): A connection between any two PCIe devices is known as a link, and is built up from a collection of one or more lanes. 

3. USB

### Quartus Tools
Altera FPGAs are commonly used in our designed, so here are some pratical tools in Quartus that could bring great convenience. 

#### IP catalog
Browse several simple test samples here as the MegaWizard Plug-In Manager before. 

#### Signal Tap -- **Nice Debug tool**
> The Signal-Tap II Embedded Logic Analyzer is a system-level debugging tool that captures and displays signals in circuits designed for implementation in Intel/Altera's FPGAs. Signal-Tap runs on the chip, with your design, in real hardware (not simulation) to provide waveforms of logic signals within the design.

#### In-System Memory Content Editor
> It allows you to read data from and write data to in-system memory in a device while the device is running at speed and independently of system clocks with a JTAG interface. You can use the In-System Memory Content Editor to debug designs, for example, by injecting faults into in-system memory while the device is running at speed, or by changing the value of coefficients in DSP applications.

#### In-System Sources and Probes Editor
>  It allows you to **read data from and write data to the design** on the device with a JTAG interface. Once you compile the design and program a device, you can read the contents of the probes in the In-System Sources and Probes Editor pane.

#### Programmer
Detect the .sof file and start programming. JTAG mode is usually applied in our designs.  

#### System Console
> The System Console performs low-level hardware debugging of SOPC Builder systems. The System Console provides read and write access to the IP cores. You can monitor the status of the FPGA through this tool. 

#### RTL Viewer
> The RTL Viewer, State Machine Viewer, and Technology Map Viewer allow you to **view schematic representations of the internal structure** of your designs. Each viewer displays a unique view of the netlist, and allows you to view different internal structures. Removes internally used tri-state buses.

#### Chip Planner
> The Chip Planner provides a visual display of chip resources. It can show logic placement, LogicLock regions, relative resource usage, detailed routing information, fan-ins and fan-outs, paths between registers, and high-speed transceiver channels. You can **view physical timing estimates, routing congestion, and clock regions**.

### Helpful Debug Tools
   1. HLS: Convert from C to RTL Design
   2. Signal Tap: Common Altera debug tool(See more in Quartus Tools)
   3. Global State: A snapshot of local states of each process in the distributed system, as well as the in-tranit messages on communication channels. 

## A Simple Instance
> Here is a sample for programming a provided design on a FPGA board.  

1. Check the **device code** of your FPGA. It is usually listed on the **DATASHEET** of the board. 
   
   Eg. "Intel FPGA PAC Components" -> ""Intel Stratix 10 FPGA" -> "1SX280HN2F43E2VG"

2. Create a new project with this device, then goto the search space in IP catalog and print "pci" for a sample design. Generate the design along with its HDL. 
   
3. **Close** the original project and **open the qpf** in the newly generated design. Compile the design. Then open the working directory, the sof file now could be programmed on the FPGA. 

4. [Optional] Check the permission of the usb port to the FPGA. Use "lsusb" to check the device status: "Bus 002 Device 003: ID 09fb:6010 Altera Stratix10 Darby Creek". Check the permission of bus 002 Device 003 with "cd /dev/bus/usb/002/". If the device you want to write to only have read permission liske "crw-rw-r-- 1 root root 189, 130 2月   8 14:16 003", you can modify it with "sudo chmod 666 ./003" then you will get "crw-rw-rw- 1 root root 189, 130 2月   8 14:16 003". 

5. Open the Programmer in the Tools. Select the device in JTAG mode. If you cannot find the FPGA, please follow note 4. Select the sof file and start programming. The process may fail in 7% (See note 6) or 65%(See note 7). 

6. [Optional] If the process fails at 7%, it probably meets a PMBus error. If you can find some related designs or update packages on **Design Store**, the qsf file inside could be a template for your current design. 
     > PMBus: slow speed two wire communications protocol based on I²C. 
      key difference between slave mode and master mode: only one PMBus device acts as a master on the bus in master mode, however, the PMBus device receives or responds to a command in slave mode. It matters whether the regulator generate or receive the command. [Analog](https://www.analog.com/en/analog-dialogue/articles/power-management-for-fpgas.html)

7. [Optional] If the process fails at 65% or 66%, it probably because the reference clock hasn't connected the correct pin on the FPGA. You can check the **DATASHEET** of the board to find te working refclk pin or just try every possible pins. Usually the pins on the same bank could work at the same time. 
