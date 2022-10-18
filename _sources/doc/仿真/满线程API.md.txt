## 满线程API(Thread-full API)

在SpinalSim中, 你可以用多线程写testbench, 这种方式比较像SystemVerilog, 也有点像VHDL/Verilog process/always块。这允许你写并行人物并用流式API控制仿真时间。

### 一、仿真线程的分叉和汇合(Fork and Join simulation threads)

```Scala
//建立新线程
val myNewThread = fork {
  // New simulation thread body
}

//一直等待`myNewThread`直到完成执行
myNewThread.join()
```

### 二、休眠和等待(Sleep and WaitUntil)

```Scala
//睡眠1000个单位时间
sleep(1000)

//在继续进行之前一直等待直到dut.io.a大于42
waitUntil(dut.io.a > 42)
```
