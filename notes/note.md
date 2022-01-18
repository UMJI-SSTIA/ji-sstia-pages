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

# 总线
## 低速总线
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

