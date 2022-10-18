## VexRiscv(RV32IM CPU)

VexRiscv是一个适配fpga的RISC-V ISA CPU实现, 具有以下特性:
+ RV32IM指令集
+ 五级流水(Fetch, Decode, Execute, Memory, WriteBack)
+ 当所有功能都启用时达到1.44 DMIPS/Mhz, 
+ 针对FPGA进行了优化
+ 可选MUL / DIV扩展
+ 可选的指令和数据缓存
+ 可选的MMU
+ 可选的调试扩展, 允许eclipse通过GDB >> openOCD >> JTAG连接进行调试
+ 使用riscv-privileged-v1.9.1规范中的Machine和User模式可选的中断以及异常处理。
+ 两个移位指令的实现, 单周期/移数(shiftNumber)循环
+ 每个阶段可能有旁路或联锁的冒险(hazard)逻辑
+ FreeRTOS端口, 请见https://github.com/Dolu1990/FreeRTOS-RISCV

更多信息请见：https://github.com/SpinalHDL/VexRiscv