## VHDL例化和Verilog IP(Instantiate VHDL and Verilog IP)

### 一、描述(Description)

黑盒(blackbox)允许使用者把已有的VHDL/Verilog模块通过指定接口集成到当前设计中, 这种调用的正确描述取决于仿真器或综合器。

### 二、定义黑盒(Defining an blackbox)

一个定义黑盒的例子如下所示：

```Scala
//定义Ram黑盒
class Ram_1w_1r(wordWidth: Int, wordCount: Int) extends BlackBox {
    //给黑盒增加VHDL范式/Verilog参数
    //你可以用String, Int, Double, Boolean和所有基于SpinalHDL的类型作为参数值
    addGeneric("wordCount", wordCount)
    addGeneric("wordWidth", wordWidth)

    //定义VHDL通道的io/Verilog模块
    val io = new Bundle {
        val clk = in Bool()
        val wr = new Bundle {
            val en   = in Bool()
            val addr = in UInt(log2Up(wordCount) bits)
            val data = in Bits(wordWidth bits)
        }
        val rd = new Bundle {
            val en   = in Bool()
            val addr = in UInt(log2Up(wordCount) bits)
            val data = out Bits(wordWidth bits)
        }
    }

    //把当前时钟域映射到io.clk引脚
    mapClockDomain(clock = io.clk)
}
```

Verilog:

```Verilog
  Ram_1w_1r #(
    .wordCount(4),
    .wordWidth(4)
  ) test (
    .io_clk     (clk                 ), //i
    .io_wr_en   (wren                ), //i
    .io_wr_addr (wraddr[1:0]         ), //i
    .io_wr_data (wrdata[3:0]         ), //i
    .io_rd_en   (rden                ), //i
    .io_rd_addr (rdaddr[1:0]         ), //i
    .io_rd_data (test_io_rd_data[3:0])  //o
  );

```

在VHDL, `Bool`信号类型会被翻译成`std_logic`, `Bits`会被翻译成`std_logic_vector`。如果你想得到`std_ulogic`, 你需要用`BlackBoxULogic`而不是`BlackBox`。

在Verilog中, `BlackBoxUlogic`没有影响。

```Scala
class Ram_1w_1r(wordWidth: Int, wordCount: Int) extends BlackBoxULogic {
    ...
}
```

### 三、范式(Generics)

有两种方式定义范式：

```Scala
class Ram(wordWidth: Int, wordCount: Int) extends BlackBox {
    addGeneric("wordCount", wordCount)
    addGeneric("wordWidth", wordWidth)

    //或者

    val generic = new Generic {
        val wordCount = Ram.this.wordCount
        val wordWidth = Ram.this.wirdWidth
    }
}
```

### 四、例化黑盒(Instantiating a blackbox)

只需要像例化`Component`一样例化`BlackBox`：

```Scala
//创建顶层并例化Ram
class TopLevel extends Component {
    val io = new Bundle {
        val wr = new Bundle {
            val en   = in Bool()
            val addr = in UInt(log2Up(16) bits)
            val data = in Bits(8 bits)
        }
        val rd = new Bundle {
            val en   = in Bool()
            val addr = in UInt(log2Up(16) bits)
            val data = out Bits(8 bits)
        }
    }

    //例化黑盒
    val ram = new Ram_1w_1r(8, 16)

    //连接所有信号
    io.wr.en    <> ram.io.wr.en
    io.wr.addr  <> ram.io.wr.addr
    io.wr.data    <> ram.io.wr.data
    io.rd.en  <> ram.io.rd.en
    io.rd.addr    <> ram.io.rd.addr
    io.rd.data  <> ram.io.rd.data
}

object Main {
    def main(args: Array[String]): Unit = {
        SpinalVhdl(new TopLevel)
    }
}
```

Verilog:

```Verilog
module MyTopLevel (
  input               io_wr_en,
  input      [3:0]    io_wr_addr,
  input      [7:0]    io_wr_data,
  input               io_rd_en,
  input      [3:0]    io_rd_addr,
  output     [7:0]    io_rd_data,
  input               clk
);

  wire       [7:0]    ram_io_rd_data;

  Ram_1w_1r #(
    .wordCount(16),
    .wordWidth(8)
  ) ram (
    .io_clk     (clk                ), //i
    .io_wr_en   (io_wr_en           ), //i
    .io_wr_addr (io_wr_addr[3:0]    ), //i
    .io_wr_data (io_wr_data[7:0]    ), //i
    .io_rd_en   (io_rd_en           ), //i
    .io_rd_addr (io_rd_addr[3:0]    ), //i
    .io_rd_data (ram_io_rd_data[7:0])  //o
  );
  assign io_rd_data = ram_io_rd_data;

endmodule

```

### 五、时钟和复位的布局(Clock and reset mapping)

在你定义的黑盒中, 你需要清晰地定义出时钟和复位wire。为了能把`ClockDomain`的信号映射到对应的黑盒输入端, 你可以使用`mapClockDomain`或`mapCurrentClockDomain`函数。`mapClockDomain`有如下参数：

|   参数名    |  数据类型   |        默认         |           描述           |
| :---------: | :---------: | :-----------------: | :----------------------: |
| clockDomain | ClockDomain | clockDomain.current |   指定提供信号的时钟域   |
|    clock    |    Bool     |       Nothing       | 连接时钟域时钟的黑盒输入 |
|    reset    |    Bool     |       Nothing       | 连接时钟域复位的黑盒输入 |
|   enable    |    Bool     |       Nothing       | 连接时钟域使能的黑盒输入 |

`mapCurrentClockDomain`和`mapClockDomain`有着近乎相同的参数, 唯一区别是没有`clockDomain`。

例如：

```Scala
class MyRam(clkDomain: ClockDomain) extends BlackBox {

    val io = new Bundle {
        val clkA = in Bool()
        ...
        val clkB = in Bool()
        ...
    }

    //A时钟映射在给定时钟域
    mapClockDomain(clkDomain, io.clkA)
    //B时钟映射在当前时钟域
    mapCurrentClockDomain(io.clkB)
}
```

### 六、io前缀(io prefix)

为了避免在黑盒的IO口书写“io_”前缀, 也可以使用`noIoPrefix()`函数, 如下所示：

```Scala
class Ram_1w_1r(wordWidth: Int, wordCount: Int) extends BlackBox {
    
    val generic = new Generic {
        val wordCount = Ram_1w_1r.this.wordCount
        val wordWidth = Ram_1w_1r.this.wordWidth
    }

    val io = new Bundle {
        val clk = in Bool()
        
        val wr = new Bundle {
            val en   = in Bool()
            val addr = in UInt(log2Up(_wordCount) bits)
            val data = in Bits(_wordWidth bits)
        }
        val rd = new Bundle {
            val en   = in Bool()
            val addr = in UInt(log2Up(_wordCount) bits)
            val data - out Bits(_wordWidth bits)
        }
    }

    noIoPrefix()

    mapCurrentClockDomain(clock=io.clk)
}
```

### 七、重命名黑盒的所有io(Rename all io of a blackbox)

`BlackBox`或`Component`中的IOs能通过`addPrePopTask`在编译期间重命名。这个函数接受一个在编译期间无变量的函数, 并且这个函数在增加重命名通道的时候很有用, 如下所示：

```Scala
class MyRam() extends Blackbox {

    val io = new Bundle {
        val clk = in Bool()
        val portA = new Bundle {
            val cs   = in Bool()
            val rwn  = in Bool()
            val dIn  = in Bits(32 bits)
            val dOut = out Bits(32 bits)
        }
        val portB = new Bundle {
            val cs   = in Bool()
            val rwn  = in Bool()
            val dIn  = in Bits(32 bits)
            val dOut = out Bits(32 bits)
        }
    }
    
    //映射时钟
    mapCurrentClockDomain(io.clk)

    //移除io_ prefix
    noIoPrefix()

    //用函数给黑盒中所有信号重命名
    private def renameIO(): Unit = {
        io.flatten.foreach(bt => {
            if(bt.getName().contains("portA")) bt.setName(bt.getName().replace("portA_", "") + "_A")
            if(bt.getName().contains("portB")) bt.setName(bt.getName().replace("portB_", "") + "_B")
        })

        //在创建模块后执行重命名函数
        addPrePopTask(() => renameIO())
    }
}
//这段代码会产生如下信号名字:
//    clk
//    cs_A, rwn_A, dIn_A, dOut_A
//    cs_B, rwn_B, dIn_B, dOut_B
```

### 八、添加RTL源(Add RTL source)

通过`addRTLPath()`你可以把黑盒和RTL源代码联系起来。在生成SpinalHDL代码后可以调用函数`mergeRTLSource`把这些源代码整合起来。

```Scala
class MyBlackBox() extends Blackbox {

    val io = new Bundle {
        val clk     = in Bool()
        val start   = in Bool()
        val dIn     = in Bits(32 bits)
        val dOut    = out Bits(32 bits)
        val ready   = out Bool()
    }

    //映射时钟
    mapCurrentClockDomain(io.clk)

    //移除io_前缀
    noIoPrefix()

    //增加所有rtl依赖
    addRTLPath("./rtl/RegisterBank.v")                      //增加verilog文件
    addRTLPath(s"./rtl/myDesign.vhd")                       //增加VHDL文件
    addRTLPath(s"${sys.env("MY_PROJECT")}/myTopLevel.vhd")  //使用环境变量MY_PROJECT(System.getenv("MY_PROJECT"))
}

...

val report = SpinalVhdl(new MyBlackBox)
report.mergeRTLSource("mergeRTL")       //把所有rtl源代码整合进mergeRTL.vhd和mergeRTL.v文件
```

### 九、VHDL——非数字类型(VHDL-No numeric type)

如果你只想在黑盒模块中使用`std_logic_vector`, 你可以给黑河增加`noNumericType`标签。

```Scala
class MyBlackBox() extends BlackBox {
    val io = new Bundle {
        val clk         = in Bool()
        val increment   = in Bool()
        val initValue   = in UInt(8 bits)
        val counter     = out UInt(8 bits)
    }

    map CurrentClockDomain(io.clk)

    noIoPrefix()

    addTag(noNumericType)       //只有std_logic_vector
}
```

上述代码会生成以下VHDL：

```VHDL
component MyBlackBox is
  port(
    clk       : in  std_logic;
    increment : in  std_logic;
    initValue : in  std_logic_vector(7 downto 0);
    counter   : out std_logic_vector(7 downto 0)
  );
end component;
```
