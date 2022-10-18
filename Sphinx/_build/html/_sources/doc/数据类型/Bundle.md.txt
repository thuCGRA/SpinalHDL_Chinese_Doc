## Bundle

### 一、描述(Description)

`Bundle`是一种符合类型, 它定义了一组命名了的信号(这些信号是SpinalHDL的基础类型)

`Bundle`类型能用来搭建数据结构, 总线和接口的模型。

### 二、声明(Declaration)

声明bundle的语句如下：

```Scala
case class myBundle extends Bundle {
    val bundleItem0 = AnyType
    val bundleItem1 = AnyType
    val bundleItemN = AnyType
}
```

例如, 一个包含颜色信号的bundle可以如下定义：

```Scala
case class Color(channelWidth: Int) extends Bundle {
    val r, g, b = UInt(channelWidth bits)
}
```

你可以在“SpinalHDL用例”当中找到APB3的定义

1. 条件信号(Conditional signals)

   `Bundle`中的信号可以根据条件选择定义。如下举例, 除非`datawidth`大于0, `mybundle`中不会有`data`信号：

   ```Scala
   case class myBundle(dataWidth: Int) extends Bundle {
       val data = (dataWidth > 0) generate (UInt(dataWidth bits))
   }
   ```

### 三、操作符(Operators)

下述是`Bundle`可用的操作符：

1. 对比操作(Comparison)

   | 操作符 |  描述  | 返回类型 |
   | :----: | :----: | :------: |
   | x===y  |  相等  |   Bool   |
   | x=/=y  | 不相等 |   Bool   |

   ```Scala
   val color1 = Color(8)
   color1.r := 0
   color1.g := 0
   color1.b := 0

   val color2 = Color(8)
   color2.r := 0
   color2.g := 0
   color2.b := 0

   myBool := color1 === color2
   ```

2. 类型转换(Type cast)

   |  操作符  |       描述       |    返回类型     |
   | :------: | :--------------: | :-------------: |
   | x.asBits | 二进制转换成Bits | Bits(w(x) bits) |

   ```Scala
   val color1 = Color(8)
   val myBits := color1.asBits
   ```

   Verilog:

   ```Verilog
   wire       [7:0]    color1_data;
   wire       [7:0]    myBits;

   assign myBits = color1_data;
   ```

   bundle的元素在二进制串中按照定义的顺序排列, 因此`r`在`color1`中`myBits`(LSB)中的0-8, 之后按顺序依次所`g`和`b`

3. 把Bits转换成Bundle(Convert Bits back to Bundle)

   `.assignFromBits`操作符相当于`.asBits`的相反操作。

   |           操作符            |                      描述                      | 返回类型 |
   | :-------------------------: | :--------------------------------------------: | :------: |
   |     x.assignFromBits(y)     |            把Bits(y)转换成Bundle(x)            |   UInt   |
   | x.assignFromBits(y, hi, lo) | 把Bits(y)转换成Bundle(x), 并带有high/low的边界 |   UInt   |

   下例把一个叫做CommonDataBus的Bundle存进环形buffer(3rd party memory)中, 之后读出Bits并把他们转换回CommonDataBus。

   ![CommonDataBus](../../img/CommonDataBus.png)

   ```Scala
   case class TestBundle () extends Component {
       val io = new Bundle {
           val we      = in    Bool()
           val addrWr  = in    UInt(7 bits)
           val dataIn  = slave (CommonDataBus())

           val addrRd  = in    UInt(7 bits)
           val dataOut = master(CommonDataBus())
       }

       val mm = Ram3rdParty_1w_1rs (
           G_DATA_WIDTH = io.dataIn.getBitsWidth,
           G_ADDR_WIDTH = io.addrWr.getBitsWidth,
           G_VENDOR     = "Intel_Arria10_M20K")

       mm.io.clk_in    := clockDomain.readClockWire
       mm.io.clk_out   := clockDomain.readClockWire
       
       mm.io.we        := io.we
       mm.io.addr_wr   := io.addrWr.asBits
       mm.io.d         := io.dataIn.asBits

       mm.io.addr_rd   := io.addrRd.asBits
       io.dataOut.assignFromBits(mm.io.q)
   }
   ```

### 四、IO类型指导(IO Element direction)

当你在模块的IO定义中定义一个`Bundle`, 你需要指定它的方向。

1. in/out

   如果bundle的所有元素方向相同, 可以使用`in(MyBundle())`或者`out(MyBundle())`。

   例如：

   ```Scala
   val io = new Bundle {
       val input  = in (Color(8))
       val output = out(Color(8))
   }
   ```

2. master/slave

   如果你的接口遵从于master/slave拓扑, 你可以使用`IMasterSlave`特性, 之后你需要对函数`def asMaster(): Unit`配置来补全每个来自master方面的元素的方向, 这之后你就可以在I定义中用`master(MyBundle())`和`slave(MyBundle())`语句。

   有些函数, 如同`Flow`类中的`toStream`方法, 会以toXXX的方式定义。这些函数常常由master侧调用。此外, fromXXX函数是为slave侧设计的。通常来说, master侧可用的函数比slave更多。

   例如：

   ```Scala
   case class HandShake(payloadWidth: Int) extends Bundle with IMasterSlave {
       val valid   = Bool()
       val ready   = Bool()
       val payload = Bits(payloadWidth bits)

       //你需要补全这个asMaster函数
       //这个函数应该设置master侧的每个信号的方向
       override def asMaster(): Unit = {
           out(valid, payload)
           in(ready)
       }
   }

   val io = new Bundle {
       val input   = slave(HandShake(8))
       val output  = master(HandShake(8))
   }
   ```

   Verilog:

   ```Verilog
   module MyTopLevel (
       input               io_input_valid,
       output reg          io_input_ready,
       input      [7:0]    io_input_payload,
       output reg          io_output_valid,
       input               io_output_ready,
       output reg [7:0]    io_output_payload
   );

   wire                when_MyTopLevel_l56;

   always @(*) begin
       io_input_ready = 1'b0;
       if(when_MyTopLevel_l56) begin
           io_input_ready = 1'b1;
       end
   end

   always @(*) begin
       io_output_valid = 1'b0;
       if(when_MyTopLevel_l56) begin
           io_output_valid = 1'b1;
       end
   end

   always @(*) begin
       io_output_payload = 8'h0;
       if(when_MyTopLevel_l56) begin
           io_output_payload = 8'h23;
       end
   end

   assign when_MyTopLevel_l56 = (io_output_ready && io_input_valid);

   endmodule

   ```
