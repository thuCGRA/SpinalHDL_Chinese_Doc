## Analog and inout (模拟与IO口)

### 一、简介

用户可以利用`Analog`/`inout`特性定义一个三态信号。这些特性的添加基于以下原因：
+ 能够将本原三状态信号添加到顶层(避免需要用一些手写的VHDL/Verilog手工封装它们)。
+ 允许定义中包含inout引脚的黑盒子(Blackbox)
+ 能够通过层次结构将黑盒子的输入引脚连接到顶层输入引脚

由于这些特性都只是为了方便而添加, 所以目前为止请不要尝试关于三态逻辑的其他花里胡哨的东西。

如果用户想建模一个类似于内存映射GPIO外设的组件, 请使用来自Spinal标准库中名为“TriState/TriStateArray”的`Bundle`组件, 该组件抽象了三态驱动的真实特性。

### 二、Analog(模拟)

`Analog`是可以将某个信号定义为模拟性质的关键词, 在数字领域可以表示`0`、`1`或`Z`(未连接、高阻态)等。例如：
```Scala
case class SdramInterface(g : SdramLayout) extends Bundle {
  val DQ    = Analog(Bits(g.dataWidth bits)) // 双向数据总线
  val DQM   = Bits(g.bytePerWord bits)
  val ADDR  = Bits(g.chipAddressWidth bits)
  val BA    = Bits(g.bankWidth bits)
  val CKE, CSn, CASn, RASn, WEn  = Bool
}
```

### 三、Inout(IO口)

inout是允许用户将`Analog`信号设置为双向(输入和输出)信号的关键字。例如：
```Scala
case class SdramInterface(g : SdramLayout) extends Bundle with IMasterSlave {
  val DQ    = Analog(Bits(g.dataWidth bits)) // 双向数据总线
  val DQM   = Bits(g.bytePerWord bits)
  val ADDR  = Bits(g.chipAddressWidth bits)
  val BA    = Bits(g.bankWidth bits)
  val CKE, CSn, CASn, RASn, WEn  = Bool

  override def asMaster() : Unit = {
    out(ADDR, BA, CASn, CKE, CSn, DQM, RASn, WEn)
    inout(DQ) // 设置DQ为组件的一个IO信号
  }
}
```

### 四、InOutWrapper(IO口封装)

`InOutWrapper`是一个允许用户将组件中的所有`master TriState/TriStateArray/ReadableOpenDrain`包(`Bundle`)转换为`inout(Analog(...))`信号的工具。该工具允许用户的硬件代码无`Analog/inout`描述并且通过转换顶层使得其可以综合。例如：
```Scala
case class Apb3Gpio(gpioWidth : Int) extends Component {
  val io = new Bundle{
    val gpio = master(TriStateArray(gpioWidth bits))
    val apb  = slave(Apb3(Apb3Gpio.getApb3Config()))
  }
  ...
}

SpinalVhdl(InOutWrapper(Apb3Gpio(32)))
```
会生成如下代码：
```Vhdl
entity Apb3Gpio is
  port(
    io_gpio : inout std_logic_vector(31 downto 0); -- 该 io_gpio最初是一个`TriStateArray Bundle`
    io_apb_PADDR : in unsigned(3 downto 0);
    io_apb_PSEL : in std_logic_vector(0 downto 0);
    io_apb_PENABLE : in std_logic;
    io_apb_PREADY : out std_logic;
    io_apb_PWRITE : in std_logic;
    io_apb_PWDATA : in std_logic_vector(31 downto 0);
    io_apb_PRDATA : out std_logic_vector(31 downto 0);
    io_apb_PSLVERROR : out std_logic;
    clk : in std_logic;
    reset : in std_logic
  );
end Apb3Gpio;
```
而不是生成如下代码：
```Vhdl
entity Apb3Gpio is
  port(
    io_gpio_read : in std_logic_vector(31 downto 0);
    io_gpio_write : out std_logic_vector(31 downto 0);
    io_gpio_writeEnable : out std_logic_vector(31 downto 0);
    io_apb_PADDR : in unsigned(3 downto 0);
    io_apb_PSEL : in std_logic_vector(0 downto 0);
    io_apb_PENABLE : in std_logic;
    io_apb_PREADY : out std_logic;
    io_apb_PWRITE : in std_logic;
    io_apb_PWDATA : in std_logic_vector(31 downto 0);
    io_apb_PRDATA : out std_logic_vector(31 downto 0);
    io_apb_PSLVERROR : out std_logic;
    clk : in std_logic;
    reset : in std_logic
  );
end Apb3Gpio;
```

### 五、Manually driving Analog bundles(手动驱动模拟束)

如果一个`Analog`包没有被驱动, 则其会被默认设置为高阻态Z。因此为了手动实现一个三态驱动器(在`InOutWrapper`类型可能有时无法使用的情况下), 用户不得不选择性地驱动信号。

手动连接一个`TriState`信号到一个`Analog`包的代码例子如下：
```Scala
case class Example extends Component {
  val io = new Bundle {
    val tri = slave(TriState(Bits(16 bits)))
    val analog = inout(Analog(Bits(16 bits)))
  }
  io.tri.read := io.analog
  when(io.tri.writeEnable) { io.analog := io.tri.write }
}
```
