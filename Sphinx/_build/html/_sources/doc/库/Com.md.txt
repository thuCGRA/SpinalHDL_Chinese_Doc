## Com

### 一、UART

1. 简介

   可以使用UART协议来发出和接收RS232 / RS485帧。

   有一个没有奇偶校验并拥有一个停止位的8位帧的例子:

   ![uart](../../img/uart.png)


2. 总线定义

   ```Scala
   case class Uart() extends Bundle with IMasterSlave {
       val txd = Bool() // Used to emit frames
       val rxd = Bool() // Used to receive frames

       override def asMaster(): Unit = {
           out(txd)
           in(rxd)
       }
   }
   ```

3. UartCtrl

   库中实现了一个Uart控制器。这个控制器的特性是使用一个采样窗口来读取`rxd`引脚, 然后使用多数投票制来过滤它的值。
   | IO口名 |  方向  |      类型      |                       描述                        |
   | :----: | :----: | :------------: | :-----------------------------------------------: |
   | config |   in   | UartCtrlConfig | 用于设置控制器的时钟分频器/奇偶校验/停止/数据长度 |
   | write  | slave  |  Stream[Bits]  |          用于请求进行帧交换传输的流端口           |
   |  read  | master |   Flow[Bits]   |              用于接收解码帧的流端口               |
   | write  | master |      Uart      |               与实际实现的连接接口                |

   控制器可以通过一个`UartCtrlGenerics`配置对象实例化:
   |       属性        | 类型  |                         描述                         |
   | :---------------: | :---: | :--------------------------------------------------: |
   |   dataWidthMax    |  Int  |                    帧内的最大位数                    |
   | clockDividerWidth |  Int  |                 内部时钟分频器的位宽                 |
   |  preSamplingSize  |  Int  |   指定有多少samplingTick在一个UART波特的起始处下降   |
   |   samplingSize    |  Int  | 指定有多少samplingTick用于采样UART波特中段的`rxd`值  |
   | postSamplingSize  |  Int  | 指定在一个UART波特值的末尾有多少个samplingTick被丢弃 |

### 二、USB设备

1. 简介
    
   SpinalHDL库中有一个USB设备控制器。在以下几个要点中, 它可以设置为:
   + 它允许CPU配置和管理端点
   + 存储端点状态和事务描述符的内部RAM
   + 多达16个端点(几乎没有额外开销)
   + 支持USB主机全速运行(12Mbps)
   + 在linux上使用自己的驱动程序进行测试(https://github.com/SpinalHDL/linux/blob/dev/drivers/usb/gadget/udc/spinal_udc.c)
   + 用于配置的Bmb内存接口
   + 内部需要一个频率是12Mhz的倍数的时钟, 至少为48Mhz
   + 控制器频率不受限制
   + 不需要外部phy

   Linux小工具测试和功能:
   + 串行连接
   + 以太网连接
   + 大容量存储(在ArtyA7 linux上达到8mbps)

   部署：
   + https://github.com/SpinalHDL/SaxonSoc/tree/dev-0.3/bsp/digilent/ArtyA7SmpLinux
   + https://github.com/SpinalHDL/SaxonSoc/tree/dev-0.3/bsp/radiona/ulx3s/smp

2. 架构(Architecture)

   控制器由以下部分组成：
   + 少数控制寄存器
   + 一个用来存储端点状态的内部RAM, 一个传输描述符合端点0配置数据。

   每个端点的描述符链表是为了处理USB的出入任务和数据。

   端点0也会像其他端点一样处理出入USB的交换任务但也会有一些额外的硬件来处理SETUP(设置)任务：
   + 它的链表在每个设置任务上被清除
   + 设置任务的数据存储在一个固定的位置(SETUP_DATA)
   + 它对设置任务有一个特定的中断标志

3. 寄存器(Registers)

   注意, 控制器的所有寄存器和内存只能以32位的字访问, 不支持字节访问。

   - 帧FRAME (0xFF00)

       |    名称    | 类型  | 比特  |      描述       |
       | :--------: | :---: | :---: | :-------------: |
       | usbFrameId |  RO   | 31-0  | 目前的usb帧的id |

   - 地址ADDRESS (0xFF04)

       |  名称   | 类型  | 比特  |                                   描述                                    |
       | :-----: | :---: | :---: | :-----------------------------------------------------------------------: |
       | address |  WO   |  6-0  | 该USB设备只会被特定地址的令牌(token)所控制。其字段会被usb重置任务自动清除 |
       | enable  |  WO   |   8   |                         如果设置, 启用USB地址过滤                         |
       | trigger |  WO   |   9   |        设置下一个EP0 IN令牌使能(见上文)。在任何EP0完成后由硬件清除        |

       这里的想法是保持整个寄存器清空, 直到EP0上接收到USB SET_ADDRESS设置包。此时, 用户可以设置地址和触发器字段, 然后向EP0提供IN零长度描述符, 以结束SET_ADDRESS序列。然后, 控制器将在描述符完成时自动打开地址过滤。

   - 中断INTERRUPT (0xFF08)

       这个寄存器的所有位都可以通过写入“1”来清除。
       |    名称    | 类型  | 比特  |           描述            |
       | :--------: | :---: | :---: | :-----------------------: |
       | endpoints  |  RC   | 15-0  |   当端点产生中断时拉高    |
       |   reset    |  RC   |  16   |    当USB复位发生时拉高    |
       |  ep0Setup  |  RC   |  17   | 当端点0收到配置请求时拉高 |
       |  suspend   |  RC   |  18   |     当端点悬挂时拉高      |
       |   resume   |  RC   |  19   |     当端点恢复时拉高      |
       | disconnect |  RC   |  20   |   当端点连接中断时拉高    |
   
   - Halt (0xFF0C)

       这个寄存器允许在休眠状态下放置一个端点, 以确保CPU操作的原子性, 允许对端点寄存器和描述符进行读/修改/写操作。如果给定的端点是由usb主机寻址的, 外围设备将返回NAK。

       |       名称       | 类型  | 比特  |                          描述                          |
       | :--------------: | :---: | :---: | :----------------------------------------------------: |
       |    endpointId    |  WO   |  3-0  |                 用户想要休眠的目标端点                 |
       |      enable      |  WO   |   4   |                                                        |
       | effective enable |  RO   |   5   | 在设置了使能后, 需要等待硬件本身设置该位, 以确保原子性 |

   - 配置CONFIG (0xFF10)

       |         名称         | 类型  | 比特  |                 描述                 |
       | :------------------: | :---: | :---: | :----------------------------------: |
       |      pullupSet       |  SO   |   0   | 写入' 1 '来启用dp引脚上的USB设备上拉 |
       |     pullupClear      |  SO   |   1   |                                      |
       |  interruptEnableSet  |  SO   |   2   |    写“1”, 让现在和未来的中断发生     |
       | interruptEnableClear |  SO   |   3   |                                      |

   - INFO (0xFF20)

       |  名称   | 类型  | 比特  |            描述            |
       | :-----: | :---: | :---: | :------------------------: |
       | ramSize |  RO   |  3-0  | 内部ram将有(1 << this)字节 |

   - 端点 ENDPOINTS (0x0000 - 0x003F)

       端点状态存储在内部ram的开头, 每个有32位字。
       |     名称      | 类型  | 比特  |                               描述                                |
       | :-----------: | :---: | :---: | :---------------------------------------------------------------: |
       |    enable     |  RW   |   0   |             若不设置, 则该端点忽略所有的流量(traffic)             |
       |     stall     |  RW   |   1   |                 若设置了, 端点将始终返回STALL状态                 |
       |     nack      |  RW   |   2   |                 若设置了, 端点将始终返回NACK状态                  |
       |   dataPhase   |  RW   |   3   |  指定使用的IO数据PID。' 0 ' = > DATA0。这个字段也由控制器更新。   |
       |     head      |  RW   | 15-4  | 指定当前描述符头部(链表)0 => empty list, byte address = this << 4 |
       |  isochronous  |  RW   |  16   | 指定当前描述符头部(链表)0 => empty list, byte address = this << 4 |
       | maxPacketSize |  RW   | 31-22 |                                                                   |

       为了获得一个端点响应需要：设置其使能标志位为1.

       还有其他一些例子:要么用户有停滞或纳标记集,因此,控制器总是有相应的响应；要么EP0设置请求,控制器不会使用描述符,但将数据写入SETUP_DATA寄存器和ACK；要么用户有一个空链表(head==0)并响应NACK；要么用户至少有一个描述符由头部指出,在这种情况下将执行和ACK。

   - SETUP_DATA (0x0040 - 0x0047)
   - 
   - 当端点0接收SETUP任务时, 该任务的数据将存储在该位置

4. 描述符(Descriptors)

   描述符允许指定端点需要如何处理IO传输任务的数据阶段。它们存储在内部ram中, 可以通过它们的链表链接在一起, 并且需要在16字节的边界上对齐.

   |       名称        |  字   | 比特  |                                                               述                                                                |
   | :---------------: | :---: | :---: |:-------------------------------------------------------------------------------------------------------------------------------: |
   |      offset       |   0   | 15-0  |                                                  指定当前传输进度(以字节为位)                                                   |
   |       code        |   0   | 19-16 |                                                0xF => in progress, 0x0 =>success                                                 |
   |       next        |   1   | 15-4  |                                      指定下一个传输符 0 => nothing, byte address = this <<4                                      |
   |      length       |   1   | 31-16 |                                                      为数据字段分配的字数                                                       |
   |     direction     |   2   |  16   |                                                       ‘0’ => OUT, ‘1’ =>IN                                                       |
   |     interrupt     |   2   |  17   |                                            如果设置了, 则描述符的完成将生成一个断。                                             |
   | completionOnFull  |   2   |  18   | 通常, 描述符补全只发生在USB传输小于maxPacketSize时。但是如果设置了这个字段, 那么当描述符被填满时也被认为事件已完成(抵消= =长度) |
   | data1OnCompletion |   2   |  19   |                                                     强制端点dataphaseDATA1                                                      |
   |      data        |  ...  |  ...  |                                                                                                                                  |

   注意, 如果控制器接收到一个帧, 其中IO不匹配描述符IO, 该帧将被忽略。

   另外, 要初始化描述符, CPU应该将代码字段设置为0xF.

5. 使用(usage)

   ```Scala
   import spinal.core._
   import spinal.core.sim._
   import spinal.lib.bus.bmb.BmbParameter
   import spinal.lib.com.usb.phy.UsbDevicePhyNative
   import spinal.lib.com.usb.sim.UsbLsFsPhyAbstractIoAgent
   import spinal.lib.com.usb.udc.{UsbDeviceCtrl, UsbDeviceCtrlParameter}


   case class UsbDeviceTop() extends Component {
       val ctrlCd = ClockDomain.external("ctrlCd", frequency = FixedFrequency(100 MHz))
       val phyCd = ClockDomain.external("phyCd", frequency = FixedFrequency(48 MHz))

       val ctrl = ctrlCd on new UsbDeviceCtrl(
           p = UsbDeviceCtrlParameter(
           addressWidth = 14
           ),
           bmbParameter = BmbParameter(
           addressWidth = UsbDeviceCtrl.ctrlAddressWidth,
           dataWidth = 32,
           sourceWidth = 0,
           contextWidth = 0,
           lengthWidth = 2
           )
       )

   val phy = phyCd on new UsbDevicePhyNative(sim = true)
   ctrl.io.phy.cc(ctrlCd, phyCd) <> phy.io.ctrl

   val bmb = ctrl.io.ctrl.toIo()
   val usb = phy.io.usb.toIo()
   val power = phy.io.power.toIo()
   val pullup = phy.io.pullup.toIo()
   val interrupts = ctrl.io.interrupt.toIo()
   }


   object UsbDeviceGen extends App{
   SpinalVerilog(new UsbDeviceTop())
   }
   ```
