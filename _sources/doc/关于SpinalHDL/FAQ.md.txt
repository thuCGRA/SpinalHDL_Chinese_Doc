## FAQ

### 一、SpinalHDL生成的RTL与手工书写VHDL/Verilog相比额外硬件开销有多少？

> 开销为零, 因为SpinalHDL不是HLS方法。它的目标不是将任何任意代码翻译成RTL, 而是提供一种强大的语言来描述RTL并提高抽象级别。

### 二、SpinalHDL未来可能会不支持吗？

> 这个问题有两个方面：
> 
> 1). SpinalHDL生成VHDL/Verilog文件, 这意味着SpinalHDL将在几十年内得到所有EDA工具的支持。
> 
> 2). 如果SpinalHDL中存在一个错误, 并且不再支持修复它, 那么这不是一个致命的情况, 因为SpinalHDL编译器是完全开源的。对于简单的问题, 你可以在几个小时内自己解决问题。请记住EDA公司在其封闭工具中修复问题或添加新功能需要多长的时间。

### 三、SpinalHDL生成的VHDL/Verilog是否保持备注？

> 不, 没有。生成的文件应视为网表。例如, 在编译C代码时, 你是否关心生成的汇编代码中的注释？

### 四、SpinalHDL可以扩展到大工程吗？

> 是的, 已经做了一些实验, 它在生成带缓存的数百个3KLUT CPU只需要12秒左右, 与模拟或综合此类设计所需的时间相比, 这是一个非常短的时间。

### 五、SpinalHDL如何产生的？

> 2014年12月至2016年4月, 这是一个个人爱好项目。但自2016年4月以来, 有一个人全职从事这项工作, 其他一些人也会定期为该项目做出贡献。

### 六、为什么有VHDL/Verilog/SystemVerilog还要开发新语言？

1. VHDL/Verilog/SystemVerilog不是硬件描述语言

    这些语言最初是为仿真/文档目的创建的事件驱动语言。只是后来, 它们被用作综合工具的输入语言。这解释了本页下面许多要点的根源。

2. 这些事件驱动范式对RTL没有任何意义

    仔细想想, 用process/always块描述数字硬件（RTL）没有任何实际意义。为什么我们要担心敏感度列表？为什么我们必须在不同性质的process/always块（组合逻辑/无重置寄存器/异步重置寄存器）之间拆分设计？

    等价的VHDL/SpinalHDL的例子：

    VHDL:

    ```VHDL
    signal mySignal : std_logic;
    signal myRegister : unsigned(3 downto 0);
    signal myRegisterWithReset : unsigned(3 downto 0);

    process(cond)
    begin
        mySignal <= '0';
        if cond = '1' then
            mySignal <= '1';
        end if;
    end process;

    process(clk)
    begin
        if rising_edge(clk) then
            if cond = '1' then
                myRegister <= myRegister + 1;
            end if;
        end if;
    end process;

    process(clk,reset)
    begin
        if reset = '1' then
            myRegisterWithReset <= 0;
        elsif rising_edge(clk) then
            if cond = '1' then
                myRegisterWithReset <= myRegisterWithReset + 1;
            end if;
        end if;
    end process;
    ```

    SpinalHDL

    ```Scala
    val mySignal             = Bool()
    val myRegister           = Reg(UInt(4 bits))
    val myRegisterWithReset  = Reg(UInt(4 bits)) init(0)

    mySignal := False
    when(cond) {
        mySignal            := True
        myRegister          := myRegister + 1
        myRegisterWithReset := myRegisterWithReset + 1
    }
    ```

    对于一切, 你都可以尝试这种事件驱动语言, 知道你尝试到更好的。
    
3. VHDL和Verilog的最新版本不可用。
            
    EDA行业在其工具中实现VHDL 2008和SystemVerilog综合功能的速度非常慢。此外, 完成后, 似乎只实现了语言的一个约束子集（不涉及模拟特性）。这导致使用这些语言版本中任何有趣的功能都是不安全的, 因为：
    
    + 这可能会使你的代码与许多EDA工具不兼容。
    + 其他公司可能不会接受你的IP, 因为他们的流程还没有准备好。
            
    让我告诉你, 我已经厌倦了等待昂贵的封闭源代码工具。
    
4. VHDL和Verilog的最新版本没有那么好。
            
    请参阅VHDL 2008参数化包和无约束记录, 确保它可以编写更好的VHDL源代码, 但从面向对象的角度来看, 它们让我感到恶心（请参阅下一个主题, 了解SpinalHDL示例）。
            
    尽管如此, 这些修订并没有改变HDL问题的核心：它们基于一种事件驱动的范式, 描述数字硬件是没有意义的。

5. VHDL记录(records), Verilog结构(struct)是破碎的(如果你可以使用, SystemVerilog在这方面做的很好)

    你不能用它们来定义接口, 因为你不能定义它们的内部信号方向。 更糟糕的是, 你不能给他们构造参数！ 所以, 定义你的 RGB 记录/结构一次, 并希望你永远不必将它与更大/更小的颜色通道一起使用......

    VHDL 的另一个奇特之处在于, 如果你想将一个数组添加到一个组件实体中, 你必须将这个数组的类型定义到一个包中......它不能被参数化......

    以下是SpinalHDL下APB3总线的定义：

    ```Scala
    //Class which can be instantiated to represent a given APB3 configuration
    case class Apb3Config(
    addressWidth  : Int,
    dataWidth     : Int,
    selWidth      : Int     = 1,
    useSlaveError : Boolean = true
    )

    //Class which can be instantiated to represent a given hardware APB3 bus
    case class Apb3(config: Apb3Config) extends Bundle with IMasterSlave {
    val PADDR      = UInt(config.addressWidth bits)
    val PSEL       = Bits(config.selWidth bits)
    val PENABLE    = Bool()
    val PREADY     = Bool()
    val PWRITE     = Bool()
    val PWDATA     = Bits(config.dataWidth bits)
    val PRDATA     = Bits(config.dataWidth bits)
    val PSLVERROR  = if(config.useSlaveError) Bool() else null  //Optional signal

    //Can be used to setup a given APB3 bus into a master interface of the host component
    override def asMaster(): Unit = {
        out(PADDR,PSEL,PENABLE,PWRITE,PWDATA)
        in(PREADY,PRDATA)
        if(config.useSlaveError) in(PSLVERROR)
    }
    }
    ```

    关于VHDL2008的部分解决方法和SystemVerilog接口/模组端口, 如果你的EDA工具/公司流程/公司规定允许你使用他们, 你很幸运。

6. VHDL和Verilog非常冗长。
            
    实际上, 使用VHDL和Verilog, 当它开始涉及组件实例互连时, 需要考虑复制过去的产品。
    
    为了更深入地理解它, 有一个SpinalHDL示例, 它执行一些外围设备实例化, 并添加访问它们所需的APB3解码器。

    ```Scala
    //Instanciate an AXI4 to APB3 bridge
    val apbBridge = Axi4ToApb3Bridge(
    addressWidth = 20,
    dataWidth    = 32,
    idWidth      = 4
    )

    //Instanciate some APB3 peripherals
    val gpioACtrl = Apb3Gpio(gpioWidth = 32)
    val gpioBCtrl = Apb3Gpio(gpioWidth = 32)
    val timerCtrl = PinsecTimerCtrl()
    val uartCtrl = Apb3UartCtrl(uartCtrlConfig)
    val vgaCtrl = Axi4VgaCtrl(vgaCtrlConfig)

    //Instanciate an APB3 decoder
    //- Drived by the apbBridge
    //- Map each peripherals in a memory region
    val apbDecoder = Apb3Decoder(
        master = apbBridge.io.apb,
        slaves = List(
            gpioACtrl.io.apb -> (0x00000, 4 KiB),
            gpioBCtrl.io.apb -> (0x01000, 4 KiB),
            uartCtrl.io.apb  -> (0x10000, 4 KiB),
            timerCtrl.io.apb -> (0x20000, 4 KiB),
            vgaCtrl.io.apb   -> (0x30000, 4 KiB)
        )
    )
    ```
    
    这样, 当你实例化一个模块\/组件时, 你不必逐个绑定每个信号, 因为你可以以面向对象的方式访问它们的接口。
    
    同样关于VHDL/Verilog 的struct/records, 我只想说它们真的是肮脏的把戏, 没有真正的参数化和可重用性功能, 一些拐杖试图掩盖这些语言设计拙劣的事实。


7. 元硬件描述的功能和能力
     
    好的, 这是一大块。基本上, VHDL/Verilog/SystemVerilog将为你提供一些精化工具, 这些工具不会直接映射到硬件中, 例如loops/generate语句/macro/function/process/task。但仅此而已。

    甚至它们也非常有限。举例来说, 为什么不能将process/alwasys/compents/module块定义为task/process？这真的是许多新奇事物的瓶颈。如果你在总线调用一个用户定义的task/procedure, 比如myHandshakeBus.quequ(depth=64)？这不是很爽吗？

    ```Scala
    //Define the concept of handshake bus
    class Stream[T <: Data](dataType:  T) extends Bundle {
        val valid   = Bool()
        val ready   = Bool()
        val payload = cloneOf(dataType)

        //Define an operator to connect the left operand (this) to the right operand (that)
        def >>(that: Stream[T]): Unit = {
            this.valid := that.valid
            that.ready := this.ready
            this.payload := that.payload
        }

        //Return a Stream connected to this via a FIFO of depth elements
        def queue(depth: Int): Stream[T] = {
            val fifo = new StreamFifo(dataType, depth)
            this >> fifo.io.push
            return fifo.io.pop
        }
    }
    ```
    
    然后让我们进一步了解, 假设你想要定义一个状态机, 你必须编写原始VHDL\/Verilog和一些开关语句来完成。你不能定义一种“状态机”抽象, 这种抽象会给你一种奇特的语法来定义它们, 相反, 你必须使用第三方工具来绘制你的状态机, 然后生成与VHDL/Verilog等效的代码。这真的很乱。
    
    因此, 所谓元硬件描述能力, 我指的是通过使用原始SpinalHDL语法, 你可以定义工具, 然后允许你以抽象的方式定义事物, 比如状态机。

    这里还有一个简单的定义在顶层SpinalHDL的状态机抽象使用的例子：

    ```Scala
    //Define a new state machine
    val fsm = new StateMachine{
        //Define all states
        val stateA, stateB, stateC = new State

        //Set the state machine entry point
        setEntry(stateA)

        //Define a register used into the state machine
        val counter = Reg(UInt(8 bits)) init (0)

        //Define the state machine behavioural
        stateA.whenIsActive (goto(stateB))

        stateB.onEntry(counter := 0)
        stateB.onExit(io.result := True)
        stateB.whenIsActive {
            counter := counter + 1
            when(counter === 4){
            goto(stateC)
            }
        }

        stateC.whenIsActive (goto(stateA))
    }
    ```
    
    同样, 如果你想生成CPU的指令解码, 可能需要一些花哨的精化时间算法来生成尽可能少的逻辑。但是, 在VHDL/Verilog/SystemVerilog中, 执行此类操作的唯一选择是编写一个脚本, 生成所需的.vhd/.v。
    
    关于元硬件描述确实有很多话要说, 但要理解它并获得它真正的味道, 唯一真正的方法是对它进行实验。它的目标是停止像猴子一样玩弄电线和门, 开始与那些低级的东西保持一定距离, 思考更大和可重用的东西。

### 七、如何使用未发布(但是已经提交到git)的SpinalHDL版本？

例如, 如果你想尝试`dev`分支, 在dummy文件夹中做如下操作：

```Scala
git clone https://github.com/SpinalHDL/SpinalHDL.git -b dev
cd SpinalHDL
sbt clean publishLocal
```

在你的工程中, 不要忘记更新build.sbt选定的SpinalHDL的版本, 应该是"dev"而不是"?.?.?"

## 支持(Support)

### 一、联系渠道(Communication channels)

- 支持错误报告和功能请求的通信渠道, 毫不犹豫地创建github问题：   https://github.com/SpinalHDL/SpinalHDL/
 
- 有关SpinalHDL语法和现场对话的问题, 可以使用Gitter频道：   https://gitter.im/SpinalHDL/SpinalHDL

- 如果问题你还可以使用带有标签StackOverflow的论坛:    spinalhl:https://StackOverflow.com/

- 也可以通过谷歌查询。你可以随意发布与SpinalHDL相关的任何主题：  https://groups.google.com/forum/#!forum/spinalhdl-hardware-description-language
