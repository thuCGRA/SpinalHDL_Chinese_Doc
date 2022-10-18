## Bits

### 一、描述(Description)

`Bits`类型对应于没有算数意义的bits向量

### 二、声明(Declaration)

声明bit向量的语法如下所示：([]中为可填项)

|                        语法                         |                    描述                     | 返回类型 |
| :-------------------------------------------------: | :-----------------------------------------: | :------: |
|                      Bits[()]                       | 创建一个BitVector, bits个数由编译器推断得到 |   Bits   |
|                    Bits(x bits)                     |          创建一个x bits的BitVector          |   Bits   |
| B(value: Int[,x bits])<br>B(value: BigInt[,x bits]) |      创建一个赋初值的x bits的BitVector      |   Bits   |
|                B"[[size']base]value"                |    创建一个基h/d/o/b的赋初值的BitVector     |   Bits   |
|              B([x bits,] element, ...)              |      创建一个由给定元素赋值的BitVector      |   Bits   |

```Scala
//Declaration
val myBits = Bits()
val myBits1 = Bits(32 bits)
val myBits2 = B(25, 8 bits)
val myBits3 = B"8'xFF"  //基可以是x/h(16), d(10), o(8), b(2)
val myBits4 = B"1001_0011"  //_增加可读性

//Element
val myBits5 = B(8 bits, default -> True) //"11111111"
val myBits6 = B(8 bits, (7 downto 5) -> B"101", 4 -> true, 3 -> True, default -> false) //"10111000"
val myBits7 = Bits(8 bits)
myBits7 := (7 -> true, default -> false)    //"10000000",在赋值的时候可以省去B
```

Verilog

```Verilog
  wire       [7:0]    myBits;
  wire       [31:0]   myBits1;
  wire       [7:0]    myBits2;
  wire       [7:0]    myBits3;
  wire       [7:0]    myBits4;
  wire       [7:0]    myBits5;
  reg        [7:0]    myBits6;
  reg        [7:0]    myBits7;

  assign myBits = 8'h19;
  assign myBits2 = 8'h19;
  assign myBits3 = 8'hff;
  assign myBits4 = 8'h93;
  assign myBits5 = 8'hff;

  function [7:0] zz_myBits6(input dummy);
    begin
      zz_myBits6 = 8'h0;
      zz_myBits6[7 : 5] = 3'b101;
      zz_myBits6[4] = 1'b1;
      zz_myBits6[3] = 1'b1;
    end
  endfunction

  assign _zz_1 = zz_myBits6(1'b0);
  always @(*) myBits6 = _zz_1;


  wire [7:0] _zz_1;
  function [7:0] zz_myBits7(input dummy);
    begin
      zz_myBits7 = 8'h0;
      zz_myBits7[7] = 1'b1;
    end
  endfunction
  wire [7:0] _zz_2;

  assign _zz_2 = zz_myBits7(1'b0);
  always @(*) myBits7 = _zz_2;

```

### 三、操作符(Operators)

1. 逻辑运算(Logic)
   以下是`Bits`类型的可用操作符

   |           操作符           |           描述            |        返回类型        |
   | :------------------------: | :-----------------------: | :--------------------: |
   |             ~x             |         按位取非          |    Bits(w(x) bits)     |
   |           x & y            |          按位与           |    Bits(w(xy) bits)    |
   |         x &#124; y         |          按位或           |    Bits(w(xy) bits)    |
   |           x ^ y            |         按位异或          |    Bits(w(xy) bits)    |
   |           x.xorR           |     x的所有bits取异或     |          Bool          |
   |           x.orR            |      x的所有bits取或      |          Bool          |
   |           x.andR           |      x的所有bits取与      |          Bool          |
   |           x >> y           |     逻辑右移, y: Int      |   Bits(w(x)-y bits)    |
   |           x >> y           |     逻辑右移, y: UInt     |    Bits(w(x) bits)     |
   |           x << y           |     逻辑左移, y: Int      |   Bits(w(x)+y bits)    |
   |           x << y           |     逻辑左移, y: UInt     | Bits(w(x)+max(y) bits) |
   |       x &#124;>> y         |   逻辑右移, y: Int/UInt   |    Bits(w(x) bits)     |
   |       x &#124;<< y         |   逻辑左移, y: Int/UInt   |    Bits(w(x) bits)     |
   |      x.rotateLeft(y)       | 逻辑循环左移, y: UInt/Int |    Bits(w(x) bits)     |
   |      x.rotateRight(y)      | 逻辑循环右移, y: UInt/Int |    Bits(w(x) bits)     |
   |       x.clearAll[()]       |       清除所有bits        |                        |
   |        x.setAll[()]        |       置位所有bits        |                        |
   | x.setAllTo(value: Boolean) |   根据给定Boolean值置位   |                        |
   |  x.setAllTo(value: Bool)   |    根据给定Bool值置位     |                        |

   > 以上移位虽说是逻辑移位, 但生成的Verilog中是算术移位, 用`>>>`或`<<<`

   ```Scala
   //Bit级操作符
   val a, b, c = Bits(32 bits)
   c := ~(a & b)

   val all_1 = a.andR  //检查a是不是全1

   //逻辑移位
   val bits_10bits = bits_8bits << 2   //结果是10bits
   val shift_8bits = bits_8bits |<< 2  //结果所8bits

   //循环移位
   val myBits = bits_8bits.rotateLeft(3)

   //置位/清零
   val a = B"8'x42"
   when(cond) {
       a.setAll()
   }
   ```

   Verilog:

   ```Verilog
   wire                cond;
   wire       [7:0]    bits_8bits;
   wire       [31:0]   a;
   wire       [31:0]   b;
   wire       [31:0]   c;
   wire                all_1;
   wire       [9:0]    bits_10bits;
   wire       [7:0]    shift_8bits;
   wire       [7:0]    myBits;
   reg        [7:0]    aa;

   assign bits_8bits = 8'h0a;
   assign c = (~ (a & b));
   assign all_1 = (&a);
   assign bits_10bits = ({2'd0,bits_8bits} <<< 2);
   assign shift_8bits = (bits_8bits <<< 2);
   assign myBits = {bits_8bits[4 : 0],bits_8bits[7 : 5]};
   always @(*) begin
     aa = 8'h42;
     if(cond) begin
       aa = 8'hff;
     end
   end
   ```

2. 比较运算(Comparison)

   | 操作符  |  描述  | 返回类型 |
   | :-----: | :----: | :------: |
   | x === y |  相等  |   Bool   |
   | x =/= y | 不相等 |   Bool   |

   ```Scala
   when(myBits === 3) {
       ...
   }

   when(myBits_32 =/= B"32'x44332211") {
       ...
   }
   ```

3. 类型转换(Type cast)

   |  操作符   |       描述       |    返回类型     |
   | :-------: | :--------------: | :-------------: |
   | x.asBits  | 二进制转换为Bits | Bits(w(x) bits) |
   | x.asUInt  | 二进制转换为UInt | UInt(w(x) bits) |
   | x.asSInt  | 二进制转换为SInt | SInt(w(x) bits) |
   | x.asBools | 转换成Bools数组  | Vec(Bool, w(x)) |
   |  B(x: T)  |  数据转换为Bits  | Bits(w(x) bits) |

   为了把`Bool`, `UInt`或`SInt`转换成`Bits`, 也可以用`B(something)`：

   ```Scala
   //把Bits转换成SInt
   val mySInt = myBits.asSInt

   //创建Bool向量
   val myVec = myBits.asBools

   //把SInt转换成Bits
   val myBits = B(mySInt)
   ```

   Verilog:

   ```Verilog
   reg        [7:0]    myBits;
   wire       [7:0]    mySInt;
   wire                myVec_0;
   wire                myVec_1;
   wire                myVec_2;
   wire                myVec_3;
   wire                myVec_4;
   wire                myVec_5;
   wire                myVec_6;
   wire                myVec_7;

   assign mySInt = myBits;

   assign myVec_0 = myBits[0];
   assign myVec_1 = myBits[1];
   assign myVec_2 = myBits[2];
   assign myVec_3 = myBits[3];
   assign myVec_4 = myBits[4];
   assign myVec_5 = myBits[5];
   assign myVec_6 = myBits[6];
   assign myVec_7 = myBits[7];
   ```

4. Bit位提取(Bit extraction)

   |           操作符           |                        描述                        |     返回类型     |
   | :------------------------: | :------------------------------------------------: | :--------------: |
   |            x(y)            |              读对应bit位, y: Int/UInt              |       Bool       |
   |   x(offset, width bits)    |        读一块bits, offset: UInt, width: Int        | Bit(width bits)  |
   |          x(range)          |       读范围内的bit, 例如myBits(4 down to 2)       | Bits(range bits) |
   |         x(y) := z          |             赋值对应bits, y: Int/UInt              | Bits(width bits) |
   | x(offset, width bits) := z |       赋值一块bits, offset: UInt, width: Int       | Bits(width bits) |
   |       x(range) := z        | 赋值范围内的bit, 例如myBits(4 down to 2) := B"010" | Bits(range bits) |

   ```Scala
   //取第4bit元素
   val myBool = myBits(4)

   //赋值
   myBits(1) := True

   //范围内操作
   val myBits_8bits = myBits_16bits(7 downto 0)
   val myBits_7bits = myBits_16bits(0 to 6)
   val myBits_6bits = myBits_16bits(0 until 6)

   myBits_8bits(3 downto 0) := myBits_4bits
   ```

   Verilog:

   ```Verilog
   reg        [7:0]    myBits;
   wire                myBool;
   reg        [15:0]   myBits_16bits;
   wire       [3:0]    myBits_4bits;
   wire       [7:0]    myBits_8bits;
   wire       [6:0]    myBits_7bits;
   wire       [5:0]    myBits_6bits;

   assign myBool = myBits[4];
   always @(*) begin
       myBits_16bits = {8'd0, myBits};
       myBits_16bits[3 : 0] = myBits_4bits;
   end

   assign myBits_4bits = myBits[3:0];
   assign myBits_8bits = myBits_16bits[7 : 0];
   assign myBits_7bits = myBits_16bits[6 : 0];
   assign myBits_6bits = myBits_16bits[5 : 0];

   always @(posedge clk or posedge reset) begin
       if(reset) begin
       myBits <= 8'h0;
       end else begin
       myBits[1] <= 1'b1;
       end
   end

   ```

5. Misc

   |         操作符          |                      描述                       |       返回类型       |
   | :---------------------: | :---------------------------------------------: | :------------------: |
   |       x.getWidth        |                   返回bit位数                   |         Int          |
   |         x.range         |       返回一个范围, 从大到小(x.high : 0)        |        Range         |
   |         x.high          |                 返回x的最高边界                 |         Int          |
   |          x.msb          |               返回最高有效位的值                |         Bool         |
   |          x.lsb          |               返回最低有效位的值                |         Bool         |
   |         x ## y          |           拼接操作, x->高位, y->低位            | Bits(w(x)+w(y) bits) |
   | x.subdivideIn(y slices) |             把x分成y个切片, y: Int              |     Vec(Bits, y)     |
   |  x.subdivideIn(y bits)  |             把x按y bits切片, y: Int             |  Vec(Bits, w(x)/y)   |
   |       x,resize(y)       | 返回x的变换大小后的复制, 如果变长则补零, y: Int |     Bits(y bits)     |
   |        x.resized        |         返回一个根据需要自动变换长度的x         |   Bits(w(x) bits)    |
   |     x.resizeLeft(x)     |        保持MSB在同一位置进行放缩, x: Int        |     Bits(x bits)     |

   ```Scala
   println(myBits_32bits.getWidth) //32
   
   myBool := myBits.lsb    //相当于myBits(0)

   //拼接
   myBits_24bits := bits_8bits_1 ## bits_8bits_2 ## bits_8bits_3

   //拆分
   val sel = UInt(2 bits)
   val myBitsWord = myBits_128bits.subdivideIn(32 bits)(sel)
   //sel = 0 => myBitsWord = myBits_128bits(127 downto 96)
   //sel = 1 => myBitsWord = myBits_128bits(95 downto 64)
   //sel = 2 => myBitsWord = myBits_128bits(63 downto 32)
   //sel = 3 => myBitsWord = myBits_128bits(31 downto 0)

   //如果你想反向访问数据
   val myVector = myBits_128bits.subdivideIn(32 bits).reverse
   val myBitsWord = myVector(sel) 

   //长度变换
   myBits_32bits := B"32'x11223344"
   myBits_8bits := myBits_32bits.resized  //自动确定长度(myBits_8bits=0x44)
   myBits_8bits := myBits_32bits.resize(8) //长度变换为(myBits_8bits=0x44)
   myBits_8bits := myBits_32bits.resizeLeft(8) //从高位变换(myBits_8bits=0x11)
   ```

   Verilog:

   ```Verilog
   reg        [31:0]   _zz_myBitsWord;
   reg        [31:0]   myBits_32bits;
   wire                myBool;
   reg        [7:0]    bits_8bits_1;
   reg        [7:0]    bits_8bits_2;
   reg        [7:0]    bits_8bits_3;
   wire       [7:0]    myBits_8bits;
   wire       [7:0]    myBits;
   wire       [23:0]   myBits_24bits;
   wire       [127:0]  myBits_128bits;
   wire       [31:0]   myBitsWord;
   wire       [1:0]    sel;

   always @(*) begin
       case(sel)
           2'b00 : _zz_myBitsWord = myBits_128bits[31 : 0];
           2'b01 : _zz_myBitsWord = myBits_128bits[63 : 32];
           2'b10 : _zz_myBitsWord = myBits_128bits[95 : 64];
       default : _zz_myBitsWord = myBits_128bits[127 : 96];
       endcase
   end

   //反向访问结果, myVector变量被优化掉了, 综合结果并不体现

   always @(*) begin
       case(sel)
           2'b00 : _zz_myBitsWord = myBits_128bits[127 : 96];
           2'b01 : _zz_myBitsWord = myBits_128bits[95 : 64];
           2'b10 : _zz_myBitsWord = myBits_128bits[63 : 32];
           default : _zz_myBitsWord = myBits_128bits[31 : 0];
       endcase
   end

   assign myBool = myBits[0];
   assign myBits_24bits = {{bits_8bits_1,bits_8bits_2},bits_8bits_3};
   assign myBitsWord = _zz_myBitsWord;

   always @(posedge clk or posedge reset) begin
       if(reset) begin
           myBits_32bits <= 32'h0;
       end else begin
           myBits_32bits <= 32'h11223344;
       end
   end

   assign myBits_8bits = myBits_32bits[7:0];
   //高位变换
   assign myBits_8bits = myBits_32bits[31 : 24];

   ```

6. bit位遮罩(MaskedLiteral)

   不关心的值用-遮罩

   ```Scala
   val myBits = B"1101"
   val test1 = myBits === M"1-01"  //True
   val test2 = myBits === M"0---"  //False
   val test3 = myBits === M"1--1"  //True
   ```

   Verilog:

   ```Verilog
   wire       [3:0]    myBits;
   wire                test1;
   wire                test2;
   wire                test3;

   assign myBits = 4'b1101;
   assign test1 = ((myBits & 4'b1011) == 4'b1001);
   assign test2 = ((myBits & 4'b1000) == 4'b0000);
   assign test3 = ((myBits & 4'b1001) == 4'b1001);
   ```
