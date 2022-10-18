## Scala指导手册(Scala Guide)

### 简介(Basics)

Scala是一款功能强大的编程语言, 它的产生受到了很多其他独特语言的影响, 但这些语言常常不为大多数程序员所使用。这也会阻碍scala新人们对scala概念和它背后的设计抉择。

下面会介绍Scala, 并常识为新人们提供学习SpinalHDL的基本Scala语法。


### 一、基础

1. 数据类型(Types)
    
   在Scala中, 有五种主要类型：
 
   | 数据类型 |     举例      |         描述          |
   | :------: | :-----------: | :-------------------: |
   | Boolean  |  true, false  |
   |   Int    |    3, 0*32    |    32bits integer     |
   |  Float   |     3.14f     | 32bits floating point |
   |  Doublt  |     3.14      | 64bits floating point |
   |  String  | "Hello world" |     UTF-16 string     |

2. 变量(Variables)

   在Scala中, 使用`var`关键字来定义变量, 即`val/var 变量名: 变量类型 = 初始值`。var在Scala中是变量的声明, 而val是常量的声明, 但是在SpinalHDL中可以用`:=`对va量赋值：

   ```Scala
   var number : Int = 0
   number = 6
   number += 4
   println(number) //10
   ``` 

   Scala能够自动推断数据类型, 在给变量赋初值的时候你不必指明数据类型。

   ```Scala
   var number = 0 //number的数据类型在编译过程中推断为Int
   ```

   然而, 在Scala中使用var并不常见。取而代之的, 由`val`定义的常量更加常用：

   ```Scala
   val two = 2
   val three = 3
   val six = two * three
   ```

3. 函数(Functions)

   例如, 如果想定义一个两个参数和大于0时返回`true`的函数, , 你可以这样做：

   ```Scala
   def sumBiggerThanZero(a: Float, b: Float): Boolean = {
       return (a + b) > 0
   }
   ```

   在调用函数的时候, 用以下方式：

   ```Scala
   sumBiggerThanZero(2.3f, 5.4f)
   ```

   你也可以通过加入参数的名字来给特定参数赋值,  这在参数较多的函数中十分有效：

   ```Scala
   sumBiggerThanZero(
       a = 2.3f,
       b = 5.4f
   )
   ```

   1. 返回(Return)

       `return`关键字并不是必要的, 当缺省时, Scala会把最后一行函数作为返回值的声明：

       ```Scala
       def sumBiggerThanZero(a: Float, b: Float): Boolean = {
           (a + b) > 0
       }
       ```

   2. 返回类型推断(Return type inferation)

       Scala能够自动推断返回类型, 无须声明：

       ```Scala
       def sumBiggerThanZero(a: Float, b: Float) = {
           (a + b) > 0
       }
       ```

   3. 花括号(Cuely braces)

       如果你的函数内只有一条声明, Scala函数不需要花括号：

       ```Scala
       def sumBiggerThanZero(a: Float, b: Float) = (a + b) > 0
       ```

   4. 无返回值的函数(Function that returns nothing)

       如果想要让一个函数不返回任何值, 返回类型应该设置为`Uint`, 这等价于C/C++中的`void`类型

       ```Scala
       def printer(): Uint = {
           println("1234")
           println("5678")
       }
       ```

   5. 参数的默认值(Argument default values)

       你可以给函数中的每个参数指定一个默认值：

       ```Scala
       def sumBiggerThanZero(a: Float, b: Float = 0.0f) = {
           (a + b) > 0
       }
       ```

   6. 应用(Apply)

       名为`apply`的函数是一类特殊的函数, 你可以不输入名字就能调用他们：

       ```Scala
       class Array() {
           def apply(index: Int): Int = index + 3
       }

       val array = new Array()
       val value = array(4) //array(4)指一个数组, apply(4)的返回值所7
       ```

       这种概念也适用于Scala中的`object`(静态)

       ```Scala
       object MajorityVote {
           def apply(value: Int): Int = ...
       }

       val value = MajorityVote(4) //会调用MajorityVote.apply(4)
       ```

4.  对象(Object)

   在Scala中, 没有`static`关键字, 取而代之的是`object`。每一个定义在`object`中的定义都是静态的。

   下面这个例子定义了一个叫做`pow2`的静态函数, 该函数以浮点数作为参数输入, 返回值类型也是浮点数。

   ```Scala
   object MathUtils {
       def pow2(value: Float): Float = value * value
   }
   ```

   然后可以通过以下书写方式调用：

   ```Scala
   MathUtils.pow2(42.0f)
   ```

5.  入口点(Entry point(main))

   Scala程序的入口点(主函数)应该作为一个名为`main`的函数定义在一个对象当中

   ```Scala
   object MyTopLevelMain{
       def main(args: Array[String]) {
           println("Hello World")
       }
   }
   ```

6.  类(Class)

   类的语法与Java非常相似。假设你想要定义一个把三个浮点数(r, g, b)作为结构体参数的`Color`类：

   ```Scala
   class Color(r: Float, g: Float, b: Float) {
       def getGrayLevel(): Float = r * 0.3f + g * 0.4f + b * 0.4f
   }
   ```

   之后, 从上一个例子中实例化类并使用`getGrayLevel`函数：

   ```Scala
   val blue= new Color(0, 0, 1)
   val grayLevelOfBlue = blue.getGrayLevel()
   ```

   需要注意的是, 如果你想从类的外部访问类中的结构体参数, 这个结构体参数应该被定义成`val`类型：

   ```Scala
   class Color(val r: Float, val g: Float, val b: Float) { ... }
   ...
   val blue = new Color(0, 0, 1)
   val redLevelOfBlue = blue.r
   ```

   1. 继承(Inheritance)

       作为例子, 假设你想要定义两个类, `Rectangle`和`Square`, 并延伸出类`Shape`：

       ```Scala
       class Shape {
           def getArea(): Float
       }

       class Square(sideLength: Float) extends Shape {
           override def getArea() = sideLength * sideLength
       }

       class Rectangle(width: Float, height: Float) extends Shape {
           override def getArea() = width * height
       }
       ```

   2.  用例类(Case class)

       用例类(Case class)是声明类的另一种方式。

       ```Scala
       case class Rectangle(width: Float, height:; Float) wxtends Shape {
           override def getArea() = width * height
       }
       ```

       但是`case class`和`class`之间有一些区别：

       + 用例类不需要`new`关键字来实例化
       + 用例类中的结构参数外界是可访问的, 不用把他们定义成`val`

       在SpinalHDL中, 这解释了代码约束背后的原因：一般来说更推荐用`case class`而不是`class`, 这样能够减少打字数并且一致性更好

7. 模板/类型参数化(Templates/Type parameterization)

   假设你想要设计一组给定数据类型的类, 在这个背景下你需要给类提供一个参数类型：

   ```Scala
   class Queue[T]() {
       def push(that: T) : Unit = ...
       def pop(): T = ...
   }
   ```

   如果你想要把`T`类型约束成给定类型的子类(例如`Shape`), 你可以用`<: Shape`语句实现。`<:`在Scala中表示给类型添加上界, 表示泛型参数必须要从该类(或本身)继承：

   ```Scala
   class Shape() {
       def getArea(): Float
   }
   class Rectangle() extends Shape { ... }

   class Queue[T <: Shape]() {
       def push(that: T): Unit = ...
       def pop(): T = ...
   }
   ```

   对于函数也是同理：

   ```Scala
   def doSomething[T <: Shape](shape: T): Something = { shape.getArea() }
   ```

### 二、代码约束(Coding conventions)

1. 介绍(Introduction)

   SpinalHDL中的代码约束与Scala Style Guide中描述的一样, 有一些额外的实用细节和案例会在下一章中讲解

   https://docs.scala-lang.org/style/

2. 类 vs 用例类(class vs case class)

   当你定义了一个`Bundle`或是一个`Component`, 声明成用例类(case class)更好, 理由如下：

   + 避免使用`new`关键字, 在某些条件下, 永远不用再使用它总比有时会用到要强。
   + `case class`提供了`clone`函数。后者在SpinalHDL中十分有用, 因为SpinalHDL中会需要克隆`Bundle`, 例如, 当你定义一个新的`Reg`或是一个新的`Stream`之类的东时。
   + 结构体参数在外部是直接可见的。

   1. [用例]类

       所有类的首字母都应该大写

       ```Scala
       class Fifo extends Component {

       }

       class Counter extends Area {

       }

       case class Color extends Bundle {

       }
       ```

    2. 伴生对象

       一个伴生对象(companion object)应该首字母大写

       ```Scala
       object Fifo {
           def apply(that: Stream[Bits]): Stream[Bits] = { ... }
       }

       object MajorityVote {
           def apply(that: Bits): UInt = { ... }
       }
       ```

       这种规则有个例外就是, 当伴生对象被用作函数(里面只有`apply`), 并且这些`apply`函数不产生硬件电路：

       ```Scala
       object log2 {
           def apply(value: Int): Int = { ... }
       }
       ```

3. 函数(Function)

   一个函数总以小写字母开头：

   ```Scala
   def sinTable = (0 until sampleCount).map(sampleIndex => {
       val sinValue = Math.sin(2 * Math.PI * sampleIndex / sampleCount)
       S((sinValue * ((1 << resolutionWidth) / 2 - 1)).toInt, resolutionWidth bits)
   })

   val rom = Mem(SInt(resolutionWidth bits), initialContent = sinTable)
   ```

4. 实例(Instances)

   类的实例应该总以小写字母开头

   ```Scala
   val fifo = new Fifo()
   val buffer = Reg(Bits(8 bits))
   ```

5. if/when

   Scala的`if`和SpinalHDL的`when`都一般以如下方式书写：

   ```Scala
   if(cond) {
       ...
   } else if(cond) {
       ...
   } else {
       ...
   }

   when(cond) {
       ...
   }.elseWhen(cond) {
       ...
   }.otherwise {
       ...
   }
   ```

   例外：

   + 可以省略`otherwise`前的点
   + 如果可以增强代码的可读性, 可以把`if`/`when`的声明写在一行

6. switch

   SpinalHDL`switch`一般应该以如下方式书写：

   ```Scala
   switch(value) {
       is(key) {

       }
       is(key) {

       }
       default {

       }
   }
   ```

   如果可以增强代码的可读性, 可以把`is`/`default`声明写在一行

7. 参数(Parameters)

   推荐把`Component`/`Bundle`的参数打包到一个用例(case)中, 如下例中`RgbConfig`, 因为：

   + 更容易携带/操作以对设计进行配置
   + 更好的维护性

   ```Scala
   case class RgbConfig(rWidth: Int, gWidth: Int, bWidth: Int) {
       def getWidth = rWidth + gWidth + bWidth
   }

   case class Rgb(c: RgbConfig) extends Bundle {
       val r = UInt(c.rWidth bits)
       val g = UInt(c.gWidth bits)
       val b = UInt(c.bWidth bits)
   }
   ```

   但这并不适用于所有情况。例如, 在FIFO中, 把`dataType`参数和`depth`参数打包在一起所不合理的, 因为一般来说, `dataType`所于设计相关的参数, 而`depth`所于配置相的参数

   ```Scala
   class Fifo[T <: Data](dataType: T, depth: Int) extends Component {

   }
   ```

### 三、交互

1. 简介(Introduction)

   事实上, SpinalHDL不是一门语言, 更像是常规的Scala库。第一次看到它可能会觉得很奇怪, 但等你用久了就会发现SpinalHDL很好地结合了RTL和Scala。

   你可以通过SpinalHDL库用整个Scala世界帮助你描述硬件电路, 但是为了恰到好处的做到这件事, 你需要理解SpinalHDL如何于Scala交互。

2. SpinalHDL是怎样在API背后工作的(How SpinalHDL works behind the API)

   当你执行SpinalHDL硬件描述, 每当你使用SpinalHDL函数、操作符或类, 都会在内存中建立一个代表你设计的网表的图。

   之后, 当硬件描述完成(顶层`Component`类的实例化), SpinalHDL会再遍历建立好的网表图, 如果一切就绪, SpinalHDL就会把图冲刷掉并把它构建成VHDL或Verilog文件。

3. 所有都是引用(Everything is a reference)

   例如, 如果你定义了一个接受`Bits`类型参数的Scala函数, 当你调用它, 它会作为引用传递。因此, 如果你在函数内给这个参数赋值, 它对底层`Bits`对象的影响就会像在函数外赋值一样。

4. 硬件类型(Hardware types)

   SpinalHDL中的硬件数据类型所两个事情的结合：

   + 一个给定Scala类型的实例化
   + 那个实例的配置

   例如`Bits(8bits)`是Scala类型`Bits`和它的`8 bits`配置(作为结构体参数)的结合

   **RGB举例**

   我们以一个RGB包举例说明：

   ```Scala
   case class Rgb(rWidth: Int, gWidth: Int, bWidth: Int) extends Bundle {
       val r = UInt(rWidth bits)
       val g = UInt(gWidth bits)
       val b = UInt(bWidth bits)
   }
   ```

   这里的硬件数据类型是Scala`Rgb`类和它的`rWidth`, `gWidth`, `bWidth`参数的结合

   以下是这个例子的使用：

   ```Scala
   // 定义一个Rgb信号
   val myRgbSignal = Rgb(5, 6, 5)

   //定义另一个与前一个相同数据类型的Rgb信号
   val myRgbCloned = cloneOf(myRgbSignal)
   ```

   上述代码生成的Verilog如下所示(位宽有所调整)：

   ```Verilog
   wire       [7:0]    myRgbSignal_r;
   wire       [7:0]    myRgbSignal_g;
   wire       [7:0]    myRgbSignal_b;
   wire       [7:0]    myRgbCloned_r;
   wire       [7:0]    myRgbCloned_g;
   wire       [7:0]    myRgbCloned_b;
   ```

   你也可以用函数来定义各种各样的类别(typedef)：

   ```Scala
   //定义一个类别函数
   def myRgbTypeDef = Rgb(5, 6, 5)

   //使用这个类别加工函数产生Rgb信号
   val myRgbFromTypeDef = myRgbTypeDef
   ```

5. 产生的RTL中信号的名称(Names of signals in the generated RTL)

   在给产生的RTL中命名信号时, SpinalHDL会用Java映射遍历各个模块的层次, 收集所有存储在类属性中的引用, 并用他们的属性名命名他们。

   每个在函数内定义的本地信号名字会丢失, 如下所示：

   ```Scala
   def myFunction(arg: UInt) {
       val temp= arg + 1 //你无法在产生的RTL中取回取回temp信号
       return temp
   }

   val value = myFunction(U"000001") + 42
   ```

   如果你想要在产生的RTL中保留内部变量的名字, 可以用`Area`：

   ```Scala
   def myFunction(arg: UInt) = new Area {
       val temp = arg + 1 //你能在产生的RTL中取回取回temp信号
   }

   val myFunctionCall = myFunction(U"000001") //会随着myFunctionCall.temp产生temp信号
   val value = myFunctionCall.temp + 42
   ```

   上述代码会产生如下Verilog：
   
   ```Verilog
   wire       [5:0]    myFunctionCall_temp;
   wire       [5:0]    value;

   assign myFunctionCall_temp = (6'h01 + 6'h01);
   assign value = (myFunctionCall_temp + 6'h2a);
   ```

6. Scala用来加工, SpinalHDL用来描述硬件(Scala is for elaboration, SpinalHDL for hardware description)

   例如, 如果你用Scala的for循环产生硬件电路, 它会产生展开后的VHDL/Verilog形式的结果

   还有, 如果你想要一个常量, 你不应该用SpinalHDL硬件语句, 而应该用Scala的。例如：

   ```Scala
   //这是错的, 因为你不能用硬件Bool作为结构参数, 这会导致层次违例
   class SubComponent(activeHigh: Bool) extends Component {
       // ...
   }

   //这是对的, 你可以用Scala的任何语句来参数化硬件电路
   class SubComponent(activeHigh: Boolean) extends Component {

   }
   ```

7. Scala的细化能力(if, for, 和函数化编程)(Scala elaboration capabilities)

   所有的Scala语法都能用来描述硬件设计, 例如, Scala的`if`语句能用来使能电路的生成：

   ```Scala
   val counter= Reg(UInt(8 bits))
   counter := counter + 1
   if(generateAClearWhenHit42) {   //加工测试, 好比是是否生成vhdl
       when(counter === 42) {      //硬件测试
           counter := 0
       }
   }
   ```

   对于Scala的`for`循环也是同理：

   ```Scala
   val value = Reg(Bits(8 bits))
   when(something) {
       //通过使用Scala的for循环置位每一bit
       for(idx <- 0 to 7) {
           value(idx) := True
       }
   }
   ```

   上述代码生成的Verilog如下所示：

   ```Verilog
   assign when_Main_l17 = something;
   always @(posedge clk) begin
       if(when_Main_l17) begin
           value[0] <= 1'b1;
           value[1] <= 1'b1;
           value[2] <= 1'b1;
           value[3] <= 1'b1;
           value[4] <= 1'b1;
           value[5] <= 1'b1;
           value[6] <= 1'b1;
           value[7] <= 1'b1;
       end
   end
   ```

   同样的, SpinalHDL类型也能使用函数化的编程技术(最好之后能补充通配符的用法)

   ```Scala
   val values = Vec(Bits(8 bits), 4)

   val valuesAre42 = values.map(_===42)
   val valuesAreAll42 = valuesAre42.reduce(_&&_)

   val valuesAreEqualToTheirIndex = values.zipWithIndex.map{ case(value, i) => value === i }
   ```

   上述代码生成的Verilog如下所示：

   ```Verilog
   wire       [7:0]    values_0;
   wire       [7:0]    values_1;
   wire       [7:0]    values_2;
   wire       [7:0]    values_3;
   wire                valuesAre42_0;
   wire                valuesAre42_1;
   wire                valuesAre42_2;
   wire                valuesAre42_3;
   wire                valuesAreAll42;
   wire                valuesAreEqualToTheirIndex_0;
   wire                valuesAreEqualToTheirIndex_1;
   wire                valuesAreEqualToTheirIndex_2;
   wire                valuesAreEqualToTheirIndex_3;

   assign valuesAre42_0 = (values_0 == 8'h2a);
   assign valuesAre42_1 = (values_1 == 8'h2a);
   assign valuesAre42_2 = (values_2 == 8'h2a);
   assign valuesAre42_3 = (values_3 == 8'h2a);
   assign valuesAreAll42 = (((valuesAre42_0 && valuesAre42_1) && valuesAre42_2) && valuesAre42_3);
   assign valuesAreEqualToTheirIndex_0 = (values_0 == 8'h0);
   assign valuesAreEqualToTheirIndex_1 = (values_1 == 8'h01);
   assign valuesAreEqualToTheirIndex_2 = (values_2 == 8'h02);
   assign valuesAreEqualToTheirIndex_3 = (values_3 == 8'h03);
   ```
