## IO包

### 一、简介

SpinalHDL会检查每个`IO`包定义内都只有in/out/inout信号。

### 二、例子

下述代码：
```Scala
class TopLevel extends Component {
  val io = new Bundle {
    val a = UInt(8 bits) 
  }
}
```
将会报错：
```
IO BUNDLE ERROR : A direction less (toplevel/io_a :  UInt[8 bits]) signal was defined into toplevel component's io bundle
  ***
  Source file location of the toplevel/io_a definition via the stack trace
  ***
```
修复如下：
```Scala
class TopLevel extends Component {
  val io = new Bundle {
    val a = in UInt(8 bits)
  }
}
```
但是可能对于元(meta)硬件的设计真的需要`io.a`是无方向的, 则可以用：
```Scala
class TopLevel extends Component {
  val io = new Bundle {
    val a = UInt(8 bits)
  }
  a.allowDirectionLessIo
}
```
