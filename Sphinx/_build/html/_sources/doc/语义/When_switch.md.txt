## When/Switch/Mux

### 一、When
正如在VHDL和Verilog中, 信号可以根据条件是否符合来选择性地赋值：

```Scala
when(cond1) {
    //当cond1真时执行操作
}.elsewhen(cond2) {
    //当cond1假但cond2真时执行操作
}.otherwise {
    //cond1和cond2同假时执行操作
}
```

> 注意：如果`otherwise`关键字和花括号的后半部分`}`在同一行, `.`可以省略
> ```Scala
> when(cond1) {
>   //当cond1真时执行操作
> } otherwise {
>   //cond1和cond2同假时执行操作
> }
> ```
> 但如果`.otherwise`在另一行, 需要`.`
> ```Scala
> when(cond1) {
>   //当cond1真时执行操作
> }
> .otherwise {
>   //cond1和cond2同假时执行操作
> }
> ```

### 二、Switch

就像在VHDL和Verilog, 当信号得到固定值的时候可以条件执行运算：

```Scala
switch(x) {
    is(value1) {
        //当x===value1执行
    }
    is(value2) {
        //当x===value2执行
    }
    default {
        //当之前的条件都没有符合执行
    }
}
```

`is`子句可以用`is(value1, value2)`这种逗号分割的方式书写。

举例：

```Scala
switch(aluop) {
    is(ALUOp.add) {
        immediate := instruction.immI.signExtend
    }
    is(ALUOp.slt) {
        immediate := instruction.immI.signExtend
    }
    is(ALUOp.sltu) {
        immediate := instruction.immI.signExtend
    }
    is(ALUOp.sll) {
        immediate := instruction.shamt
    }
    is(ALUOp.sra) {
        immediate := instruction.shamt
    }
}
```

上述代码等价于

```Scala
switch(aluop) {
    is(ALU0p.add, ALU0p.slt, ALU0p.sltu) {
        immediate := instruction.immI.signExtend
    }
    is(ALU0p.sll, ALU0p.sra) {
        immediate := instruction.shamt
    }
}
```

### 三、本地声明(Local declaration)

也可以在when/switch描述内定义新的信号：

```Scala
val x, y = UInt(4 bits)
val a, b = UInt(4 bits)

when(cond) {
    val tmp = a + b
    x := tmp
    y := tmp + 1
} otherwise {
    x := 0
    y := 0
}
```

> 备注：SpinalHDL会检查信号是否只在该区域内被定义。

### 四、Mux

如果你只需要带有`Bool`选择信号的`Mux`, 有两种等价的语句形式：

|               语句               | 返回类型 |                       描述                        |
| :------------------------------: | :------: | :-----------------------------------------------: |
|  Mux(cond, whenTrue, whenFalse)  |    T     | 当`cond`为真, 返回`whenTrue`, 否则返回`whenFalse` |
| cond ? whenTrue &#124; whenFalse |    T     | 当`cond`为真, 返回`whenTrue`, 否则返回`whenFalse` |

```Scala
val cond = Bool
val whenTrue, whenFalse = UInt(8 bits)
val muxOutput   = Mux(cond, whenTrue, whenFalse)
val muxOutput2  = cond ? whenTrue | whenFalse
```

### 五、Bit级选择(Bitwise selection)

bit级的选择看起来像VHDL中`when`语句。

**举例**

```Scala
val bitwiseSelect = UInt(2 bits)
val bitwiseResult = bitwiseSelect.mux(
    0 -> (io.src0 & io.src1),
    1 -> (io.src0 | io.src1),
    2 -> (io.src0 ^ io.src1),
    default -> (io.src0)
)
```

同样, 如果条件是完备的, 不需要些default：

```Scala
val bitwiseSelect = UInt(2 bits)
val bitwiseResult = bitwiseSelect.mux(
    0 -> (io.src0 & io.src1),
    1 -> (io.src0 | io.src1),
    2 -> (io.src0 ^ io.src1),
    3 -> (io.src0)
)
```

或者, 如果未覆盖的值并不重要, 他们可以用`muxListDc`使其不赋值。

`muxLists(...)`是另一种bit级选择器, 该选择器以元组作为输入, 下图是一个把128 bits分割成32 bits的例子。

![MuxList](../../img/MuxList.png)

```Scala
val sel   = UInt(2 bits)
val data  = Bits(128 bits)

//把宽bit类型分成小块, 可以用mux：
val dataWord = sel.muxList(for (index <- until 4) yield (index, data(index*32+32-1 downto index*32)))

//一种书写更短的方式书写上述代码：
val dataWord = data.subdivideIn(32 bits)(sel)
```
