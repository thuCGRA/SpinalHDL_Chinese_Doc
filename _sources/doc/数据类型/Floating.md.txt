## Floating

> 注意：SpinalHDL定点数只是部分支持并部分经过测试的, 如果你发现它有任何bug, 或者你认为有些函数丢失了, 请创建一个Github issue。并且, 请不要在代码中用未在doc中指出的特征。

### 一、描述(Description)

`Floating`类型对应于IEEE-754编码规则。第二种叫做`RecFloating`的类型能通过对浮点数的重编码简化一些IEEE-754浮点数的边缘案例。

它由1 bit符号位, 指数区域和底数区域组成。不同区域的宽度定义在IEEE-754或者de-facto标准中。

使用浮点数时需要先进行如下操作：

```Scala
import spinal.lib.experimental.math._
```

**IEEE-754浮点格式(IEEE-754 floating format)**

数字会根据IEEE-754编码：https://en.wikipedia.org/wiki/IEEE_754

**浮点形式重编码(Recoded floating format)**

因为IEEE-754在非正规(denormalized)数字和特殊值的时候有一些不方便的地方, Berkeley提出了另一种浮点数值的编码方式。

为了能将非正规的数值规范(normalized)地处理, 底数经过了一些修改。

指数部分比IEEE-754多1 bit。

符号位没有改变。

例子可以在这里看到：https://github.com/ucb-bar/berkeley-hardfloat/blob/master/README.md

**零(Zero)**

指数字段的三个前导零被设置为0来代表0的编码。

**非正规数值(Denormalized values)**

非正规的数值也会编码成一般的浮点数, 底数被移位使第一位隐藏, 指数编码为107(十进制)加bit值是1的最高位的序号。

**规范数值(Normalized values)**

重编码的底数和IEEE-754标准的底数完全一致, 指数编码为130(十进制)加最初的指数值。

**无穷大(Infinity)**

重编码的底数值不用关心, 重编码的指数值的最高3 bits值是6(十进制), 剩余部分不用关心。

**NaN**

重编码的规范化底数部分与IEEE-754一致, 重编码的指数部分最高3 bits是7(十进制), 剩余部分不用关心。

### 二、声明(Declaration)

声明浮点数的语句如下所示：

**IEEE-754数字(IEEE-754 Number)**

|                      语法                      |                  描述                  |
| :--------------------------------------------: | :------------------------------------: |
| Floating(exponentSize: Int, mantissaSize: Int) | 带有标准指数和底数长度的IEEE-754浮点数 |
|                  Floating16()                  |          半精度IEEE-754浮点数          |
|                  Floating32()                  |          单精度IEEE-754浮点数          |
|                  Floating64()                  |          双精度IEEE-754浮点数          |
|                 Floating128()                  |          四精度IEEE-754浮点数          |

|                       语法                        |                  描述                  |
| :-----------------------------------------------: | :------------------------------------: |
| RecFloating(exponentSize: Int, mantissaSize: Int) | 带有标准指数和底数长度的IEEE-754浮点数 |
|                  RecFloating16()                  |           重编码半精度浮点数           |
|                  RecFloating32()                  |           重编码单精度浮点数           |
|                  RecFloating64()                  |           重编码双精度浮点数           |
|                 RecFloating128()                  |           重编码四精度浮点数           |

### 三、操作符(Operators)

以下所`Floating`和`RecFloating`类型所支持的操作符：

**类型转换(Type cast)**    ##有问题, 前后不一##

|       操作符        |              描述              |      返回类型       |
| :-----------------: | :----------------------------: | :-----------------: |
|      x.asBits       |        二进制转换为Bits        |   Bits(w(x) bits)   |
|      x.asBools      |         转换为Bool数组         | Vec(Bool, width(x)) |
| x.toUInt(size: Int) | 返回对应的UInt(自带truncation) |        UInt         |
| x.toSInt(size: Int) | 返回对应的SInt(自带truncation) |        SInt         |
|     x.fromUInt      | 返回对应的UFix(自带truncation) |        UFix         |
|     x.fromUInt      | 返回对应的SFix(自带truncation) |        SFix         |
