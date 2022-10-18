## Misc

### 一、Plic映射器

PLIC映射器定义了PLIC(平台级中断控制器)的寄存器生成和访问。

1. `PlicMapper.apply`

   `(bus: BusSlaveFactory, mapping: PlicMapping)(gateways : Seq[PlicGateway], targets : Seq[PlicTarget])`

   PlicMapper参数:
   + 总线:连接此控制的总线
   + 映射:一个映射配置(参见上面)
   + 网关:用于生成总线访问控制的PlicGateway(中断源)序列
   + 目标: 生成总线访问控制的PlicTarget的序列(如:多核)

   它遵循由riscv给出的接口:https://github.com/riscv/riscv-plic-spec/blob/master/riscv-plic.adoc
   
   到目前为止, 有两个内存映射可用:

2. `PlicMapping.sifive`

   遵循SiFive PLIC映射(例如。E31核心复合手册), 基本上是一个成熟的PLIC

3. `PlicMapping.light`

   这种映射以丢失一些可选特性为代价, 产生了一个更轻量级的PLIC:
   + 不读取中断的优先级
   + 不读取中断的悬挂位(必须使用claim/complete机制)
   + 不读取目标的阈值

   剩下的寄存器和逻辑被生成。
