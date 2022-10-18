## Bool

### 一、描述(Description)

`Bool`数据类型对应于布尔值(True/False)

### 二、声明(Declaration)

声明布尔值的语法如下所示：([]中的内容可填项)

|        语法         |                      描述                      | 返回类型 |
| :-----------------: | :--------------------------------------------: | :------: |
|      Bool[()]       |                   创建Bool量                   |   Bool   |
|        True         |             创建值为`true`的Bool量             |   Bool   |
|        False        |            创建值为`false`的Bool量             |   Bool   |
| Bool(value:Boolean) | 创建一个值为Scala Boolean(true, false)的Bool量 |

```Scala
val myBool_1 = Bool()   //创建一个Bool
myBool_1 := False       //:=是赋值操作符

val myBool_2 = False    //与上一行等价

val myBool_3 = Bool(5 > 12) //用Scala Boolean类型创建Bool
```

Verilog:

```Verilog
wire                myBool_1;
wire                myBool_2;
wire                myBool_3;
reg        [7:0]    counter;

assign myBool_1 = 1'b0;
assign myBool_2 = 1'b0;
assign myBool_3 = 1'b0;
```

### 三、操作符(Operators)

下列操作符可以用`Bool`类型：

1. 逻辑运算(Logic)

|             操作符             |          描述           | 返回类型 |
| :----------------------------: | :---------------------: | :------: |
|               !x               |         逻辑非          |   Bool   |
|        x && y<br>x & y         |         逻辑与          |   Bool   |
| x &#124;&#124; y<br>x &#124; y |         逻辑或          |   Bool   |
|             x ^ y              |        逻辑异或         |   Bool   |
|           x.set[()]            |        置位True         |          |
|          x.clear[()]           |        置位False        |          |
|        x.setWhen(cond)         |      条件True置位1      |   Bool   |
|       x.clearWhen(cond)        |      条件True置位0      |   Bool   |
|        x.riseWhen(cond)        | x是False且条件True置位1 |   Bool   |
|        x.fallWhen(cond)        | x是True且条件True置位0  |   Bool   |

```Scala
val a, b, c = Bool()
val res = (!a & b) ^ c

val d = False
when(cond) {
    d.set()     //等价于d := True
}

val e = False
e.setWhen(cond) //等价于when(cond) { d := True }

val f = RegInit(False) fallWhen(ack) setWhen(req)
/*
*等价于
*when(f && ack) { f := False}
*when(req) {f := True}
*or
*f := req || (f && !ack)
*/

//注意赋值顺序!
val g = RegInit(False) setWhen(req) fallWhen(ack)
//等价于 g := ((!g) && req) || (g && !ack)
```

Verilog:

```Verilog
  wire                a;
  wire                b;
  wire                c;

  assign res = (((! a) && b) ^ c);

  reg                 d;
  always @(*) begin
    d = 1'b0;
    if(io_cond0) begin
      d = 1'b1;
    end
  end

  reg                 e;
  always @(*) begin
    e = 1'b0;
    if(cond) begin
      e = 1'b1;
    end
  end

  reg                 f;
  assign when_MyTopLevel_l47 = (f && ack);
  always @(posedge clk or posedge reset) begin
    if(reset) begin
      f <= 1'b0;
    end else begin
      if(when_MyTopLevel_l47) begin
        f <= 1'b0;
      end
      if(req) begin
        f <= 1'b1;
      end
    end
  end

  reg                 g;
  assign when_MyTopLevel_l49 = (g && ack);
  always @(posedge clk or posedge reset) begin
    if(reset) begin
      g <= 1'b0;
    end else begin
      if(req) begin
        g <= 1'b1;
      end
      if(when_MyTopLevel_l49) begin
        g <= 1'b0;
      end
    end
  end
```

2. 边沿检测(Edge detection)

|        操作符        |                    描述                    | 返回类型  |
| :------------------: | :----------------------------------------: | :-------: |
|      x.edge[()]      |            当x发生跳变返回True             |   Bool    |
| x.edge(initAt: Bool) |  与x.edge相同但多了复位值, 对reg_next复位  |   Bool    |
|      x.rise[()]      | 当x在上一周期低电平下一周期高电平返回True  |   Bool    |
| x.rise(initAt: Bool) |  与x.rise相同但多了复位值, 对reg_next复位  |   Bool    |
|      x.fall[()]      | 当x在上一周期高电平下一周期低电平返回False |   Bool    |
|     x.edges[()]      |         返回包(rise, fall ,toggle)         | BoolEdges |
| x.edges(initAt:Bool) | 与x.edges相同但多了复位值, 对reg_next复位  | BoolEdges |

```Scala
when(myBool_1.rise(False)) {
    //当检测到上升沿时做点什么
}

val edgeBundle = myBool_2.edges(False)
when(edgeBundle.rise) {
    //当检测到上升沿时做点什么
}
when(edgeBundle.fall) {
    //当检测到下降沿时做点什么
}
when(edgeBundle.toggle) {
    //当检测到边沿时做点什么
}
```

3. 比较操作(Comparison)

| 操作符  |  描述  | 返回类型 |
| :-----: | :----: | :------: |
| x === y |  相等  |   Bool   |
| x =/= y | 不相等 |   Bool   |

```Scala
when(myBool) {
    //等价于when(myBool === True)
}
    
when(!myBool) {
    //等价于when(myBool === False)
}
```

4. 类型转换(Type cast)

|       操作符       |            描述            |      返回类型       |
| :----------------: | :------------------------: | :-----------------: |
|      x.asBits      |      二进制转换为Bits      |   Bits(w(x) bits)   |
|      x.asUInt      |      二进制转换为UInt      |   UInt(w(x) bits)   |
|      x.asSInt      |      二进制转换为SInt      |   SInt(w(x) bits)   |
| x.asUInt(bitCount) | 二进制转换为UInt并改变大小 | UInt(bitCount bits) |
| x.asBits(bitCount) | 二进制转换为Bits并改变大小 | Bits(bitCount bits) |

```Scala
//给SInt值进位
val carry = Bool()
val res = mySInt + carry.asSInt
```

Verilog:

```Verilog
  reg        [7:0]    mySInt;
  assign _zz_res_1 = carry;
  assign _zz_res = {{7{_zz_res_1[0]}}, _zz_res_1};
  assign res = ($signed(mySInt) + $signed(_zz_res));
```

5. Misc

| 操作符 |          描述          |       返回类型       |
| :----: | :--------------------: | :------------------: |
| x ## y | 拼接, x->高位, y->低位 | Bits(w(x)+w(y) bits) |

```Scala
val a, b, c = Bool

//把三个Bool拼接成Bits
val myBits = a ## b ## c
```

Verilog:

```Verilog
  wire                a;
  wire                b;
  wire                c;
  wire       [2:0]    myBits;

  assign myBits = {{a,b},c};
```