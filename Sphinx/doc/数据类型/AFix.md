## AFix(AFix是最新版Spinal新增的数据结构，不知为何该板块有些代码无法通过编译)

### 一、描述(Description)

`AFix`(Auto-ranging Fixed-Point), 是一类定点数能在进行定点数操作的过程中跟踪其表示范围的变化。

> 注意：这部分代码的绝大多数都仍在开发中, API和函数都可能随着版本变化, 欢迎大家提出反馈意见

### 二、声明(Declaration)

`AFix`能用bit大小或指数来创建：

```Scala
AFix.U(12 bits)         //U12.0
AFix.UQ(8 bits, 4 bits) //U8.4
AFix.U(8 exp, 12 bits)  //U8.4
AFix.U(8 exp, -4 exp)   //U8.4
AFix.U(8 exp, 4 exp)    //U8.-4

AFix.S(12 bits)         //S11 + sign
AFix.SQ(8 bits, 4 bits) //S8.4 + sign
AFix.S(8 exp, 12 bits)  //S8.3 + sign
AFix.S(8 exp, -4 exp)   //S8.4 + sign
```

例如：

AFix.U(12 bits)的范围是0到4095.

AFix.SQ(8 bits, 4 bits)的范围是-4096(-256)到4095(255.9375)

AFix.U(8 exp, 4 exp)的范围所0到256.

定制化的`AFix`范围能通过直接初始化类来创建。

```Scala
class AFix(val maxValue: BigInt, val minValue: BigInt, val exp: ExpNumber)

new AFix(4096, 0, 0 exp)        //[0 to 4096, 2^0]
new AFix(256, -256, -2 exp)     //[-256 to 256, 2^-2]
new AFix(16, 8, 2 exp)          //[8 to 16, 2^2]
```

`maxValue`和`minValue`存储着可表示的支持的整数值, 这些值代表乘以`2^exp`之后的真实定点值。

`AFix.U(2 exp, -1 exp)`能表示：`0.0, 0.5, 1.0, 1.5, 2.0, 2.5, 3.0, 3.5`

`AFix.U(2 exp, -2 exp)`能表示：`-2.0, -1.75, -1.5, -1.25, -1.0, -0.75, -0.5, -0.25, 0.0, 0.25, 0.5, 0.75, 1.0, 1.25, 1.5, 1.75`

指数可以大于0并且能代表大于1的值。

`AFix.S(2 exp, 1 exp)`能代表：`-4, 2, 0, 2`

`AFix(8, 16, 2 exp)`能代表：`32, 36, 40, 44, 48, 52, 56, 60, 65`

> 备注：`AFix`会用5 bits来存储这个类型, 正如它的`maxValue`存储`16`。

### 三、数学操作(Mathematical Operations)

`AFix`在硬件层面支持加法(`+`), 减法(`-`), 和乘法(`*`)。除法(`/`), 和取模(`%`), 也都提供了但在硬件描述中不建议使用。

`AFix`值的运算就好像一般的`Int`一样, 有符号和无符号数都是可执行的, 且有符号与无符号数之间没有数据类型的区别。

```Scala
//整数和分数的扩充
val a = AFix.U(4 bits)          // [   0 (  0.)     to  15 (15.  )]  4 bits, 2^0
val b = AFix.UQ(2 bits, 2 bits) // [   0 (  0.)     to  15 ( 3.75)]  4 bits, 2^-2
val c = a + b                   // [   0 (  0.)     to  77 (19.25)]  7 bits, 2^-2
val d = new AFix(-4, 8, -2 exp) // [-  4 (- 1.25)   to   8 ( 2.00)]  5 bits, 2^-2
val e = c * d                   // [-308 (-19.3125) to 616 (38.50)] 11 bits, 2^-4

//整数不扩充
val aa = new AFix(8, 16, -4) // [8 to 16] 5 bits, 2^-4
val bb = new AFix(1, 15, -4) // [1 to 15] 4 bits, 2^-4
val cc = aa + bb                 // [9 to 31] 5 bits, 2^-4
```

`AFix`支持运算且不用表示数扩展范围, 这种特点是通过选择每个输入的一致的最大最小范围来实现的。

`+|`无扩展加, `-|`无扩展减。

### 四、不相等操作(Inequality Operators)

`AFix`支持标准比较操作

```Scala
A === B
A =\= B
A < B
A <= B
A > B
A >= B
```

> 注意：在编译时超出区间范围的操作会被优化掉！

### 五、Bit移位(Bitshifting)

`AFix`支持小数点和bit移位。

`<<`把小数点向左移动, 移动的距离加到指数上。`>>`把小数点向右移动, 移动的距离从指数中减去。`<<|`把bits左移, 添加bits 0。`>>|`把bits右移, 移除bits

### 六、近似和饱和(Saturation and Rounding)

`AFix`实现了饱和和所有普遍的近似方法。

饱和可以饱和到`AFix`值的可支持的值, 有很多辅助函数考虑了指数。

```Scala
val a = new AFix(63, 0, -2 exp) // [0 to 63, 2^-2]
a.sat(63, 0)                    // [0 to 63, 2^-2]
a.sat(63, 0, -3 exp)            // [0 to 31, 2^-2]
a.sat(new AFix(31, 0, -1 exp))  // [0 to 31, 2^-2]
```

`AFix`近似模式：

```Scala
// The following require exp < 0
.floor() or .truncate()
.ceil()
.floorToZero()
.ceilToInf()
// The following require exp < -1
.roundHalfUp()
.roundHalfDown()
.roundHalfToZero()
.roundHalfToInf()
.roundHalfToEven()
.roundHalfToOdd()
```

这些近似模式的数学上的表述可以查看Wiki

所有这些模式都会产生一个指数部分为0的`AFix`。如果近似到不同的指数需要考虑移位或者用`truncated`赋值。

### 七、赋值(Assignment)

`AFix`会在赋值的时候自动地检查范围和精度。默认的, 把`AFix`赋值给另一个有更小的范围或精度的`AFix`会产生错误。

`.truncated`函数用来控制如何给更小的数值类型赋值。

```Scala
def truncated(
    saturation  : Boolean = false,
    overflow    : Boolean = true,
    rounding    : RoundType = RoundType.FLOOR)

def saturated(): AFix = this.truncated(saturation = true, overflow = false)
```

`RoundType`

```Scala
RoundType.FLOOR
RoundType.CEIL
RoundType.FLOORTOZERO
RoundType.CEILTOINF
RoundType.ROUNDUP
RoundType.ROUNDDOWN
RoundType.ROUNDTOZERO
RoundType.ROUNDTOINF
RoundType.ROUNDTOEVEN
RoundType.ROUNDTOODD
```

`saturation`标志在赋值数据类型的时候增加逻辑产生饱和。

`overflow`标志在近似后不检查范围区间直接赋值。

当用精度不同的数据去赋值的时候都需要近似处理。
