## IO口

### 一、ReadableOpenDrain

ReadableOpenDrain包的定义如下:

```Scala
case class ReadableOpenDrain[T<: Data](dataType : HardType[T]) extends Bundle with IMasterSlave{
  val write,read : T = dataType()

  override def asMaster(): Unit = {
    out(write)
    in(read)
  }
}
```

然后, 作为主端口, 可以使用`read`信号读取外部值, 并使用`write`信号设置用户希望在输出上驱动的值。

这里有一个使用例子:
```Scala
val io = new Bundle{
  val dataBus = master(ReadableOpenDrain(Bits(32 bits)))
}

io.dataBus.write := 0x12345678
when(io.dataBus.read === 42){

}
```

### 二、三态(Tristate)

1. 简介

   三态信号在很多情况下处理起来都很奇怪:
   + 它们不满足真正的数字性
   + 除了IO, 它们不用于数字设计
   + 三态概念不适合SpinalHDL内部图。

   SpinalHDL为三态信号提供了两种不同的抽象。`TriState`包和`analog and inout`信号。两者都有不同的目的:
   + 三态应用于大多数目的, 特别是在设计中。包包含一个额外的信号来传递当前的方向。
   + `analog and inout`应该用于设备边界上的驱动程序和其他一些特殊情况。请参阅参考文档页了解更多细节。

   如上所述, 推荐的方法是在设计中使用`TriState`。然后在顶层文件中, 将`TriState`包分配给一个`analog and inout`口, 以获得综合工具来推断正确的I/O驱动程序。这可以通过InOutWrapper自动完成, 也可以根据需要手动完成。

2. 三态

   `TriState`定义如下;
   ```Scala
   case class TriState[T <: Data](dataType : HardType[T]) extends Bundle with IMasterSlave{
       val read,write : T = dataType()
       val writeEnable = Bool()

       override def asMaster(): Unit = {
           out(write,writeEnable)
           in(read)
       }
   }
   ```
   主端口可以使用`read`信号来读取外部值, `writeEnable`来启用输出, 最后使用`write`来设置输出驱动的值。
   
   使用样例：
   ```Scala
   val io = new Bundle{
       val dataBus = master(TriState(Bits(32 bits)))
   }

   io.dataBus.writeEnable := True
   io.dataBus.write := 0x12345678
   when(io.dataBus.read === 42){

   }
   ```

3. 三态阵列(TriStateArray)

   在某些情况下, 用户需要控制每个单独的引脚的输出启用(就像GPIO一样)。在这种情况下, 可以使用TriStateArray包。

   其定义如下：
   ```Scala
   case class TriStateArray(width : BitCount) extends Bundle with IMasterSlave{
       val read,write,writeEnable = Bits(width)

       override def asMaster(): Unit = {
           out(write,writeEnable)
           in(read)
       }
   }
   ```

   它与TriState包相同, 不同的是`writeEnable`是一个Bits来控制每个输出缓冲区。

   使用样例：
   ```Scala
   val io = new Bundle{
       val dataBus = master(TriStateArray(32 bits)
   }

   io.dataBus.writeEnable := 0x87654321
   io.dataBus.write := 0x12345678
   when(io.dataBus.read === 42){

   }
   ```
