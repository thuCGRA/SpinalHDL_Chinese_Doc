## VHDL和Verilog生成(VHDL and Verilog generation)

### 一、从一个SpinalHDL组件生成VHDL或Verilog代码(Generate VHDL and Verilog from a SpinalHDL Component)

为了从一个SpinalHDL组件生成VHDL, 用户需要在Scala的`main`函数中调用`SpinalVhdl(new YourComponent)`。 生成Verilog代码的声明是一致的, 只是需要将上述声明中的`SpinalVHD`转换为`SpinalVerilog`。

```
import spinal.core._

// 一个简单的组件定义.
class MyTopLevel extends Component {
  // 定义一些IO信号。
  val io = new Bundle {
    val a = in  Bool()
    val b = in  Bool()
    val c = out Bool()
  }

  // 定义一些异步逻辑
  io.c := io.a & io.b
}

// 这是生成VHDL或是Verilog的主函数
object MyMain {
  def main(args: Array[String]) {
    SpinalVhdl(new MyTopLevel)
    SpinalVerilog(new MyTopLevel)
  }
}
```
> **重要：`SpinalVhdl`和`SpinalVerilog`可能需要创建组件类的多个实例, 因此第一个参数不是`component`引用, 而是返回新组件的函数。**

> **重要：`SpinalVerilog`于2016年6月5日开始实施。在后端成功地通过了与VHDL相同的回归测试(RISCV CPU, 多核和流水线mandelbrot, UART RX/TX, 单时钟fifo, 双时钟fifo, 灰色计数器, …)。如果您对这个新的后端有任何问题, 请在Github上发表问题描述。**

1. Scala中的参数化配置(Parametrization from Scala)

   |             声明命名             |         类型          |                                         默值                                         |                           描述                           |
   | :------------------------------: | :-------------------: |:------------------------------------------------------------------------------------: |:------------------------------------------------------: |
   |              `mode`              |      SpinalMode       |                                         null                                          | 设置SpinalHDL的生成模式。可以设置为`VHDL`或者`Verilog`。 |
   |  `defaultConfigForClockDomains`  |   ClockDomainConfig   | RisingEdgeClock <br> AsynchronousReset <br> ResetActiveHigh <br>ClockEnableActiveHigh |            设置所有新时钟域使用的默认时钟配置            |
   | `onlyStdLogicVectorAtTopLevelIo` |        Boolean        |                                        false                                          |             改变所有顶层为`std_logic_vector`             |
   |  `defaultClockDomainFrequency`   | IClockDomainFrequency |                                   UnknownFrequency                                    |                       默认时钟频率                       |
   |        `targetDirectory`         |        String         |                                   Currentdirectory                                    |                    生成文件的存放目录                    |

   指定上述配置的语法如下：
   ```Scala
   SpinalConfig(mode=VHDL, targetDirectory="temp/myDesign").generate(new UartCtrl)

   // 或对于Verilog可以用拓展性更好的格式
   SpinalConfig(
   mode=Verilog,
   targetDirectory="temp/myDesign"
   ).generate(new UartCtrl)
   ```

2. 利用Shell进行参数配置(Parametrization from shell)

   用户同样可以使用命令行进行参数配置：
   ```Scala
   def main(args: Array[String]): Unit = {
   SpinalConfig.shell(args)(new UartCtrl)
   }
   ```
   命令行示例如下：
   ```
   Usage: SpinalCore [options]

   --vhdl
           Select the VHDL mode
   --verilog
           Select the Verilog mode
   -d | --debug
           Enter in debug mode directly
   -o <value> | --targetDirectory <value>
           Set the target directory
   ```

### 二、已生成的VHDL或Verilog(Generated VHDL and Verilog)

最重要的是理解SpinalHDL的RTL描述是如何转换成Verilog/VHDL的：
+ 在Scala中的命名会保存在VHDL/Verilog中
+ Scala中`Component`的结构会保存在VHDL/Verilog中
+ Scala中`When`语句会映射为VHDL/Verilog中的if语句
+ Scala中`switch`语句会映射为VHDL/Verilog中的在所有标准情况下的Case语句

1. 组织结构(Organization)

   当用户使用VHDL生成器, 所有模块会被生成为包含三个部分的单个文件：
   + 包含对所有Enums定义的集合
   + 包含被结构中的单元所使用的函数的集合
   + 包含用户设计中所需的所有组件
   当用户使用VHDL生成器, 所有模块会被生成为包含两个部分的单个文件：
   + 所有使用的数字定义
   + 用户设计中所需的所有模块

2. 组合逻辑(Combinational logic)

   Scala：
   ```Scala
   class TopLevel extends Component {
       val io = new Bundle {
           val cond           = in  Bool()
           val value          = in  UInt(4 bits)
           val withoutProcess = out UInt(4 bits)
           val withProcess    = out UInt(4 bits)
       }
       io.withoutProcess := io.value
       io.withProcess := 0
       when(io.cond) {
           switch(io.value) {
               is(U"0000") {
                   io.withProcess := 8
               }
               is(U"0001") {
                   io.withProcess := 9
               }
               default {
                   io.withProcess := io.value+1
               }
           }
       }
   }
   ```
   VHDL：
   ```Vhdl
   entity TopLevel is
       port(
           io_cond : in std_logic;
           io_value : in unsigned(3 downto 0);
           io_withoutProcess : out unsigned(3 downto 0);
           io_withProcess : out unsigned(3 downto 0)
       );
   end TopLevel;

   architecture arch of TopLevel is
   begin
       io_withoutProcess <= io_value;
       process(io_cond,io_value)
       begin
       io_withProcess <= pkg_unsigned("0000");
       if io_cond = '1' then
           case io_value is
               when pkg_unsigned("0000") =>
                   io_withProcess <= pkg_unsigned("1000");
               when pkg_unsigned("0001") =>
                   io_withProcess <= pkg_unsigned("1001");
               when others =>
                   io_withProcess <= (io_value + pkg_unsigned("0001"));
           end case;
       end if;
       end process;
   end arch;
   ```
3. 时序逻辑

   Scala：
   ```Scala
   class TopLevel extends Component {
       val io = new Bundle {
           val cond   = in Bool()
           val value  = in UInt (4 bits)
           val resultA = out UInt(4 bits)
           val resultB = out UInt(4 bits)
   }

       val regWithReset = Reg(UInt(4 bits)) init(0)
       val regWithoutReset = Reg(UInt(4 bits))

       regWithReset := io.value
       regWithoutReset := 0
       when(io.cond) {
           regWithoutReset := io.value
       }

       io.resultA := regWithReset
       io.resultB := regWithoutReset
   }
   ```
   VHDL：
   ```Vhdl
   entity TopLevel is
       port(
           io_cond : in std_logic;
           io_value : in unsigned(3 downto 0);
           io_resultA : out unsigned(3 downto 0);
           io_resultB : out unsigned(3 downto 0);
           clk : in std_logic;
           reset : in std_logic
       );
   end TopLevel;

   architecture arch of TopLevel is

       signal regWithReset : unsigned(3 downto 0);
       signal regWithoutReset : unsigned(3 downto 0);
   begin
       io_resultA <= regWithReset;
       io_resultB <= regWithoutReset;
       process(clk,reset)
       begin
       if reset = '1' then
           regWithReset <= pkg_unsigned("0000");
       elsif rising_edge(clk) then
           regWithReset <= io_value;
       end if;
       end process;

       process(clk)
       begin
       if rising_edge(clk) then
           regWithoutReset <= pkg_unsigned("0000");
           if io_cond = '1' then
               regWithoutReset <= io_value;
           end if;
       end if;
       end process;
   end arch;
   ```
### 三、VHDL与Verilog属性(VHDL and Verilog attributes)
    
在某些情况下, 对设计中的某些信号设置一些属性来修正他们最终的综合结果是很有用的。

为了实现上述效果, 用户可以对设计中的信号或者存储调用如下函数：

|            句式             |                        描述                         |
| :-------------------------: | :-------------------------------------------------: |
|    `addAttribute(name)`     |    添加一个Boolean属性并对给定的`name`设定为true    |
| `addAttribute(name, value)` | 添加一个String属性并对给定的`name`集合设置为`value` |

例子：
```Scala
val pcPlus4 = pc + 4
pcPlus4.addAttribute("keep")
```
生成的VHDL解释：
```Vhdl
attribute keep : boolean;
signal pcPlus4 : unsigned(31 downto 0);
attribute keep of pcPlus4: signal is true;
```
生成的Verilog解释：
```Verilog
(* keep *) wire [31:0] pcPlus4;
```
