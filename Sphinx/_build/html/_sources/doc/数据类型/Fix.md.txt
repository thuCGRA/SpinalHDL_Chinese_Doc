## UFix/SFix

> 注意：SpinalHDL定点数只是部分支持并部分经过测试的, 如果你发现它有任何bug, 或者你认为有些函数丢失了, 请创建一个Github issue。并且, 请不要在代码中用未在doc中指出的特征。

### 一、描述(Description)

`UFix`和`SFix`类型对应于适用于定点数算法的bit向量。

### 二、声明(Declaration)

声明定点数的语法如下所示：

1. 无符号定点数(Unsigned Fixed-Point)

   |                     语法                     |    整数宽度     |     分辨率     |          max          |  min  |
   | :------------------------------------------: | :-------------: | :------------: | :-------------------: | :---: |
   | UFix(peak: ExpNumber, resolution: ExpNumber) | peak-resolution |  2^resolution  |  2^peak-2^resolution  |   0   |
   |    UFix(peak: ExpNumber, width: BitCount)    |      width      | 2^(peak-width) | 2^peak-2^(peak-width) |   0   |

2. 有符号定点数(Signed Fixed-Point)

   |                     语法                     |     整数宽度      |      分辨率      |           max           |    min    |
   | :------------------------------------------: | :---------------: | :--------------: | :---------------------: | :-------: |
   | SFix(peak: ExpNumber, resolution: ExpNumber) | peak-resolution+1 |   2^resolution   |   2^peak-2^resolution   | -(2^peak) |
   |   SFix(peak: ExpNumber, width: ExpNumber)    |       width       | 2^(peak-width-1) | 2^peak-2^(peak-width-1) | -(2^peak) |

3. 格式(Format)

   SpinalHDL中定点数的格式按照Q notation的格式定义, 详情可见Wikipedia关于Q notation的讲解。

   举例来说, Q8.2表示8+2 bits的定点数, 其中8 bits是整数部分而2 bits是小数部分。如果定点数是有符号数, 需要整数中的1 bit用作符号位。

   分辨率定义为能表征的最小非零数是2的几次幂。

   > 备注：为了减少定点数在表示中2的几次幂时的错误, 在`spinal.core`中有一个数量类型叫做`ExpNumber`, 用来生成定点数类型, 另一种比较方便的实现可以用`exp`函数(如下页代码所示)

4. 例子(Examples)

   ```Scala
   //无符号定点数
   val UQ_8_2 = UFix(peak = 8 exp, resolution = -2 exp)    //bit宽度是8-(-2)=10 bits
   val UQ_8_2 = UFix(8 exp, -2 exp)

   val UQ_8_2 = UFix(peak = 8 exp, width = 10 bits)
   val UQ_8_2 = UFix(8 exp, 10 bits)

   //有符号定点数
   val Q_8_2 = SFix(peak = 8 exp, width = 10 bits)         //bit宽度是8-(-2)+1=11 bits
   val Q_8_2 = SFix(8 exp, -2exp)

   val Q_8_2 = SFix(peak = 8 exp, width = 11 bits)
   val Q_8_2 = SFix(8 exp, 11 bits)
   ```

### 三、赋值(Assignments)

1. 有效赋值(Valid Assignments)

   没有bit丢失时, 对定点数的赋值是有效的, 任何一位的bit丢失都会产生错误。
   
   如果定点数的来源数太大, `.truncated`函数可以帮你改变元数据的尺寸来匹配目标尺寸。

   **举例**

   ```Scala
   val i16_m2  = SFix(16 exp, -2 exp)
   val i16_0   = SFix(16 exp,  0 exp)
   val i8_m2   = SFix( 8 exp, -2 exp)
   val o16_m2  = SFix(16 exp, -2 exp)
   val o16_m0  = SFix(16 exp,  0 exp)
   val o14_m2  = SFix(14 exp, -2 exp)

   o16_m2  := i16_m2           //OK
   o16_m0  := i16_m2           //Not OK, bit丢失
   o14_m2  := i16_m2           //Not OK, bit丢失
   o16_m0  := i16_m2.truncated //OK, 尺寸自动改变
   o14_m2  := i16_m2.truncated //OK, 尺寸自动改变
   ```

   Verilog:

   ```Verilog
   wire       [18:0]   i16_m2;
   wire       [16:0]   i16_0;
   wire       [10:0]   i8_m2;
   wire       [18:0]   o16_m2;
   wire       [16:0]   o16_m0;
   wire       [16:0]   o14_m2;
   reg        [7:0]    counter;

   assign o16_m2 = i16_m2;
   assign o16_m0 = (i16_m2 >>> 2);
   assign o14_m2 = i16_m2[16:0];

   ```

2. 赋值来自于Scala常量(From a Scala constant)

   Scala`BigInt`或者`Double`类型能在给`UFix`或者`SFix`信号赋值的时候视为常量。

   **举例**

   ```Scala
   val i4_m2 = SFix(4 exp, -2 exp)
   i4_m2 := 1.25   //会在i4_m2.raw中载入5
   i4_m2 := 4      //会在i4_m2.raw中载入16
   ```

   Verilog:

   ```Verilog
   wire       [6:0]    i4_m2;

   assign i4_m2 = 7'h05;
   assign i4_m2 = 7'h10;
   ```

### 四、Raw值(Raw value)

定点数的整体表达可以通过`raw`属性读写数据。

**举例**

```Scala
val UQ_8_2 = UFix(8 exp, 10 bits)
UQ_8_2.raw := 4     //00000001 00赋值为1
UQ_8_2.raw := U(17) //00000100 01赋值为4.25
```

Verilog:

```Verilog
wire       [9:0]    UQ_8_2;

assign UQ_8_2 = 10'h004;
assign UQ_8_2 = 10'h011;
```

### 五、操作符(Operators)

以下所`UFix`类型所支持的操作符：

1. 算数运算(Arithmetic)

   |   操作符   |      描述       |      返回的小数部分分辨率       |      返回的整数部分范围       |
   | :--------: | :-------------: | :-----------------------------: | :---------------------------: |
   |    x+y     |      加法       | Min(x.resolution, y.resolution) | Max(x.amplitude, y.amplitude) |
   |    x-y     |      减法       | Min(x.resolution, y.resolution) | Max(x.amplitude, y.amplitude) |
   |    x*y     |      乘法       |    x.resolution*y.resolution    |    x.amplitude*y.amplitude    |
   |    x>>y    | 算数右移, y:Int |         x.amplitude>>y          |        x.resolution>>y        |
   |    x<<y    | 算数左移, y:Int |         x.amplitude<<y          |        x.resolution<<y        |
   | x>>&#124;y | 算数右移, y:Int |         x.amplitude>>y          |         x.resolution          |
   | x<<&#124;y | 算数右移, y:Int |         x.amplitude<<y          |         x.resolution          |

2. 比较运算(Comparison)

   | 操作符 |   描述   | 返回类型 |
   | :----: | :------: | :------: |
   | x===y  |   相等   |   Bool   |
   | x=/=y  |  不相等  |   Bool   |
   |  x>y   |   大于   |   Bool   |
   |  x>=y  | 大于等于 |   Bool   |
   |  x<y   |   小于   |   Bool   |
   |  x<=y  | 小于等于 |   Bool   |

3. 类型转换(Type cast)

   |  操作符   |              描述              |      返回类型       |
   | :-------: | :----------------------------: | :-----------------: |
   | x.asBits  |        二进制转换为Bits        |   Bits(w(x) bits)   |
   | x.asUInt  |        二进制转换为UInt        |   UInt(w(x) bits)   |
   | x.asSInt  |        二进制转换为SInt        |   SInt(w(x) bits)   |
   | x.asBools |         转换为Bool数组         | Vec(Bool, width(x)) |
   | x.toUInt  | 返回对应的UInt(自带truncation) |        UInt         |
   | x.toSInt  | 返回对应的SInt(自带truncation) |        SInt         |
   | x.toUFix  |         返回对应的UFix         |        UFix         |
   | x.toSFix  |         返回对应的SFix         |        SFix         |

4. Misc##有问题, x.resolution那里##

   |    操作符    |          描述           | 返回类型 |
   | :----------: | :---------------------: | :------: |
   |  x.maxValue  |   返回可存储的最大值    |  Double  |
   |  x.minValue  |   返回可存储的最小值    |  Double  |
   | x.resolution | x.amplitude*y.amplitude |  Double  |
