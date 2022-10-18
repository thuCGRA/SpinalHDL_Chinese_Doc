## RAM/ROM

### 一、语义(Syntax)

用`Mem`类在SpinalHDL中创建memory, 它允许你定义memory并增加读写端口。

以下表格展示了如何例化memory：

|                     语句                     |                                   描述                                   |
| :------------------------------------------: | :----------------------------------------------------------------------: |
|          Mem(type: Data, size: Int)          |                                 创建RAM                                  |
| Mem(type: Data, initialContent: Array[Data]) | 创建ROM, 如果目标是FPGA, 因为memory会被推断成ram模块, 故也可以创建写端口 |

> 备注：如果你想定义ROM, `initialContent`阵列的元素应该是纯粹的值(没有操作符, 没有改变尺寸函数), 在例子里有Sinus rom的例子。

> 备注：给RAM初始化时, 也可以用`init`函数。

> 备注：写mask宽度是灵活的, 可以把mem字拆分成和mask宽度一样的片段。例如如果你有32 bits的mem字并提供了4 bits的mask, 那么这就是字节(byte)mask。如果你提供32 bits的mask, 那这就是bit mask。

以下表格展示了如何给memory增加端口：

| 语句                                                                                                           |                                        描述                                         | 返回类型 |
| :------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------: | :------: |
| mem(address):=data                                                                                             |                                       同步写                                        |          |
| mem(x)                                                                                                         |                                       异步读                                        |    T     |
| mem.write(<br>address<br>data<br>[enable]<br>[mask]<br>)                                                       |     用可选择的mask同步写。如果没有给定enable, 它会自动从被引用的条件区域里推断      |          |
| mem.readAsync(<br>address<br>[readUnderWrite]<br>)                                                             |                            基于可选择的读下写原则异步读                             |    T     |
| mem.readSync(<br>address<br>[enable]<br>[readUnderWrite]<br>[clockCrossing]<br>)                               |               用可选择的enable, 读下写原则和`clockCrossing`模式同步写               |    T     |
| mem.readWriteSync(<br>address<br>data<br>enable<br>write<br>[mask]<br>[readUnderWrite]<br>[clockCrossing]<br>) | 推断读写端口。当`enable && write`时`data`被写入。返回读数据, 当`enable`为真时触发读 |    T     |

> 备注：如果处于某些原因你需要指定非SpinalHDL实现的memory端口, 你可以通过给memory指定一个黑盒来抽象它。

> 重要：SpinalHDL的memory端口不是推断得到的, 而是清晰地定义出的。你不能用像VHDL/Verilog一样的代码模板帮助综合工具推断出memory。

以下是一个推断出双端口ram(32 bits*256)的例子：

```Scala
val mem = Mem(Bits(32 bits), wordCount = 256)
mem.write(
    enable  = io.writeValid,
    address = io.writeAddress,
    data    = io.writeData
)

io.readData := mem.readSync(
    enable  = io.readValid,
    address = io.readAddress
)
```

Verilog:

```Verilog
module MyTopLevel (
  input               io_writeValid,
  input      [7:0]    io_writeAddress,
  input      [31:0]   io_writeData,
  input               io_readValid,
  input      [7:0]    io_readAddress,
  output     [31:0]   io_readData,
  input               clk,
  input               reset
);

  reg        [31:0]   _zz_mem_port1;
  reg [31:0] mem [0:255];

  always @(posedge clk) begin
    if(io_writeValid) begin
      mem[io_writeAddress] <= io_writeData;
    end
  end

  always @(posedge clk) begin
    if(io_readValid) begin
      _zz_mem_port1 <= mem[io_readAddress];
    end
  end

  assign io_readData = _zz_mem_port1;

endmodule

```

### 二、同步使能quirk(Synchronous enable quirk)

如果在条件模块中使用使能信号, 如同`when`, 只有使能信号会作为访存条件生成, `when`条件被忽视。

```Scala
val rom = Mem(Bits(10 bits), 32)
when(cond) {
    io.rdata := rom.readSync(io.addr, io.rdEna)
}
```

在上述例子中, 条件`cond`不会在生成后的RTL中描述, 在使能信号中包含条件`cond`的方法如下：

```Scala
io.rdata := rom.readSync(io.addr, io.rdEna & cond)
```

### 三、读下写原则(Read-under-write policy)

这个规则指定了当在读和写发生在同一周期同一地址时, 写会如何影响。

|     类别     |           描述           |
| :----------: | :----------------------: |
|  `dontCare`  | 当情况发生时不关心读的值 |
| `readFirst`  |   读会取到旧(写前)的值   |
| `writeFirst` |   读会得到新(写后)的值   |

> 重要：生成的VHDL/Verilog总是`readFirst`模式, 会兼容`dontCare`但是不兼容`writeFirst`。为了产生有这个特点的电路, 需要使能*自动化mem黑盒*

### 四、混合宽度ram(Mixed-width ram)

你可以指定访存mem的端口的位宽, 这个位宽是mem位宽的一部分, 是二的幂次方, 函数如下：

|                                                           语句                                                           |                                   描述                                   |
| :----------------------------------------------------------------------------------------------------------------------: | :----------------------------------------------------------------------: |
|                             mem.writeMixedWidth(<br>address<br>data<br>[readUnderWrite]<br>)                             |                            类似于`mem.write`                             |
|                           mem.readAsyncMixedWidth(<br>address<br>data<br>[readUnderWrite]<br>)                           | 类似于`mem.readAsync`, 但是它会驱动`data`给定的信号/对象而不是返回读的值 |
|            mem.readSyncMixedWidth(<br>address<br>data<br>[enable]<br>[readUnderWrite]<br>[clockCrossing]<br>)            | 类似于`mem.readSync`, 但是它会驱动`data`给定的信号/对象而不是返回读的值  |
| mem.readWriteSyncMixedWidth(<br>address<br>data<br>enable<br>write<br>[mask]<br>[readUnderWrite]<br>[clockCrossing]<br>) |                        等价于`mem.readWriteSync`                         |

> 重要：对于读下写机制, 为了使用这个特点你需要使能*自动化mem黑盒*, 因为没有一般性的VHDL/Verilog代码模板推断ram的混合位宽

### 五、自动化mem黑盒(Automatic blackboxing)

因为用传统VHDL/Verilog推断所有ram类型是不可能的, SpinalHDL集成了可选择的自动化黑盒(automatic blackboxing system)系统。这个系统会检查你的RTL网表中所有出现了的mem并把他们用黑盒替代。之后产生的代码会根据第三方IP提供mem的特性, 例如读时写(read-during-write)机制和混合带宽端口。

以下例子说明了如何默认开启mem黑盒：

```Scala
def main(args: Array[String]) {
    SpinalConfig()
        .addStandardMemBlackboxing(blackboxAll)
        .generateVhdl(new TopLevel)
}
```

如果对于你的设计, 标准黑盒工具还不够用, 不要犹豫去创建一个*Github issue*。这里还有一种你创建自己黑盒工具的方法。

1. 黑盒机制(Blackboxing policy)

有多种机制供你使用, 你可以选择你想要把哪个mem变成黑盒, 并且当黑盒不可用时要做什么：

|               种类                |                               描述                               |
| :-------------------------------: | :--------------------------------------------------------------: |
|           `blackboxAll`           |            把所有mem变成黑盒。对于无法黑盒化的mem报错            |
|     `blackboxAllWhatsYouCan`      |                   把所有可黑盒化的mem变成黑盒                    |
| `blackboxRequestedAndUninferable` | 把用户给定的和已知可被推断的mem变成黑盒。对于无法黑盒化的mem报错 |
|     `blackboxOnlyIfRequested`     |         用户指定哪些mem变成黑盒。对于无法黑盒化的mem报错         |

为了清晰地把mem设置成黑盒, 你可以用`generateAsBlackBox`函数

```Scala
val mem = Mem(Rgb(rgbConfig), 1 << 16)
mem.generateAsBlackBox()
```

你也可以通过拓展`MemBlackboxingPolicy`类定义你自己的黑盒机制。

2. 标准mem黑盒(Standard memory blackboxed)

以下展示的是SpinalHDL中用的标准黑盒的VHDL定义：

```VHDL
-- Simple asynchronous dual port (1 write port, 1 read port)
component Ram_1w_1ra is
    generic(
        wordCount : integer;
        wordWidth : integer;
        technology : string;
        readUnderWrite : string;
        wrAddressWidth : integer;
        wrDataWidth : integer;
        wrMaskWidth : integer;
        wrMaskEnable : boolean;
        rdAddressWidth : integer;
        rdDataWidth : integer
    );
    port(
        clk : in std_logic;
        wr_en : in std_logic;
        wr_mask : in std_logic_vector;
        wr_addr : in unsigned;
        wr_data : in std_logic_vector;
        rd_addr : in unsigned;
        rd_data : out std_logic_vector
    );
end component;

-- Simple synchronous dual port (1 write port, 1 read port)
component Ram_1w_1rs is
    generic(
        wordCount : integer;
        wordWidth : integer;
        clockCrossing : boolean;
        technology : string;
        readUnderWrite : string;
        wrAddressWidth : integer;
        wrDataWidth : integer;
        wrMaskWidth : integer;
        wrMaskEnable : boolean;
        rdAddressWidth : integer;
        rdDataWidth : integer;
        rdEnEnable : boolean
    );
    port(
        wr_clk : in std_logic;
        wr_en : in std_logic;
        wr_mask : in std_logic_vector;
        wr_addr : in unsigned;
        wr_data : in std_logic_vector;
        rd_clk : in std_logic;
        rd_en : in std_logic;
        rd_addr : in unsigned;
        rd_data : out std_logic_vector
    );
end component;

-- Single port (1 readWrite port)
component Ram_1wrs is
    generic(
        wordCount : integer;
        wordWidth : integer;
        readUnderWrite : string;
        technology : string
    );
    port(
        clk : in std_logic;
        en : in std_logic;
        wr : in std_logic;
        addr : in unsigned;
        wrData : in std_logic_vector;
        rdData : out std_logic_vector
    );
end component;

--True dual port (2 readWrite port)
component Ram_2wrs is
    generic(
        wordCount : integer;
        wordWidth : integer;
        clockCrossing : boolean;
        technology : string;
        portA_readUnderWrite : string;
        portA_addressWidth : integer;
        portA_dataWidth : integer;
        portA_maskWidth : integer;
        portA_maskEnable : boolean;
        portB_readUnderWrite : string;
        portB_addressWidth : integer;
        portB_dataWidth : integer;
        portB_maskWidth : integer;
        portB_maskEnable : boolean
    );
    port(
        portA_clk : in std_logic;
        portA_en : in std_logic;
        portA_wr : in std_logic;
        portA_mask : in std_logic_vector;
        portA_addr : in unsigned;
        portA_wrData : in std_logic_vector;
        portA_rdData : out std_logic_vector;
        portB_clk : in std_logic;
        portB_en : in std_logic;
        portB_wr : in std_logic;
        portB_mask : in std_logic_vector;
        portB_addr : in unsigned;
        portB_wrData : in std_logic_vector;
        portB_rdData : out std_logic_vector
    );
end component;
```

如你所见, 黑盒有一个技术参数。你可以在对应的mem上用`setTechnology`函数来设置它。当前有4中可用的技术：

+ `auto`
+ `ramBlock`
+ `distributedLut`
+ `registerFile`