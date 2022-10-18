## UInt/SInt

### 一、描述(Description)

`UInt`/`SInt`数据类型是一个能用在有/无符号计算中的bits向量

### 二、声明(Declaration)

声明整型的语法如下所示：([]中的是可填项)

|                                                     语法                                                      |                       描述                       |   返回类型   |
| :-----------------------------------------------------------------------------------------------------------: | :----------------------------------------------: | :----------: |
|                                             UInt[()]<br>SInt[()]                                              |    创建有/无符号整型, bits个数编译器推断得到     | UInt<br>SInt |
|                                         UInt(x bits)<br>SInt(x bits)                                          |            创建x bits的有/无符号整型             | UInt<br>SInt |
| U(value: Int[, xbits])<br>U(value: BigInt[, x bits])<br>S(value: Int[, x bits])<br>S(value: BigInt[, x bits]) |         创建有/无符号整型, 并赋值'value'         | UInt<br>SInt |
|                                U"[[size']base]value"<br>S"[[size']base]value"                                 | 创建有/无符号整型, 并赋值'value'(基：h, d, o, b) | UInt<br>SInt |
|                           U([x bits, ] element, ...)<br>S([x bits, ] element, ...)                            |       创建有/无符号整型, 赋值由element决定       | UInt<br>SInt |

```Scala
val myUInt = UInt(8 bits)
myUInt := U(2, 8 bits)
myUInt := U(2)
myUInt := U"0000_0101"  //基默认二进制
myUInt := U"h1A"        //x/h->基16, d->基10, o->基8, b->基2
myUInt := U"8'h1A"
myUInt := 2             //可以使用Scala Int作为字面值

val myBool := myUInt === U(7 -> true, (downto 0) -> false)
val myBool := myUInt === U(myUInt.range -> true)

//在赋值的时候, 可以去掉U/S, 也支持[default -> ???]
myUInt := (default -> true)                 //赋值“11111111”
myUInt := (myUInt.range -> true)            //赋值“11111111

myUInt := (7 -> true, default -> false)     //赋值“10000000”
myUInt := ((4 downto 1) -> true, default -> false)  //赋值“00011110”
```

```Verilog
  wire       [7:0]    myUInt;
  reg        [7:0]    _zz_myBool;
  wire                myBool;

  function [7:0] zz__zz_myBool(input dummy);
    begin
      zz__zz_myBool[7] = 1'b1;
      zz__zz_myBool[6 : 0] = 7'h0;
    end
  endfunction
  wire [7:0] _zz_1;

  assign myUInt = 8'h02;
  assign myUInt = 8'h02;
  assign myUInt = 8'h05;
  assign myUInt = 8'h1A;
  assign myUInt = 8'h1A;
  assign myUInt = 8'h02;
  
  assign _zz_1 = zz__zz_myBool(1'b0);
  always @(*) _zz_myBool = _zz_1;
  assign myBool = (myUInt == _zz_myBool);

  assign _zz_myBool[7 : 0] = 8'hff;
  assign myBool = (myUInt == _zz_myBool);
  assign myUInt = 8'hff;
  assign myUInt = 8'hff;
  assign myUInt = 8'h80;
  assign myUInt = 8'h1E;
```

### 三、操作符(Operators)

下面是`UInt`和`SInt`支持的操作符：

1. 逻辑运算(Logic)##这里有问题##

   |           操作符           |           描述            |        返回类型         |
   | :------------------------: | :-----------------------: | :---------------------: |
   |           x ^ y            |         逻辑异或          |          Bool           |
   |             ~x             |         按位取非          |      T(w(x) bits)       |
   |           x & y            |          按位与           | T(max(w(x), w(y)) bits) |
   |         x &#124; y         |          按位或           | T(max(w(x), w(y)) bits) |
   |           x ^ y            |         按位异或          | T(max(w(x), w(y)) bits) |
   |           x.xorR           |     x的所有bits取异或     |          Bool           |
   |           x.orR            |      x的所有bits取或      |          Bool           |
   |           x.andR           |      x的所有bits取与      |          Bool           |
   |           x >> y           |     算术右移, y: Int      |     T(w(x)-y bits)      |
   |           x >> y           |     算术右移, y: UInt     |      T(w(x) bits)       |
   |           x << y           |     算术左移, y: Int      |     T(w(x)+y bits)      |
   |           x << y           |     算术左移, y: UInt     |   T(w(x)+max(y) bits)   |
   |             x              |           >> y            |  逻辑右移, y: Int/UInt  | T(w(x) bits) |
   |             x              |           << y            |  逻辑左移, y: Int/UInt  | T(w(x) bits) |
   |      x.rotateLeft(y)       | 逻辑循环左移, y: UInt/Int |      T(w(x) bits)       |
   |      x.rotateRight(y)      | 逻辑循环右移, y: UInt/Int |      T(w(x) bits)       |
   |       x.clearAll[()]       |       清除所有bits        |                         |
   |        x.setAll[()]        |       置位所有bits        |                         |
   | x.setAllTo(value: Boolean) |   根据给定Boolean值置位   |                         |
   |  x.setAllTo(value: Bool)   |    根据给定Bool值置位     |                         |

   > 备注：`x rotateLeft y`和`x rotateRight y`也是有效语句

   > 备注：注意`x >> 2`和`x >> U(2)`的区别, 前者返回类型是`T(w(x)-2)`, 后者返回类型是`T(w(x))`, 其次更重要的是, 前者的2视为Int后者的U(2)视为硬件信号。

   ```Scala
   val a, b, c = SInt(32 bits)
   a := S(5)
   b := S(10)

   //比特级操作
   c := ~(a & b)
   assert(c.getWidth == 32)

   //移位
   val arithShift = UInt(8 bits) << 2  //结果10 bits
   val logicShift = UInt(8 bits) |<< 2 //结果8 bits
   assert(arithShift.getWidth == 10)
   assert(logicShift.getWidth == 8)

   //循环移位
   val rotated = UInt(8 bits) rotateLeft 3
   assert(rotated.getWidth == 8)

   //当a全1, b也全1
   when(a.andR) { b.setAll() }
   ```

   Verilog:

   ```Verilog
   wire       [31:0]   _zz_c;
   wire       [31:0]   a;
   reg        [31:0]   b;
   wire       [31:0]   c;
   wire       [7:0]    _zz_arithShift;
   wire       [9:0]    arithShift;
   wire       [7:0]    _zz_logicShift;
   wire       [7:0]    logicShift;
   wire       [7:0]    _zz_rotated;
   wire       [7:0]    rotated;
   wire                when_MyTopLevel_l51;

   assign _zz_c = (a & b);
   assign a = 32'h00000005;
   always @(*) begin
       b = 32'h0000000a;
       if(when_MyTopLevel_l51) begin
           b = 32'hffffffff;
       end
   end

   assign c = (~ _zz_c);
   assign arithShift = ({2'd0,_zz_arithShift} <<< 2);
   assign logicShift = (_zz_logicShift <<< 2);
   assign rotated = {_zz_rotated[4 : 0],_zz_rotated[7 : 5]};
   assign when_MyTopLevel_l51 = (&a);

   ```

2. 运算(Arithmetic)

   |  操作符   |    描述    |         返回类型          |
   | :-------: | :--------: | :-----------------------: |
   |    x+y    |    加法    |  T(max(w(x), w(y)) bits)  |
   |   x+^y    | 产生进位加 | T(max(w(x), w(y))+1 bits) |
   | x+&#124;y | 溢出判断加 |  T(max(w(x), w(y)) bits)  |
   |    x-y    |    减法    |  T(max(w(x), w(y)) bits)  |
   |   x-^y    | 产生借位减 | T(max(w(x), w(y))+1 bits) |
   |    x-     |     y      |           减法            | T(max(w(x), w(y)) bits) |
   |    x*y    |    乘法    |  T(max(w(x), w(y)) bits)  |
   |    x/y    |    除法    |        T(w(x)bits)        |
   |    x%y    |    取模    |        T(w(x)bits)        |

   ```Scala
   val a, b, c = UInt(8 bits)
   a := U"xf0"
   b := U"x0f"

   c := a + b
   assert(c === U"8'xff")

   val d = a +^ b
   assert(d === U"9'x0ff")

   val e = a +| U"8'x20"
   assert(e === U"8'xff")
   ```

   Verilog:

   ```Verilog
   wire       [7:0]    a;
   wire       [7:0]    b;
   wire       [7:0]    c;
   wire       [8:0]    d;
   wire       [8:0]    _zz_e;
   reg        [7:0]    e;
   wire                when_UInt_l119;
   reg        [7:0]    counter;

   assign a = 8'hf0;
   assign b = 8'h0f;
   assign c = (a + b);
   assign d = ({1'b0,a} + {1'b0,b});
   assign _zz_e = ({1'b0,a} + {1'b0,8'h20});
   assign when_UInt_l119 = (|_zz_e[8 : 8]);
   always @(*) begin
       if(when_UInt_l119) begin
           e = 8'hff;
       end else begin
           e = _zz_e[7 : 0];
       end
   end

   ```

   > 备注：需要注意的是, 该例程的仿真assert用的是`===`, 而相对的上一例程中细化assert用的是`==`

3. 对比操作(Comparison)

   | 操作符 |   描述   | 返回类型 |
   | :----: | :------: | :------: |
   | x===y  |   相等   |   Bool   |
   |  x=/=  |  不相等  |   Bool   |
   |  x>y   |   大于   |   Bool   |
   |  x>=y  | 大于等于 |   Bool   |
   |  x<y   |   小于   |   Bool   |
   |  x<=y  | 小于等于 |   Bool   |

   ```Scala
   val a = U(5, 8 bits)
   val b = U(10, 8 bits)
   val c = UInt(2 bits)

   when (a > b) {
       c := U"10"
   } elsewhen (a =/= b) {
       c := U"01"
   } elsewhen (a === U(0)) {
       c.setAll()
   } otherwise {
       c.clearAll()
   }
   ```

   Verilog:

   ```Verilog
   wire       [7:0]    a;
   wire       [7:0]    b;
   reg        [1:0]    c;
   wire                when_MyTopLevel_l39;
   wire                when_MyTopLevel_l41;
   wire                when_MyTopLevel_l43;
   reg        [7:0]    counter;

   assign a = 8'h05;
   assign b = 8'h0a;
   assign when_MyTopLevel_l39 = (b < a);
   always @(*) begin
       if(when_MyTopLevel_l39) begin
           c = 2'b10;
       end else begin
           if(when_MyTopLevel_l41) begin
               c = 2'b01;
           end else begin
               if(when_MyTopLevel_l43) begin
                   c = 2'b11;
               end else begin
                   c = 2'b00;
               end
           end
       end
   end

   ssign when_MyTopLevel_l41 = (a != b);
   assign when_MyTopLevel_l43 = (a == 8'h0);
   ```

4. 类型转换(Type cast)

   |   操作符   |         描述         |     返回类型      |
   | :--------: | :------------------: | :---------------: |
   |  x.asBits  |   二进制转换成Bits   |  Bits(w(x) bits)  |
   |  x.asUInt  |   二进制转换成UInt   |  UInt(w(x) bits)  |
   |  x.asSInt  |   二进制转换成SInt   |  SInt(w(x) bits)  |
   | x.asBools  |    转换成Bool数组    |  Vec(Bool, w(x))  |
   |   S(x:T)   |   把数据转换成SInt   |  SInt(w(x) bits)  |
   |   U(x:T)   |   把数据转换成UInt   |  UInt(w(x) bits)  |
   | x.intoSInt | 扩展符号位转换成SInt | SInt(w(x)+1 bits) |

   将`Bool`, `Bits`, `SInt`转化成`UInt`可以用`U(something)`, 将类型转化为SInt的时候可以用`S(something)`。

   ```Scala
   //把SInt转换成Bits
   val myBits = mySInt.asBits

   //创建一个Bool量
   val myVec = myUint.asBools

   //把Bits转换成SInt
   val mySInt = S(myBits)
   ```

   Verilog:

   ```Verilog
   wire       [7:0]    mySInt;
   wire       [7:0]    myUInt;
   wire       [7:0]    myBits;
   wire                myVec_0;
   wire                myVec_1;
   wire                myVec_2;
   wire                myVec_3;
   wire                myVec_4;
   wire                myVec_5;
   wire                myVec_6;
   wire                myVec_7;

   assign myBits = mySInt;
   assign myVec_0 = myUInt[0];
   assign myVec_1 = myUInt[1];
   assign myVec_2 = myUInt[2];
   assign myVec_3 = myUInt[3];
   assign myVec_4 = myUInt[4];
   assign myVec_5 = myUInt[5];
   assign myVec_6 = myUInt[6];
   assign myVec_7 = myUInt[7];

   assign mySInt = myBits;
   ```

 1. 提取比特位(Bit extraction)

   |       操作符        |                       描述                       |   返回类型    |
   | :-----------------: | :----------------------------------------------: | :-----------: |
   |        x(y)         |                  读第y bits的值                  |     Bool      |
   |  x(offset, width)   |       读bit区域, offset: UInt, width: Int        | T(width bits) |
   |      x(range)       |      读某范围内bits, 例如myBits(4 downto 2)      | T(range bits) |
   |       x(y):=z       |              赋值某bit, y:Int/UInt               |     Bool      |
   | x(offset, width):=z |       赋值bit区域, offset:UInt, width:Int        | T(width bits) |
   |     x(range):=z     | 赋值某范围内bits, 例如myBits(4 downto 2):=U"010" | T(range bits) |

   ```Scala
   //取第4bit数据
   val myBool = myUInt(4)

   //把第1bit赋值为True
   mySInt(1) := True

   //范围内赋值
   val myUInt_8bits = myUInt_16bits(7 downto 0)
   val myUInt_7bits = myUInt_16bits(0 to 6)
   val myUInt_6bits = myUInt_16bits(0 until 6)

   mySInt_8bits(3 downto 0) := mySInt_4bits
   ```

   Verilog:

   ```Verilog
   wire       [7:0]    myUInt;
   reg        [7:0]    mySInt;
   wire       [15:0]   myUInt_16bits;
   reg        [7:0]    mySInt_8bits;
   wire       [3:0]    mySInt_4bits;
   wire                myBool;
   wire       [7:0]    myUInt_8bits;
   wire       [6:0]    myUInt_7bits;
   wire       [5:0]    myUInt_6bits;

   assign myUInt_16bits = {8'd0, myUInt};
   assign myBool = myUInt[4];
   assign myUInt_8bits = myUInt_16bits[7 : 0];
   assign myUInt_7bits = myUInt_16bits[6 : 0];
   assign myUInt_6bits = myUInt_16bits[5 : 0];
   always @(*) begin
       mySInt_8bits[3 : 0] = mySInt_4bits;
       mySInt_8bits[7 : 4] = 4'b0000;
   end

   always @(posedge clk or posedge reset) begin
       if(reset) begin
           mySInt <= 8'h0;
       end else begin
           mySInt[1] <= 1'b1;
       end
   end
   ```

5. Misc

   |            操作符             |                                  描述                                  |        返回类型        |
   | :---------------------------: | :--------------------------------------------------------------------: | :--------------------: |
   |          x.getWidth           |                              返回bit位数                               |          Int           |
   |             x.msb             |                             返回最高有效位                             |          Bool          |
   |             x.lsb             |                             返回最低有效位                             |          Bool          |
   |            x.range            |                        返回区间范围(x.high到0)                         |         Range          |
   |            x.high             |                              返回x的上限                               |          Int           |
   |             x##y              |                       数据拼接, x->高位, y->低位                       |  Bits(w(x)+w(y) bits)  |
   |             x@@y              |                    数据拼接, x:T, y:Bool/SInt/UInt                     |   T(w(x)+w(y) bits)    |
   |    x.subdivideln(y slices)    |                          把x切片成y份, y:Int                           |        Vec(T,y)        |
   |     x.subdivideln(y bits)     |                         把x按y bits切片, y:Int                         |     Vec(T,w(x)/y)      |
   |          x.resize(y)          | 返回x的长度变换后的复制, UInt长度增大补零, SInt长度增大补符号位, y:Int |       T(y bits)        |
   |           x.resized           |                        返回按需自动确定长度的x                         |      T(w(x) bits)      |
   | myUInt.twoComplement(en:Bool) |                         用补码把UInt转换成SInt                         |   SInt(w(myUInt)+1)    |
   |          mySInt.abs           |                          返回UInt类型的绝对值                          |  UInt(w(mySInt),bits)  |
   |      mySInt.abs(en:Bool)      |                    如果en是True返回UInt类型的绝对值                    |  UInt(w(mySInt),bits)  |
   |          mySInt.sign          |                             返回最高有效位                             |          Bool          |
   |           x.expand            |                           返回1bit拓展后的x                            |     T(w(x)+1 bits)     |
   |       mySInt.absWithSym       |                    返回对称收缩1 bit的UInt的绝对值                     | UInt(w(mySInt)-1 bits) |

   ```Scala
   myBool := mySInt.lsb    //等价于mySInt(0)

   //拼接
   val mySInt = mySInt_1 @@ mySInt_1 @@ myBool
   val myBits = mySInt_1 ## mySInt_1 ## myBool

   //分块
   val sel = UInt(2 bits)
   val mySIntWord = mySInt_128bits.subdivideIn(32 bits)(sel)
   //sel = 0 => mySIntWord = mySInt_128bits(128 downto 96)
   //sel = 1 => mySIntWord = mySInt_128bits(95 downto 64)
   //sel = 2 => mySIntWord = mySInt_128bits(63 downto 32)
   //sel = 3 => mySIntWord = mySInt_128bits(31 downto 0)

   //如果想要反向访问数据：
   val myVector = mySInt_128bits.subdivideIn(32 bits).reverse
   val mySIntWord = myVector(sel)

   //变换大小
   myUInt_32bits := U"32'x11223344"
   myUInt_8bits := myUInt_32bits.resized   //长度自动推断(myUInt_8bits=0x44)
   myUInt_8bits := myUInt_32bits.resize(8) //变换为8 bits(myUInt_8bits=0x44)

   //取补码
   mySInt := myUInt.twoComplement(myBool)

   //取绝对值
   mySInt_abs := mySInt.abs
   ```

   Verilog:

   ```Verilog
   wire       [15:0]   _zz_mySInt;
   reg        [31:0]   _zz_mySIntWord;
   wire       [31:0]   _zz_mySIntWord_1;
   wire       [31:0]   _zz_mySIntWord_2;
   wire       [31:0]   _zz_mySIntWord_3;
   wire       [31:0]   _zz_mySIntWord_4;
   wire                myBool;
   wire       [7:0]    mySInt_1;
   wire       [127:0]  mySInt_128bits;
   wire       [16:0]   mySInt;
   wire       [16:0]   myBits;
   wire       [1:0]    sel;
   wire       [31:0]   mySIntWord;
   wire       [31:0]   myUInt_32bits;
   wire       [7:0]    myUInt_8bits;
   wire       [7:0]    myUInt;
   wire       [16:0]   mySInt_abs;

   assign _zz_mySInt = {mySInt_1,mySInt_1};
   assign _zz_mySIntWord_1 = mySInt_128bits[31 : 0];
   assign _zz_mySIntWord_2 = mySInt_128bits[63 : 32];
   assign _zz_mySIntWord_3 = mySInt_128bits[95 : 64];
   assign _zz_mySIntWord_4 = mySInt_128bits[127 : 96];
   always @(*) begin
       case(sel)
           2'b00 : _zz_mySIntWord = _zz_mySIntWord_1;
           2'b01 : _zz_mySIntWord = _zz_mySIntWord_2;
           2'b10 : _zz_mySIntWord = _zz_mySIntWord_3;
           default : _zz_mySIntWord = _zz_mySIntWord_4;
       endcase
   end

   assign myBool = mySInt[0];
   assign mySInt = {_zz_mySInt,myBool};
   assign myBits = {{mySInt_1,mySInt_1},myBool};
   assign mySIntWord = _zz_mySIntWord;
   assign myUInt_32bits = 32'h11223344;
   assign myUInt_8bits = myUInt_32bits[7:0];
   assign myUInt = 8'h28;
   assign mySInt_abs = 17'h0;

   assign _zz_mySInt = ({myBool,(myBool ? (~ myUInt) : myUInt)} + _zz_mySInt_1);
   assign _zz_mySInt_2 = myBool;
   assign _zz_mySInt_1 = {8'd0, _zz_mySInt_2};

   assign _zz_mySInt_abs = (mySInt[8] ? _zz_mySInt_abs_1 : mySInt);
   assign _zz_mySInt_abs_1 = (~ mySInt);
   assign _zz_mySInt_abs_3 = mySInt[8];
   assign _zz_mySInt_abs_2 = {8'd0, _zz_mySInt_abs_3};

   assign mySInt_abs = (_zz_mySInt_abs + _zz_mySInt_abs_2);
   ```

### 四、定点数操作(FixPoint operations)

对于定点数, 我们可以把它分成两个部分：

+ 低位操作(四舍五入)
+ 高位操作(饱和运算)

1. 低位操作(Lower bit operations)

   ![lowerBitOperation](../../img/lowerBitOperation.png)

   | SpinalHDL舍入类型 | Wikipedia舍入类型 |     API     |        数学算法        | 返回(align=false) | 支持程度 |
   | :---------------: | :---------------: | :---------: | :--------------------: | :---------------: | :------: |
   |       FLOOR       |     RoundDown     |    floor    |        floor(x)        |    w(x)-n bits    |   Yes    |
   |    FLOORTOZERO    |    RoundToZero    | floorToZero |   sign*floor(abs(x))   |    w(x)-n bits    |   Yes    |
   |       CEIL        |      RoundUp      |    ceil     |        ceil(x)         |   w(x)-n+1 bits   |   Yes    |
   |     CEILTOINF     |    RoundToInf     |  ceilToInf  |   sign*ceil(abs(x))    |   w(x)-n+1 bits   |   Yes    |
   |      ROUNDUP      |    RoundHalfUp    |   roundUp   |      floor(x+0.5)      |   w(x)-n+1 bits   |   Yes    |
   |     ROUNDDOWN     |   RoundHalfDown   |  roundDown  |      ceil(x-0.5)       |   w(x)-n+1 bits   |   Yes    |
   |    ROUNDTOZERO    |  RoundHalfToZero  | roundToZero | sign*ceil(abs(x)-0.5)  |   w(x)-n+1 bits   |   Yes    |
   |    ROUNDTOINF     |  RoundHalfToInf   | roundToInf  | sign*floor(abs(x)+0.5) |   w(x)-n+1 bits   |   Yes    |
   |    ROUNDTOEVEN    |  RoundHalfToEven  | roundToEven |                        |                   |          |
   |    ROUNDTOODD     |  RoundHalfToOdd   | roundToOdd  |                        |                   |          |

   > 备注：RoundToEven和RoundToOdd模式都非常特殊, 常用在高精度大数据统计领域, SpinalHDL还没有支持它们。

   你会发现ROUNDUP, ROUNDDOWN, ROUNDTOZERO, ROUNDTOINF, ROUNDTOEVEN, ROUNDTOODD在行为上非常相似, ROUNDTOINF所最常见的, 不同编程语言中的四舍五入方法可能不同。

   |  编程语言  | 默认四舍五入类型 |                             举例                             |        评估         |
   | :--------: | :--------------: | :----------------------------------------------------------: | :-----------------: |  |
   |   Matlab   |    ROUNDTOINF    | round(1.5)=2, round(2.5)=3<br>round(-1.5)=-2, round(-2.5)=-3 | round to +-Infinity |
   |  python2   |    ROUNDTOINF    | round(1.5)=2, round(2.5)=3<br>round(-1.5)=-2, round(-2.5)=-3 | round to +-Infinity |
   |  python3   |   ROUNDTOEVEN    |    round(1.5)=round(2.5)=2<br>round(-1.5)=round(-2.5)=-2     |    close to Even    |
   | Scala.math |    ROUNDTOUP     | round(1.5)=2, round(2.5)=3<br>round(-1.5)=-1, round(-2.5)=-2 | always to +Infinity |
   | SpinalHDL  |    ROUNDTOINF    | round(1.5)=2, round(2.5)=3<br>round(-1.5)=-2, round(-2.5)=-3 | round to +-Infinity |

   > 备注：在SpinalHDL中ROUNDTOINF默认是RoundType(`round = roundToInf`)

   ```Scala
   val A = SInt(16 bits)
   val B = A.roundToInf(6)    //默认'align=false', 会得到11 bits
   val B = A.roundToInf(6, align = true)  //舍去进位, 得到10 bits
   val B = A.floor(6 bits)         //返回10 bits
   val B = A.floorToZero(6 bits)   //返回10 bits
   val B = A.ceil(6 bits)          //带进位故返回11 bits
   val B = A.ceil(6 bits, align = true)    //舍去进位, 得到10 bits
   val B = A.ceilToInf(6 bits)
   val B = A.roundUp(6 bits)
   val B = A.roundDown(6 bits)
   val B = A.roundToInf(6 bits)
   val B = A.roundToZero(6 bits)
   val B = A.round(6 bits)         //SpinalHDL用roundToInf作为默认rounding模式

   val B0 = A.roundToInf(6 bits, align = true) //---+
                                               //--> equal
   val B1 = A.roundToInf(6 bits, align = false).sat(1) //---+
   ```

   Verilog:

   ```Verilog
   //这里只举一个例子，其他的可自行生成
   //val B = A.roundToInf(6, align = true)     当前版本默认align = true，与文档有所不同    有符号数两边近似
   wire       [16:0]   _zz__zz_when_SInt_l337_2;
   wire       [16:0]   _zz__zz_when_SInt_l337_2_1;
   wire       [5:0]    _zz_when_SInt_l191;
   wire       [10:0]   _zz__zz_B_3;
   wire       [10:0]   _zz__zz_B_3_1;
   wire       [16:0]   _zz__zz_B;
   wire       [16:0]   _zz__zz_B_1;
   wire       [16:0]   _zz__zz_B_2;
   wire       [1:0]    _zz_when_SInt_l131;
   wire       [0:0]    _zz_when_SInt_l137;
   wire       [15:0]   A;
   reg        [10:0]   _zz_B;
   wire       [15:0]   _zz_B_1;
   wire       [15:0]   _zz_when_SInt_l337;
   wire       [15:0]   _zz_when_SInt_l337_1;
   wire       [16:0]   _zz_when_SInt_l337_2;
   wire       [15:0]   _zz_B_2;
   wire                when_SInt_l337;
   reg        [10:0]   _zz_B_3;
   wire                when_SInt_l191;
   reg        [9:0]    B;
   wire                when_SInt_l130;
   wire                when_SInt_l131;
   wire                when_SInt_l137;
   reg        [7:0]    counter;

   assign _zz__zz_when_SInt_l337_2 = {_zz_when_SInt_l337_1[15],_zz_when_SInt_l337_1};  //A符号位拓展1bit
   assign _zz__zz_when_SInt_l337_2_1 = {_zz_when_SInt_l337[15],_zz_when_SInt_l337};    //前6bit为0，其余为1的符号位拓展, 即-2^5
   assign _zz_when_SInt_l191 = _zz_when_SInt_l337_2[5 : 0];                            //A-0.5的小数部分
   assign _zz__zz_B_3 = _zz_when_SInt_l337_2[16 : 6];                                  //A-0.5的整数部分
   assign _zz__zz_B_3_1 = 11'h001;                                                     //1
   assign _zz__zz_B = ($signed(_zz__zz_B_1) + $signed(_zz__zz_B_2));                   //符号位拓展后的A+2^5, 相当于A+0.5
   assign _zz__zz_B_1 = {_zz_B_2[15],_zz_B_2};                                         //A符号位拓展1bit
   assign _zz__zz_B_2 = {_zz_B_1[15],_zz_B_1};                                         //2^5符号位拓展1bit
   assign _zz_when_SInt_l131 = _zz_B[10 : 9];                                          //溢出判断位
   assign _zz_when_SInt_l137 = _zz_B[9 : 9];                                           //符号位
   assign _zz_B_1 = {{10'h0,1'b1},5'h0};                                               //16‘b0000_0000_0010_0000, 即2^5
   assign _zz_when_SInt_l337 = {11'h7ff,5'h0};                                         //前5bit为0，其余为1. 即-2^5
   assign _zz_when_SInt_l337_1 = A[15 : 0];
   assign _zz_when_SInt_l337_2 = ($signed(_zz__zz_when_SInt_l337_2) + $signed(_zz__zz_when_SInt_l337_2_1));    //符号位拓展后的A-2^5, 相当于A-0.5
   assign _zz_B_2 = A[15 : 0];                                                         //A
   assign when_SInt_l337 = _zz_when_SInt_l337_2[16];                                   //A-0.5符号位
   assign when_SInt_l191 = (|_zz_when_SInt_l191);                                      //A-0.5小数部分是否为0
   always @(*) begin
       if(when_SInt_l191) begin
           _zz_B_3 = ($signed(_zz__zz_B_3) + $signed(_zz__zz_B_3_1));                  //非0的话A-0.5的整数部分+1
       end else begin
           _zz_B_3 = _zz_when_SInt_l337_2[16 : 6];                                     //0的话结果为 A-0.5 整数部分
       end
   end

   always @(*) begin
       if(when_SInt_l337) begin                                                        //负数时取
           _zz_B = _zz_B_3;
       end else begin
           _zz_B = (_zz__zz_B >>> 6);                                                  //非负时A+0.5整数部分
       end
   end

   assign when_SInt_l130 = _zz_B[10];
   assign when_SInt_l131 = (! (&_zz_when_SInt_l131));
   always @(*) begin
       if(when_SInt_l130) begin
           if(when_SInt_l131) begin
               B = 10'h200;
           end else begin
               B = _zz_B[9 : 0];
           end
       end else begin
           if(when_SInt_l137) begin
               B = 10'h1ff;
           end else begin
               B = _zz_B[9 : 0];
           end
       end
   end

   assign when_SInt_l137 = (|_zz_when_SInt_l137);
   ```

   > 备注：只有`floor`和`floorToZero`没有`align`选项, 它们不需要进位bit(align=true)。其他近似算法都默认带进位信息。

   |     API     | UInt/SInt |            描述             | Return(align=false) | Return(align=true) |
   | :---------: | :-------: | :-------------------------: | :-----------------: | :----------------: |
   |    floor    |   Both    |                             |     w(x)-n bits     |    w(x)-n bits     |
   | floorToZero |   SInt    |      与无符号floor相等      |     w(x)-n bits     |    w(x)-n bits     |
   |    ceil     |   Both    |                             |    w(x)-n+1 bits    |    w(x)-n bits     |
   |  ceilToInf  |   SInt    |      与无符号ceil相等       |    w(x)-n+1 bits    |    w(x)-n bits     |
   |   roundUp   |   Both    |          仅为了HW           |    w(x)-n+1 bits    |    w(x)-n bits     |
   |  roundDown  |   Both    |                             |    w(x)-n+1 bits    |    w(x)-n bits     |
   | roundToInf  |   SInt    |           最常用            |    w(x)-n+1 bits    |    w(x)-n bits     |
   | roundToZero |   SInt    |    无符号时等于roundDown    |    w(x)-n+1 bits    |    w(x)-n bits     |
   |    round    |   Both    | 在SpinalHDL中默认roundToInf |    w(x)-n+1 bits    |    w(x)-n bits     |

   > 备注：尽管`roundToInf`很常用, `roundUp`有着最小的开销和最好的时序, 并且几乎没有性能损失。因此强烈推荐使用`roundUp`

2. 高位操作(High bit operations)

   ![highBitOperation](../../img/highBitOperation.png)

   |   函数   | 操作  |              正数操作               |               负数操作               |
   | :------: | :---: | :---------------------------------: | :----------------------------------: |
   |   sat    | 饱和  | when(Top[w-1, w-n].orR)set maxValue | when(Top[w-1, w-n].andR)set minValue |
   |   trim   | 舍弃  |                 N/A                 |                 N/A                  |
   | symmetry | 对称  |                 N/A                 |          minValue=-maxValue          |

   对称仅对`SInt`有效

   ```Scala
   val A = SInt(8 bits)
   val B = A.sat(3 bits)       //高3 bits饱和的5 bits返回值
   val B = A.sat(3)            //与上式同理
   val B = A.trim(3 bits)      //高3 bits舍弃的5bits返回值
   val C = A.symmetry          //像(-128~127 to -127~127)一样返回8 bits
   val C = A.sat(3).symmetry          //像(-16~15 to -15~15)一样返回5 bits
   ```

   Verilog:

   ```Verilog
   //饱和
   wire       [3:0]    _zz_when_SInt_l131;
   wire       [2:0]    _zz_when_SInt_l137;
   wire       [7:0]    A;
   reg        [4:0]    B;
   wire                when_SInt_l130;
   wire                when_SInt_l131;
   wire                when_SInt_l137;

   assign _zz_when_SInt_l131 = A[7 : 4];
   assign _zz_when_SInt_l137 = A[6 : 4];
   assign when_SInt_l130 = A[7];
   assign when_SInt_l131 = (! (&_zz_when_SInt_l131));
   always @(*) begin
       if(when_SInt_l130) begin
           if(when_SInt_l131) begin
               B = 5'h10;
           end else begin
               B = A[4 : 0];
           end
       end else begin
           if(when_SInt_l137) begin
               B = 5'h0f;
           end else begin
               B = A[4 : 0];
           end
       end
   end

   assign when_SInt_l137 = (|_zz_when_SInt_l137);

   //trim舍弃
   wire       [7:0]    A;
   wire       [4:0]    B;

   assign B = A[4 : 0];

   //对称
   wire       [7:0]    _zz_B;
   wire       [7:0]    _zz_B_1;
   wire       [7:0]    A;
   wire       [7:0]    B;

   assign _zz_B = 8'h80;
   assign _zz_B_1 = 8'h81;
   assign B = (($signed(A) == $signed(_zz_B)) ? _zz_B_1 : A);

   ```

3. fixTo函数(fixTo function)

   针对`UInt`/`SInt`提供了两种定点方法：

   ![fixPoint](../../img/fixPoint.png)

   强烈推荐在RTL工程中使用`fixTo`, 有了这个函数你就不需要处理进位alignment和bit宽度的计算问题, 如同上图Way1那样。

   Fix函数关于自动处理饱和：

   |                 函数                 |        描述         |     返回类型      |
   | :----------------------------------: | :-----------------: | :---------------: |
   | fixTo(section, roundType, symmetric) | Factory FixFunction | section.size bits |

   ```Scala
   val A = SInt(16 bits)
   val B = A.fixTo(10 downto 3) //默认的RoundType是ROUNDTOINF, sym=false
   val B = A.fixTo(8 downto 0, RoundType.ROUNDUP)
   val B = A.fixTo(9 downto 3, RoundType.CEIL, sym = false)
   val B = A.fixTo(16 downto 1, RoundType.ROUNDTOINF, sym = true)
   val B = A.fixTo(10 downto 3, RoundType.FLOOR)   //向下取整3 bits, 舍弃进位5 bits @ highest
   val B = A.fixTo(20 downto 3, RoundType.FLOOR)   //向下取整3 bits,拓展进位2 bits @ highest
   ```

   Verilog:

   ```Verilog
   wire       [7:0]    fixTo_dout;
   wire       [15:0]   A;

   SInt16fixTo10_3_ROUNDTOINF fixTo (
       .din  (A[15:0]        ), //i
       .dout (fixTo_dout[7:0])  //o
   );

   module SInt16fixTo10_3_ROUNDTOINF (
     input      [15:0]   din,
     output     [7:0]    dout
   );

     wire       [16:0]   _zz__zz_dout_4;
     wire       [16:0]   _zz__zz_dout_4_1;
     wire       [2:0]    _zz_when_SInt_l191;
     wire       [13:0]   _zz__zz_dout_6;
     wire       [13:0]   _zz__zz_dout_6_1;
     wire       [16:0]   _zz__zz_dout;
     wire       [16:0]   _zz__zz_dout_1;
     wire       [16:0]   _zz__zz_dout_2;
     wire       [6:0]    _zz_when_SInt_l131;
     wire       [5:0]    _zz_when_SInt_l137;
     reg        [13:0]   _zz_dout;
     wire       [15:0]   _zz_dout_1;
     wire       [15:0]   _zz_dout_2;
     wire       [15:0]   _zz_dout_3;
     wire       [16:0]   _zz_dout_4;
     wire       [15:0]   _zz_dout_5;
     wire                when_SInt_l337;
     reg        [13:0]   _zz_dout_6;
     wire                when_SInt_l191;
     reg        [7:0]    _zz_dout_7;
     wire                when_SInt_l130;
     wire                when_SInt_l131;
     wire                when_SInt_l137;

   assign _zz__zz_dout_4 = {_zz_dout_3[15],_zz_dout_3};
   assign _zz__zz_dout_4_1 = {_zz_dout_2[15],_zz_dout_2};
   assign _zz_when_SInt_l191 = _zz_dout_4[2 : 0];
   assign _zz__zz_dout_6 = _zz_dout_4[16 : 3];
   assign _zz__zz_dout_6_1 = 14'h0001;
   assign _zz__zz_dout = ($signed(_zz__zz_dout_1) + $signed(_zz__zz_dout_2));
   assign _zz__zz_dout_1 = {_zz_dout_5[15],_zz_dout_5};
   assign _zz__zz_dout_2 = {_zz_dout_1[15],_zz_dout_1};
   assign _zz_when_SInt_l131 = _zz_dout[13 : 7];
   assign _zz_when_SInt_l137 = _zz_dout[12 : 7];
   assign _zz_dout_1 = {{13'h0,1'b1},2'b00};
   assign _zz_dout_2 = {14'h3fff,2'b00};
   assign _zz_dout_3 = din[15 : 0];
   assign _zz_dout_4 = ($signed(_zz__zz_dout_4) + $signed(_zz__zz_dout_4_1));
   assign _zz_dout_5 = din[15 : 0];
   assign when_SInt_l337 = _zz_dout_4[16];
   assign when_SInt_l191 = (|_zz_when_SInt_l191);
   always @(*) begin
       if(when_SInt_l191) begin
           _zz_dout_6 = ($signed(_zz__zz_dout_6) + $signed(_zz__zz_dout_6_1));
       end else begin
           _zz_dout_6 = _zz_dout_4[16 : 3];
       end
   end

   always @(*) begin
       if(when_SInt_l337) begin
           _zz_dout = _zz_dout_6;
       end else begin
           _zz_dout = (_zz__zz_dout >>> 3);
       end
   end

   assign when_SInt_l130 = _zz_dout[13];
   assign when_SInt_l131 = (! (&_zz_when_SInt_l131));
   always @(*) begin
       if(when_SInt_l130) begin
           if(when_SInt_l131) begin
               _zz_dout_7 = 8'h80;
           end else begin
               _zz_dout_7 = _zz_dout[7 : 0];
           end
       end else begin
           if(when_SInt_l137) begin
               _zz_dout_7 = 8'h7f;
           end else begin
               _zz_dout_7 = _zz_dout[7 : 0];
           end
       end
   end

   assign when_SInt_l137 = (|_zz_when_SInt_l137);
   assign dout = _zz_dout_7;

   ```
