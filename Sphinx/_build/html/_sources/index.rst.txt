.. SpinalHDL_Chinese documentation master file, created by
   sphinx-quickstart on Mon Oct 17 06:53:07 2022.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

.. toctree::
   :maxdepth: 2

   doc/关于SpinalHDL/index
   doc/开始入门/index
   doc/数据类型/index
   doc/结构/index
   doc/语义/index
   doc/时序逻辑/index
   doc/设计错误/index
   doc/其它语言特征/index
   doc/库/index
   doc/仿真/index
   doc/形式验证/index
   doc/例子/index

欢迎来到SpinalHDL中文文档
=============================================

译者序
---------------------------------------------

1. 本团队对SpinalHDL-Doc的翻译工作及其成果非商业目的，所有公司、团队、个人均可无任何限制地免费试用，如有收费行为与本团队无关，为收费者个人行为。


2. 感谢原作者Charles Papon对本团队翻译的支持和认可，更感谢他开发了SpinalHDL，让我们硬件工程师有更加灵活便捷的硬件语言生成工具，让大规模架构的设计变得更加可参数化、可模型化、可配置化。同时本团队也希望该译本能够帮助更多的国内硬件工程师开发和设计灵活的硬件电路，为集成电路设计行业注入新的活力。


3. 该译本尽可能地保证了与原文档表述的一致性，文中出现的绝大多数专用词汇也均有其对应的英文表述，方便大家对照Scala和SpinalHDL进行学习。此外，在原文档的基础上，该中文译本为每个可生成Verilog的SpinalHDL代码增加了其对应的生成结果，方便各位读者对比学习。最后，本团队为EE出身，专注于硬件架构设计，文中如有翻译不到位的地方，请各位读者参考作者的原文档: https://spinalhdl.github.io/SpinalDoc-RTD/master/index.html。此外，请各位读者注意，该译本所翻译的SpinalHDL版本为v1.7.2，不同版本之间可能有所差异，请各位读者在编程的时候注意工程下build.sbt中的版本号。


4. 本团队归属于清华大学尹首一教授课题组可重构计算团队，热烈欢迎大家加入我们团队！

前言
=============================================

网站的目的和结构
------------------

该网站呈现了 *SpinalHDL* 语言并用一些例子告诉读者如何使用它。

如果想要粗略学习该语言, `该展示 <https://cdn.jsdelivr.net/gh/SpinalHDL/SpinalDoc@master/presentation/en/presentation.pdf>`_ 会是一个很好的开始。

什么是SpinalHDL?
----------------

SpinalHDL是一个 `开源 <https://github.com/SpinalHDL/SpinalHDL>`_ 高层次硬件描述语言，可以作为VHDL或Verilog的更好的替代品。

并且，SpinalHDL不是HLS方法，它的目的不是把一些抽象描述引入触发器和门，而是通过用简单的元素(触发器、门、if/case语句)创建抽象层次并帮助设计者复用代码，而不是一遍又一遍地书写它。

.. note::
   SpinalHDL完全可以和标准的基于VHDL/Verilog的EDA工具(仿真器和综合器)交互进而由工具链生成VHDL或Verilog输出。同时它还支持在SpinalHDL模块中引入VHDL或Verilog IPs进行混合设计。

SpinalHDL与VHDL/Verilog相比的优势
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

因为SpinalHDL基于高层次语言，提供了很多优势来改进硬件代码:

#. **没有无穷无尽的wire** - 像AXI一样在一行创建并连接复杂的总线
#. **演化能力** - 创建你自己定义的总线和抽象层
#. **减少代码量** - 通过高层次的系数可以减少代码量，尤其是wire类型。这使你对代码基础由更加好的全局理解，提高生产力并减少令人头疼的部分
#. **免费并且用户友好的IDE** - 多亏自动完成的Scala工具、导航快捷方式、错误高亮、等等
#. **强大并且便捷的类型转换** - 任何数据类型和bits都可以双向翻译，当从CPu层加载复杂数据结构的时候很管用
#. **环路检测** - 工具可以检测没有组合逻辑环/Latch
#. **安全的时钟域** - 工具会告诉你有没有无目的性的跨时钟域传输
#. **范式设计** - 通过使用Scala结构让你的硬件描述没有对泛型的约束

License
--------

SpinalHDL用两个licenses，一个是spinal.core，另一个是spinal.lib，其他的都在repo中

spinal.core(编译器)用的是LGPL许可证，可以总结为以下语句:

* 你可以用SpinalHDL描述和它生成的RTL盈利
* 你不必分享出你的SpinalHDL描述和它生成的RTL
* 没有费用和版税
* 如果你对SpinalHDL core有改进，请共享出来你的改动并让这个工具对大家更加方便

spinal.lib在MIT许可证(通用目的的模块/工具/接口库)

开始！
------

想要亲自尝试一下吗？跳转到 :ref:`开始入门模块 <开始入门>` 开始享受SpinalHDL之旅吧！

Links
-----

| SpinalHDL仓库:
| `https://github.com/SpinalHDL/SpinalHDL <https://github.com/SpinalHDL/SpinalHDL>`_

| 简短的演示(PDF):
| `motivation.pdf <https://cdn.jsdelivr.net/gh/SpinalHDL/SpinalDoc@master/presentation/en/motivation.pdf>`_

| 语言的呈现(PDF):
| `presentation.pdf <https://cdn.jsdelivr.net/gh/SpinalHDL/SpinalDoc@master/presentation/en/presentation.pdf>`_

| SBT base project:
| `https://github.com/SpinalHDL/SpinalTemplateSbt <https://github.com/SpinalHDL/SpinalTemplateSbt>`_

| Jupyter bootcamp:
| `https://github.com/SpinalHDL/Spinal-bootcamp <https://github.com/SpinalHDL/Spinal-bootcamp>`_

| Workshop:
| `https://github.com/SpinalHDL/SpinalWorkshop <https://github.com/SpinalHDL/SpinalWorkshop>`_

| VexRiscv CPU和SoC:
| `https://github.com/SpinalHDL/VexRiscv <https://github.com/SpinalHDL/VexRiscv>`_

| StackOverflow (tag: SpinalHDL) :
| `StackOverflow <https://stackoverflow.com/>`_

| Google group:
| `https://groups.google.com/forum/#!forum/spinalhdl-hardware-description-language <https://groups.google.com/forum/#!forum/spinalhdl-hardware-description-language>`_

| Donation channel :
| `https://opencollective.com/spinalhdl <https://opencollective.com/spinalhdl>`_

| Chinese version of documentation:
| `https://github.com/thuCGRA/SpinalHDL_Chinese_Doc <https://github.com/thuCGRA/SpinalHDL_Chinese_Doc>`_

.. image:: https://badges.gitter.im/SpinalHDL/SpinalHDL.svg
   :target: https://gitter.im/SpinalHDL/SpinalHDL?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge
   :alt: Join the chat at https://gitter.im/SpinalHDL/SpinalHDL



.. image:: https://api.travis-ci.org/SpinalHDL/SpinalHDL.svg?branch=master
   :target: https://travis-ci.org/SpinalHDL/SpinalHDL
   :alt: Build Status


