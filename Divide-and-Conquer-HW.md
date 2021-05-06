---
title: 电路设计中的分治法 —— 以 Chisel 实现优先编码器为例
date: 2021-01-02 19:25:05
tags:
    - 数字逻辑设计与计算机组成
categories: 数字逻辑
mathjax: true
---

在实现我的毕业设计时，遇到了这么一个需求：找出一组数字中最低（高）位的 `1` 的位置 —— 换句话说，就是一个优先编码器。显然最基本的版本实现起来很简单，尤其是我们还是在使用 `Chisel` 这么个懒人神器的情况下，初代代码如下（仅作为例子，下同），一行就解决了问题。

```scala
class Example extends Module {
  val io = IO(new Bundle() {
    val i = Input(UInt(64.W))
    val o = Output(UInt(log2Ceil(64).W))
  })
  io.o := PriorityEncoder(io.i)
}
```

但是我们看生成的 Verilog 以及 `yosys` 的报告，发现资源消耗比较大，时序算下来也能把我血压给拉满。

以下是生成的 Verilog 的一部分，可以发现就是一组串联的 Mux，时序绝对会爆炸。

```verilog
module Example(
  input         clock,
  input         reset,
  input  [63:0] io_i,
  output [5:0]  io_o
);
  wire [5:0] _io_o_T_64 = io_i[62] ? 6'h3e : 6'h3f; // @[Mux.scala 47:69]
  wire [5:0] _io_o_T_65 = io_i[61] ? 6'h3d : _io_o_T_64; // @[Mux.scala 47:69]
  wire [5:0] _io_o_T_66 = io_i[60] ? 6'h3c : _io_o_T_65; // @[Mux.scala 47:69]
  wire [5:0] _io_o_T_67 = io_i[59] ? 6'h3b : _io_o_T_66; // @[Mux.scala 47:69]
  ...
  wire [5:0] _io_o_T_121 = io_i[5] ? 6'h5 : _io_o_T_120; // @[Mux.scala 47:69]
  wire [5:0] _io_o_T_122 = io_i[4] ? 6'h4 : _io_o_T_121; // @[Mux.scala 47:69]
  wire [5:0] _io_o_T_123 = io_i[3] ? 6'h3 : _io_o_T_122; // @[Mux.scala 47:69]
  wire [5:0] _io_o_T_124 = io_i[2] ? 6'h2 : _io_o_T_123; // @[Mux.scala 47:69]
  wire [5:0] _io_o_T_125 = io_i[1] ? 6'h1 : _io_o_T_124; // @[Mux.scala 47:69]
  assign io_o = io_i[0] ? 6'h0 : _io_o_T_125; // @[Mux.scala 47:69]
endmodule
```

### 分治法

后来我意识到，可以把分治思想应用到我们的电路中，也就是把输入分为左右两部分，分别处理，这样应该能把之前的串联的 $O(N)$ 层 Mux 串起来的电路转化为 $O(log N)$ 层的电路，以此可以获得更好的时序。话不多说，直接看代码，相信大家都可以看懂。

```scala
object BinaryPriorityEncoder {
  def apply(in: Bits): Valid[UInt] = {
    require(in.getWidth > 0)
    val roundedInputWidth = 1 << log2Ceil(in.getWidth)
    print(roundedInputWidth)
    val roundedInput = Wire(UInt(roundedInputWidth.W))
    roundedInput := in
    checkedApply(roundedInput, roundedInputWidth)
  }

  def checkedApply(in: UInt, width: Int): Valid[UInt] = {
    val outputWidth = log2Ceil(width)
    val result = Wire(Valid(UInt(outputWidth.W)))

    if (width == 2) {
      val idx = in(1) && (~in(0)).asBool
      val valid = in.orR()
      result.bits := idx
      result.valid := valid
    } else {
      val leftWidth = width >> 1
      val leftResult = checkedApply(in(leftWidth - 1, 0), leftWidth)
      val rightResult = checkedApply(in(width - 1, leftWidth), leftWidth)

      result.valid := leftResult.valid || rightResult.valid
      result.bits := Mux(leftResult.valid, leftResult.bits, Cat(rightResult.valid, rightResult.bits))
    }
    result
  }
}
```

经过验证，结果一致。