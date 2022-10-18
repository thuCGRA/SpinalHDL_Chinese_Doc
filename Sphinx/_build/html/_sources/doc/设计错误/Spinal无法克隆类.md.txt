## Spinal无法克隆类(Spinal can’t clone class)

### 一、简介

当SpinalHDL想要通过`cloneOf`函数创建一个新的数据类型实例, 但是不能这样做时, 就会发生这个错误。出现这种情况的原因总是因为它无法检索`Bundle`的构造参数。

### 二、例子1

下述代码：
```Scala
// cloneOf(this)无法检索用于构造自身的位宽
class RGB(width : Int) extends Bundle {
  val r, g, b = UInt(width bits)
}

class TopLevel extends Component {
  val tmp = Stream(new RGB(8)) // Stream需要cloneOf(new RGB(8))的功能
}
```

将会报错：
```Scala
*** Spinal can't clone class spinal.tester.PlayDevMessages$RGB datatype
*** You have two way to solve that :
*** In place to declare a "class Bundle(args){}", create a "case class Bundle(args){}"
*** Or override by your self the bundle clone function
  ***
  Source file location of the RGB class definition via the stack trace
  ***
```

修复为：
```Scala
case class RGB(width : Int) extends Bundle {
  val r, g, b = UInt(width bits)
}

class TopLevel extends Component {
  val tmp = Stream(RGB(8))
}
```

### 二、例子2

下列代码：
```Scala
case class Xlen(val xlen: Int) {}

case class MemoryAddress()(implicit xlenConfig: Xlen) extends Bundle {
    val address = UInt(xlenConfig.xlen bits)
}

class DebugMemory(implicit config: Xlen) extends Component {
    val io = new Bundle {
        val inputAddress = in(MemoryAddress())
    }

    val someAddress = RegNext(io.inputAddress) // -> ERROR *****************************
}
```

报错：
```
[error] *** Spinal can't clone class debug.MemoryAddress datatype
```
在这种情况下, 一种解决方案是覆盖克隆函数来传播隐式参数。
```Scala
case class MemoryAddress()(implicit xlenConfig: Xlen) extends Bundle {
  val address = UInt(xlenConfig.xlen bits)

  override def clone = MemoryAddress()
}
```
> **注意：我们需要克隆的是硬件的单元, 而不是最终在其中赋值的值**

> **注意：另一种方法是使用ScopeProperty(请见"其他语言特性"章节中)**
