## Vec

### 一、描述(Description)

`Vec`是定义了一组带有标号的信号的复合信号(基于SpinalHDL基础类别)

### 二、声明(Declaration)

声明向量的语法如下：

|            声明            |                           描述                           |
| :------------------------: | :------------------------------------------------------: |
| Vec(type: Data, size: Int) |         创建一个能容纳`size`个`data`类元素的向量         |
|       Vec(x, y, ...)       | 创建一个标号对应给定元素的向量, 这种方式支持混合元素宽度 |

#### Examples

```Scala
//创建一个有两个有符号整型的向量
val myVecOfSInt = Vec(SInt(8 bits), 2)
myVecOfSInt(0) := 2
myVecOfSInt(1) := myVecOfSInt(0) + 3

//创建一个有三个不同类型元素的向量
val myVecOfMixedInt = Vec(UInt(3 bits), UInt(5 bits), UInt(8 bits))

val x, y, z = UInt(8 bits)
val myVecOf_xyz_ref = Vec(x, y, z)

//向量的迭代
for(element <- myVecOf_xyz_ref) {
    element := 0    //给所有元素赋值0
}

//向量Map映射
myVecOfMixedInt.map(_ := 0) //给所有元素赋值0

//给向量的第一个元素赋值3
myVecOf_xyz_ref(1) := 3
```

Verilog:

```Verilog
  wire       [7:0]    _zz_myVecOfSInt_1;
  wire       [7:0]    myVecOfSInt_0;
  wire       [7:0]    myVecOfSInt_1;
  wire       [2:0]    myVecOfMixedInt_0;
  wire       [4:0]    myVecOfMixedInt_1;
  wire       [7:0]    myVecOfMixedInt_2;
  wire       [7:0]    x;
  wire       [7:0]    y;
  wire       [7:0]    z;
  reg        [7:0]    counter;

  assign _zz_myVecOfSInt_1 = 8'h03;
  assign myVecOfSInt_0 = 8'h02;
  assign myVecOfSInt_1 = ($signed(myVecOfSInt_0) + $signed(_zz_myVecOfSInt_1));
  assign x = 8'h0;
  assign y = 8'h0;
  assign z = 8'h0;
  assign myVecOfMixedInt_0 = 3'b000;
  assign myVecOfMixedInt_1 = 5'h0;
  assign myVecOfMixedInt_2 = 8'h0;

```

### 三、操作符(Operators)

以下操作符是`Vec`类型所支持的：

1. 比较操作(Comparison)

   | 操作符 |  描述  | 返回类型 |
   | :----: | :----: | :------: |
   | x===y  |  相等  |   Bool   |
   | x=/=y  | 不相等 |   Bool   |

   ```Scala
   //创建一个有两个有符号整型的向量
   val vec2 = Vec(SInt(8 bits), 2)
   val vec1 = Vec(SInt(8 bits), 2)

   myBool := vec2 === vec1 //比较两个向量
   ```

   Verilog:

   ```Verilog
   wire       [7:0]    vec2_0;
   wire       [7:0]    vec2_1;
   wire       [7:0]    vec1_0;
   wire       [7:0]    vec1_1;
   wire                myBool;

   assign myBool = (($signed(vec2_0) == $signed(vec1_0)) && ($signed(vec2_1) == $signed(vec1_1)));

   ```

2. 类型转换(Type cast)

   |  操作符  |       描述       |    返回类型     |
   | :------: | :--------------: | :-------------: |
   | x.asBits | 二进制转换为Bits | Bits(w(x) bits) |

   ```Scala
   //创建一个有两个有符号整型的向量
   val vec1 = Vec(SInt(8 bits), 2)

   myBits_16bits := vec1.asBits
   ```

3. Misc

   |     操作符     |     描述      | 返回类型 |
   | :------------: | :-----------: | :------: |
   | x.getBitsWidth | 返回Vec的长度 |   Int    |

   ```Scala
   //创建一个有两个有符号整型的向量
   val vec1 = Vec(SInt(8 bits), 2)

   println(vec1.getBitsWidth)  //16

4. 库辅助函数(Lib Helper functions)

   > 使用这些函数, 需要`import spinal.lib._`

   |               操作符               |                                   描述                                   | 返回类型 |
   | :--------------------------------: | :----------------------------------------------------------------------: | :------: |
   |    x.sCount(condition: T=>Bool)    |                     计算Vec中与给定条件相匹配的个数                      |   UInt   |
   |         x.sCount(value: T)         |                          计算与给定值相等的个数                          |   UInt   |
   |   x.sExists(condition: T=>Bool)    |                        检查Vec中是否有匹配的条件                         |   Bool   |
   |       x.sContains(value: T)        |                       检查Vec中是否有给定的值存在                        |   Bool   |
   |  x.sFindFirst(condition: T=>Bool)  |        找到Vec中第一个与给定条件相匹配的元素, 返回那个元素的标号         |   UInt   |
   | x.reduceBalancedTree(op:(T, T)=>T) | 平衡化的归约函数, 能尽量减小生成电路的深度。`op`应该是可交换的且可结合的 |    T     |
   | x.shuffle(indexMapping: Int=>Int)  |              用一个Map函数把旧索引映射到新索引上, 对Vec重组              |  Vec[T]  |

   ```Scala
   import spinal.lib._

   //创建有4个无符号整型的向量
   val vec1 = Vec(UInt(8bits), 4)

   //...向量在某些地方被赋值后

   val c1: UInt = vec1.sCount(_ < 128)     //向量中有多少值小于128
   val c2: UInt = vec1.sCount(0)           //向量中有多少0值

   val b1: Bool = vec1.sExist(_ > 250)    //向量中有没有比250大的
   val b2: Bool = vec1.sContains(0)        //向量中有没有0

   val u1: UInt = vec1.sFindFirst(_ < 10)  //得到第一个比10小的元素的标号, 该函数在转化时有问题, 有待商榷
   val u2: UInt = vec1.reduceBalancedTree(_ + _)   //向量求和
   ```

   Verilog:

   ```Verilog
   wire       [1:0]    _zz_c1;
   wire       [1:0]    _zz_c1_1;
   wire       [0:0]    _zz_c1_2;
   wire       [2:0]    _zz_c1_3;
   wire       [1:0]    _zz_c1_4;
   wire       [1:0]    _zz_c1_5;
   wire       [0:0]    _zz_c1_6;
   wire       [1:0]    _zz_c2;
   wire       [1:0]    _zz_c2_1;
   wire       [0:0]    _zz_c2_2;
   wire       [2:0]    _zz_c2_3;
   wire       [1:0]    _zz_c2_4;
   wire       [1:0]    _zz_c2_5;
   wire       [0:0]    _zz_c2_6;
   wire       [7:0]    _zz_u2;
   wire       [7:0]    _zz_u2_1;
   wire       [7:0]    vec1_0;
   wire       [7:0]    vec1_1;
   wire       [7:0]    vec1_2;
   wire       [7:0]    vec1_3;
   wire       [2:0]    c1;
   wire       [2:0]    c2;
   wire                b1;
   wire                b2;
   wire       [7:0]    u2;
   reg        [7:0]    counter;

   assign _zz_c1 = ({1'b0,(vec1_0 < 8'h80)} + _zz_c1_1);
   assign _zz_c1_2 = (vec1_1 < 8'h80);
   assign _zz_c1_1 = {1'd0, _zz_c1_2};
   assign _zz_c1_4 = ({1'b0,(vec1_2 < 8'h80)} + _zz_c1_5);
   assign _zz_c1_3 = {1'd0, _zz_c1_4};
   assign _zz_c1_6 = (vec1_3 < 8'h80);
   assign _zz_c1_5 = {1'd0, _zz_c1_6};
   assign _zz_c2 = ({1'b0,(vec1_0 == 8'h0)} + _zz_c2_1);
   assign _zz_c2_2 = (vec1_1 == 8'h0);
   assign _zz_c2_1 = {1'd0, _zz_c2_2};
   assign _zz_c2_4 = ({1'b0,(vec1_2 == 8'h0)} + _zz_c2_5);
   assign _zz_c2_3 = {1'd0, _zz_c2_4};
   assign _zz_c2_6 = (vec1_3 == 8'h0);
   assign _zz_c2_5 = {1'd0, _zz_c2_6};
   assign _zz_u2 = (vec1_0 + vec1_1);
   assign _zz_u2_1 = (vec1_2 + vec1_3);
   assign c1 = ({1'b0,_zz_c1} + _zz_c1_3);
   assign c2 = ({1'b0,_zz_c2} + _zz_c2_3);
   assign b1 = ((((1'b0 || (8'hfa < vec1_0)) || (8'hfa < vec1_1)) || (8'hfa < vec1_2)) || (8'hfa < vec1_3));
   assign b2 = ((((1'b0 || (vec1_0 == 8'h0)) || (vec1_1 == 8'h0)) || (vec1_2 == 8'h0)) || (vec1_3 == 8'h0));
   assign u2 = (_zz_u2 + _zz_u2_1);

   ```

   > 备注：sXXX前缀用来与接受lambda函数作为参数的同名Scala函数的歧义
