## Manager your Environment 
1. Download Linux Environmentr as SSH server [MobaXterm](https://mobaxterm.mobatek.net/download.html) 
2. Register with your IP(given)
3. Make several softwares accessiable on your Linux

## Linux basic command
ll

dd

p

find ./ | grep [name]

lsusb --查看USB端口 

ps 查看进程pid

unzip (-d) (destination) filename

tar -zcvf file.tar.gz [source_destination] #压缩

tar -ztvf file.tar.gz #只查看，不解压

tar -zxvf file.tar.gz -C [desitination] #解压到指定位置

tar -zxvf file.tar.gz -C [destination] [filename] #只解压压缩包中的特定文件到指定位置

## Download possible packages

## Get familiar with some commonly used softwares

Q: Have no admin access so cannot install or test the acceleration stack and ODAE successfully.

A: 

Q: 

## quantus使用, sof file load in the board
找产品型号
寻找例程（可参考alTera的其他例程）
qsf file for quartus to execute
编译成sof之类的文件是可以烧进FPGA里面的
hardware setup cannot detect the board 但是usb可以看到串口连接-> 可能是权限不能访问usb端口，但是sudo quartus不能传过来UI界面
可以通过lsusb查询Altera串口,然后用chmod修改一下权限让他能够write，就能在programmer里面检测到了

Q：1SX280HH1这个芯片型号和datasheet里面1SX280HN2F43E2VG型号不兼容
A: 查找到对的code之后，更改，重新跑一下generate bitstream就可以进去了

Q: 7%时fail，报错"a PMBus error, VID setting is incorrect in the Intel Quartus Prime project". 可能是I/O port assignment的问题

A1：先找类似的example(Design store)试试引脚是否能用，不行再找对应板子的pin表自己对照着手动配置。先把qsf文件内PWBus相关的内容注释掉看看compile结果如何

A2: 找到 [Intel® Stratix® 10 Power Management User Guide](https://www.intel.com/content/www/us/en/docs/programmable/683418/21-1/power-management-and-vid-parameters.html) 里面的PM和VID参数配置, 通过编译验证不能改成Slave模式，但是三个coefficient还是不知道怎么设置，找不到芯片对应的具体数值（随便范围内设置不知道能不能过）

A3: reference 里面找到sof文件dcp的可以成功烧入(是纯探测output的工程)。这个FPGA默认让你用OPAE和配套的openCL和驱动使用，所以有不少接口信息找不到


[>_<]: # (No paths exist between clock target "dut|dut|altera_pcie_s10_hip_ast_pipen1b_inst|altera_pcie_s10_hip_ast_pllnphy_inst|g_phy_g1x1.phy_g1x1|phy_g1x1|altera_xcvr_hip_channel_s10_ch0|altera_xcvr_pcie_hip_channel_s10_ch0|g_xcvr_native_insts[0].ct2_xcvr_native_inst|inst_ct2_xcvr_channel_multi|gen_rev.ct2_xcvr_channel_inst|gen_ct1_hssi_pldadapt_tx.inst_ct1_hssi_pldadapt_tx|pld_pcs_tx_clk_out1_dcm" of clock "dut|dut|altera_pcie_s10_hip_ast_pipen1b_inst|altera_pcie_s10_hip_ast_pllnphy_inst|g_phy_g1x1.phy_g1x1|phy_g1x1|xcvr_hip_native|ch0" and its clock source. Assuming zero source clock latency.)




## Quanrtus 尝试
1. pin planner
   如果datasheet找不到对应管脚？
   should we understand the Verilog code in sample project?  
   Some auto selectiona, please check
2. chip planner
3. RTL viewer

simulation using VCS

### pcie的存在
pcie transactions; PIPE mechanism 一些接线方法实现不一样的协议（管线带宽固定之后是加频率用来提升）
可以先从基础逻辑理解从而排除一些错误答案

> HLS--从C到RTL design

> signal tab --alTera debug tool

> Global xxx -- snapshot debug -- 可以看到某一时刻的很多Reg


PMBus regulator
1. EM2130 L/H
2. LTM4625

## Power Management for FPGAs
   >参考 [analog](https://www.analog.com/en/analog-dialogue/articles/power-management-for-fpgas.html)

   > 更多可能的资料：
   BMC上power sequence

   ### PMBus
      slow speed two wire communications protocol based on I²C
      key difference--only one PMBus device acts as a master on the bus in master mode, however, the PMBus device receives or responds to a command in Slave mode. whether the regulator generate or receive the command. 
      master mode:
      slave mode:

## Device is in Configuration State
   the pin planner? the initialized configuration? Could the user change the refclk pin other than the fitter provides?


## PMBus and VID Setting Solution
 I checked the update package of the d5005 chip and searched for qsf file in openCL hardware manager, and changed power management setting in my project. The core error is a wrong address number of the slave device protected by the openCL interface.

# FPGA目前可能的应用方向：
1. 本身CPU和GPU相连，在单独process GPU，连接silicon和host的时候需要一个接口的处理和代码的转换。此时就可以用一个FPGA将cxl的bus转换成pcie的传给host，可以在host上面简单实现部分实操中的功能。
   
   这里的gap在于做好的RTL system Verilog很多是不可synthesis的，是直接给GPU使用的，但是要在FPGA上跑需要转换成可以synthesis的代码。

   这一应用的意义在于通过FPGA的烧入可以在做出RTL的design之后现在板子上简单验证，并且看到一些只看simulation不能发现的问题，而无需等到做出ASIC的产品（往往需要较长的工期）。这样可以在做出design后很快开始相应软件的开发，有助于软硬件的开发时间线的统一。
2. 对于一些网卡，某些性能需求不那么高的固定功能可以将ASIC替换成FPGA来实现。
3. 一些acceleration stack可以用于verify从GPU转到ASIC的attempt（阶梯式上升）。

之后接触一些常见的硬件模型
FIFO, RAM

## FPGA Configuration

## FPGA Assembly

## FIFO
同博客如下

## SRAM -- static RAM
[CSDN博客](https://blog.csdn.net/edward_zcl/article/details/100086555)

--------------------
--------------------

# Bus Type
## Low speed Bus
### SPI

SPI, sometimes called a four-wire serial bus, is a synchronous serial communication interface for short distance communication. It's in full-duplex mode which allows communication in both directions at the same time, and a master-slave architecture. 
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
  1. more pins required than I2C or three-wire varants
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
--------------------
  
### I2C -- Inter-Integrated Circuit
> It's a synchronous, multi-controller/target, serial communication bus, which is widely used for attaching **lower-speed** peripheral ICs to processors and microcontrollers in **short-distance, intra-board** communication. SMBus is a derived interface from I2C with  stronger robustness and interoperability. 
+ Design
  - I2C uses only two bidirectional open-collector or open-drain lines: serial data line **(SDA)** and serial clock line **(SCL)**, pulled up with resistors. Typical voltages used are +5 V or +3.3 V, although systems with other voltages are permitted. (the FPGA I programmed used 1.8 V)
  - The I2C reference design has a **7-bit address space**, with a rarely used 10-bit extension. Note that the bit rates are quoted for the transfers between controller and target without clock stretching or other hardware overhead. Thus the actual transfer rate of user data is lower than those peak bit rates alone would imply. 
+ Message protocols
  - I2C defines basic types of transactions, each of which begins with a *START* and ends with a *STOP*. 
  - Pure I2C systems support arbitrary message structures. However, SMBus is restricted to nine of those structures. PMBus extends SMBus with a Group protocol, allowing multiple such SMBus transactions to be sent in one combined message. The terminating *STOP* indicates when those grouped actions should **take effect**. 
+ Physical layer
  
   At the physical layer, both SCL and SDA lines are an open-drain (MOSFET) or open-collector (BJT) bus design, thus a pull-up resistor is needed for each line， meaning they're active-high. High-speed systems (and some others) may use a current source instead of a resistor to pull-up only SCL or both SCL and SDA, to accommodate higher bus capacitance and enable faster rise times. 
   
   An important consequence of this is that multiple nodes may be driving the lines simultaneously. If any node is driving the line low, it will be low. Nodes that are trying to transmit a logical one (i.e. letting the line float high) can detect this and conclude that another node is active at the same time.
+ Buffering and multiplexing(related to time domain problems)
  
  When there are many I2C devices in a system, there can be a need to include bus buffers or multiplexers to split large bus segments into smaller ones. Buffers can be used to isolate capacitance on one segment from another and/or allow I2C to be sent over longer cables or traces. 
+ Applications
  - I2C is appropriate for peripherals where simplicity and low manufacturing cost are more important than speed.
  - A particular strength of I2C is the capability of a microcontroller to control a network of device chips with just **two** general-purpose I/O pins and software. 
  - Enable **plug and play** operation via small ROM in SPD, VGA, HDMI, etc.
  - System Management for PC systems through SMBus, such as in PCI and PCIe.
  - Access real-time clock + low speed DACs and ADCs.
  - Changing and read settings in monitors through Display Data Channel. 
  - Comtrolling small LCD or OLED displays. 
  - Turning on and off the power supply of system componets.
+ Limitations
  - **Address Management:** The assignment of target addresses is one weakness of I2C. Seven bits is too few to prevent address collisions between the many thousands of available devices. 
  - **Speed:** I2C supports a limited range of speeds. 
  - Devices are allowed to **stretch clock cycles** to suit their particular needs, which can starve bandwidth needed by faster devices and increase latencies when talking to other device addresses. 
  - **Potential Fault:** Because I2C is a shared bus, there is the potential for any device to have a fault and **hang the entire bus**. 

### UART -- Universal asynchronous receiver-transmitter
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


### JTAG -- named after the **Joint Test Action Group** which codified it
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
   2. **Non-JTAG enabled device:**  DDR, SDRAM, SRAM, flash, MDIO controlled Ethernet PHYs, SPI and I2C temperature sensors, real time clocks, ADCs and DACs are just some examples of such devices. The connection test will still provide excellent coverage for short circuit faults on the nets linking these non-JTAG devices to JTAG enabled devices; however it cannot check for open circuit faults at either the JTAG device or the non-JTAG device.

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

REference: See figures on first three chapters of [XJTAG](https://www.xjtag.com/about-jtag/what-is-jtag/). 



## High Speed Bus

### DDR

### PCIe

### USB


----------------------
# PLL -- Phase-locked loop
> It is a control system that generates an output signal whose phase is related to the phase of an input signal. 
> 
> Keeping the input and output phase in lock step also implies keeping the input and output frequencies the same. Consequently, in addition to synchronizing signals, a phase-locked loop can track an input frequency, or it can generate a frequency that is a multiple of the input frequency. These properties are used for computer clock synchronization, demodulation, and frequency synthesis.
+ Clock Clean-Up Circuit
  > In its most basic configuration, a phase-locked loop compares the phase of a reference signal (FREF) to the phase of an adjustable feedback signal (RFIN) F0. 
  
  When the comparison is in steady-state, and the output frequency and phase are matched to the incoming frequency and phase of the error detector, we say that the PLL is **locked**. ![Basic PLL Configuration](https://www.analog.com/-/media/images/analog-dialogue/en/volume-52/number-3/articles/phase-locked-loop-pll-fundamentals/184330_fig_02.png?la=en&imgver=1)

+ PFD -- phase frequency detector: 
  
  > PFD compares the frequency and phase of the input to REFIN to the frequency and phase of the feedback to RFIN. As such, it can be used with a high quality voltage controlled crystal oscillator (VCXO) and a narrow low-pass filter to clean up a noisy REFIN clock. (With the feedback divider **N = 1**). ![PFD Circuit](https://www.analog.com/-/media/images/analog-dialogue/en/volume-52/number-3/articles/phase-locked-loop-pll-fundamentals/184330_fig_03.png?la=en&imgver=1)The phase frequency detector in Figure 3 compares the input to FREF at +IN and the feedback signal at –IN. It uses two D-type flip flops with a delay element. One Q output enables a positive current source, and the other Q output enables a negative current source. These current sources are known as the charge pump. 
  ![When there's no phase difference](https://www.analog.com/-/media/images/analog-dialogue/en/volume-52/number-3/articles/phase-locked-loop-pll-fundamentals/184330_fig_05.png?la=en&imgver=1)

+ High Frequency Integer-N Architecture 
  
  To generate a range of higher frequencies, a VCO is used, which tunes over a wider range than a VCXO. In such PLLs, the output is **a high multiple** of the reference frequency. 
  > Voltage controlled oscillators contain a variable tuning element, such as a varactor diode, which varies its capacitance with input voltage, allowing a tuneable resonant circuit, which permits a range of frequencies to be generated. The PLL can be thought of as a control system for this VCO. 
  
  **A feedback divider** is used to divide the VCO frequency to the PFD frequency, which allows a PLL to generate output frequencies that are multiples of the PFD frequency. A divider may also be used in the reference path, which permits higher frequency references to be used than the PFD frequency. **The PLL counters** are the second essential element to be considered in some circuits. 
  > Phase Noise: The by-products of the frequency synthesis process, or spurious frequencies (spurs for short). For integer-N PLLs, spurious frequencies are generated by the PFD frequency. An ideal tone would have no noise or additional spurious frequency, but in practice phase noise appears as a skirt around a carrier. 

+ Integer-N and Fractional-N Divider
  > For narrow-band applications, the channel spacing is narrow (typically <5 MHz) and the feedback counter, N, is high. Gaining high N values with a small circuit is achieved by the use of a dual modulus P/P + 1 prescaler. (Here is a calculation: N = P*B + A). The prescaler is generally designed using a higher frequency circuit technology, such as bipolar emitter coupled logic (ECL) circuits, while the A and B counters can take this lower frequency prescaler output and can be manufactured with lower speed CMOS circuitry. This reduces circuit area and power consumption. Low frequency clean up PLLs omit this prescaler.

REference: [Analog](https://www.analog.com/en/analog-dialogue/articles/phase-locked-loop-pll-fundamentals.html). 
