## SpinalEnum

### 一、描述(Description)

`Enumeration`类型代表一组值组成的列表

### 二、声明(Declaration)

枚举数据类型的声明方式如下：

```Scala
object Enumeration extends SpinalEnum {
    val element0, element1, ..., elementN = newElement()
}
```

对于上述例子, 使用默认的编码方式。原生的枚举类型用于VHDL, 二进制编码用于Verilog。

能够通过下述方式强制使用枚举编码：

```Scala
object Enumeration extends SpinalEnum(defaultEncoding=encodingOfYourChoice) {
    val element0, element1, ..., elementN = newElement()
}
```

> 备注：如果想为给定的模块定义枚举类型的输入/输出, 应该用如下方式定义：`in(MyEnum())`或者`out(MyEnum)`

1. 编码(Encoding)

   SpinalHDL支持下述的枚举编码方式：

   |     编码方式     |      Bit位宽       |                   描述                   |
   | :--------------: | :----------------: | :--------------------------------------: |
   |      native      |                    |    这是默认编码方式, 使用VHDL枚举系统    |
   | binarySequential | log2Up(stateCount) | 以声明的顺序使用Bits存储状态(值从0到n-1) |
   |   binaryOneHot   |     stateCount     | 使用Bits来存储状态, 每个bit对应一个状态  |

   定制化的编码方式能用静态和动态的方式实现：

   ```Scala
   /*
    *静态编码方式
    */
   object MyEnumStatic extends SpinalEnum {
       val e0, e1, e2, e3 = newElement()
       defaultEncoding = SpinalEnumEncoding("staticEncoding")(
           e0 -> 0,
           e1 -> 2,
           e2 -> 3,
           e3 -> 7
       )

   /*
    *用函数动态编码： _ * 2 + 1
    *e.g. :  e0 => 0 * 2 + 1 = 1
    *        e1 => 1 * 2 + 1 = 3
    *        e2 => 2 * 2 + 1 = 5
    *        e3 => 3 * 2 + 1 = 7
    */
       val encoding = SpinalEnumEncoding("dynamicEncoding", _ * 2 + 1)

       object MyEnumDynamic extends SpinalEnum(encoding) {
           val e0, e1, e2, e3 = newElement()
       }
   }
   ```

2. 举例(Example)

   初始化枚举信号并给它赋值：

   ```Scala
   object UartCtrlTxState extends SpinalEnum {
       val sIdle, sStart, sData, sParity, sStop = newElement()
   }

   val stateNext = UartCtrlTxState()
   stateNext := UartCtrlTxState.sIdle

   //你也可以import枚举类型使其元素可视化, 需要放在package里
   import UartCtrlTxState._
   stateNext := sIdle
   ```

   Verilog:

   ```Scala
   localparam UartCtrlTxState_sIdle = 3'd0;
   localparam UartCtrlTxState_sStart = 3'd1;
   localparam UartCtrlTxState_sData = 3'd2;
   localparam UartCtrlTxState_sParity = 3'd3;
   localparam UartCtrlTxState_sStop = 3'd4;

   wire       [2:0]    stateNext;

   `ifndef SYNTHESIS
       reg [55:0] stateNext_string;
   `endif


   `ifndef SYNTHESIS
       always @(*) begin
           case(stateNext)
           UartCtrlTxState_sIdle : stateNext_string = "sIdle  ";
           UartCtrlTxState_sStart : stateNext_string = "sStart ";
           UartCtrlTxState_sData : stateNext_string = "sData  ";
           UartCtrlTxState_sParity : stateNext_string = "sParity";
           UartCtrlTxState_sStop : stateNext_string = "sStop  ";
           default : stateNext_string = "???????";
           endcase
       end
   `endif

   assign stateNext = UartCtrlTxState_sIdle;

   ```

### 三、操作符(Operators)

以下是可供`Enumeration`类型使用的操作符：

1. 比较运算(Comparison)

   |操作符|描述|返回类型|
   |x===y|相等|Bool|
   |x=/=y|不相等|Bool|

   ```Scala
   import UartCtrlTxState._

   val stateNext = UartCtrlTxState()
   stateNext := sIdle

   when(stateNext === sStart) {
       ...
   }

   switch(stateNext) {
       is(sIdle) {
           ...
       }
       is(sStart) {
           ...
       }
       ...
   }
   ```

2. 类型(Types)

   为了使用你的enum, 例如在函数里, 你需要它的类型。

   enum中值的类型(例如sIdle的类型)：

   ```Scala
   spinal.core.SpinalEnumElement[UartCtrlTxState.type]
   ```

   或者等价地, 也可以如下方式

   ```Scala
   UartCtrlTxState.E
   ```

   bundle的类型(例如stateNext的类型)：

   ```Scala
   spinal.core.SpinalEnumCraft[UartCtrlTxState.type]
   ```

   或者等价地, 也可以用如下方式

   ```Scala
   UartCtrlTxState.C
   ```

3. 类型转换(Type cast)

   |         操作符         |       描述       |    返回类型     |
   | :--------------------: | :--------------: | :-------------: |
   |        x.asBits        | 二进制转换为Bits | Bits(w(x) bits) |
   |        x.asUInt        | 二进制转换为UInt | UInt(w(x) bits) |
   |        x.asSInt        | 二进制转换为SInt | SInt(w(x) bits) |
   | e.assignFromBits(bits) |  Bits转换为enum  |    MyEnum()     |

   ```Scala
   import UartCtrlTxState._

   val stateNext = UartCtrlTxState()
   myBits := sIdle.asBits

   stateNext.assignFromBits(myBits)
   ```

   Verilog:

   ```Verilog
   localparam UartCtrlTxState_sIdle = 3'd0;
   localparam UartCtrlTxState_sStart = 3'd1;
   localparam UartCtrlTxState_sData = 3'd2;
   localparam UartCtrlTxState_sParity = 3'd3;
   localparam UartCtrlTxState_sStop = 3'd4;

   wire       [2:0]    myBits;
   wire       [2:0]    stateNext;
   wire       [2:0]    _zz_stateNext;
   reg        [7:0]    counter;
   `ifndef SYNTHESIS
       reg [55:0] stateNext_string;
       reg [55:0] _zz_stateNext_string;
   `endif


   `ifndef SYNTHESIS
       always @(*) begin
           case(stateNext)
               UartCtrlTxState_sIdle : stateNext_string = "sIdle  ";
               UartCtrlTxState_sStart : stateNext_string = "sStart ";
               UartCtrlTxState_sData : stateNext_string = "sData  ";
               UartCtrlTxState_sParity : stateNext_string = "sParity";
               UartCtrlTxState_sStop : stateNext_string = "sStop  ";
               default : stateNext_string = "???????";
           endcase
       end
       always @(*) begin
           case(_zz_stateNext)
               UartCtrlTxState_sIdle : _zz_stateNext_string = "sIdle  ";
               UartCtrlTxState_sStart : _zz_stateNext_string = "sStart ";
               UartCtrlTxState_sData : _zz_stateNext_string = "sData  ";
               UartCtrlTxState_sParity : _zz_stateNext_string = "sParity";
               UartCtrlTxState_sStop : _zz_stateNext_string = "sStop  ";
               default : _zz_stateNext_string = "???????";
           endcase
       end
   `endif

   assign myBits = UartCtrlTxState_sIdle;
   assign _zz_stateNext = myBits;
   assign stateNext = _zz_stateNext;

   ```
