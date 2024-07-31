## Aribiters 仲裁

多对一的选择器,从多个`Flipped(Decoupled)`接口选择数据，从一个`Decoupled`接口输出数据，给予优先级，分为两类。

- `Arbiter`: prioritizes lower-index producers，关注索引较低的供应者
- `RRArbiter`: runs in round-robin order  RR算法

举例代码

```scala
class Arbiters extends Module{
  val io = IO(new Bundle {
    val in = Flipped(Vec(2, Decoupled(UInt(8.W))))
    val out = Decoupled(UInt(8.W))
  })

  val arbiter = Module(new Arbiter(UInt(8.W), 2))  // 2 to 1 Priority Arbiter
  arbiter.io.in <> io.in
  io.out <> arbiter.io.out

}
```
```verilog
//verilog接口
module Arbiter(
  output       io_in_0_ready,
  input        io_in_0_valid,
  input  [7:0] io_in_0_bits,
  output       io_in_1_ready,
  input        io_in_1_valid,
  input  [7:0] io_in_1_bits,
  input        io_out_ready,
  output       io_out_valid,
  output [7:0] io_out_bits
);
```

```scala
test(new Module {
    // Example circuit using a priority arbiter
    val io = IO(new Bundle {
      val in = Flipped(Vec(2, Decoupled(UInt(8.W))))
      val out = Decoupled(UInt(8.W))
    })

    val arbiter = Module(new Arbiter(UInt(8.W), 2))  

    // 2 to 1 Priority Arbiter
    //必须使用Module(new Arbiter(  , )),而不能直接使用new Arbiter

    arbiter.io.in <> io.in
    io.out <> arbiter.io.out
    //arbiter的decoupled的输入接入module的输入
    //arbiter的decoupled的输出接入module的输出

  }) { c =>
    //Stage1
    c.io.in(0).valid.poke(false.B) //不进入数据,数据无效
    c.io.in(1).valid.poke(false.B) //不进入数据,数据无效
    c.io.out.ready.poke(false.B)   //也不出数据,接收端没准备好
    
    println(s"Start:")
    println(s"\tin(0).ready=${c.io.in(0).ready.peek().litValue}, in(1).ready=${c.io.in(1).ready.peek().litValue}")
    //此时输入端口的两个ready数据都是1,因为里面没有数,所以准备好接受
    println(s"\tout.valid=${c.io.out.valid.peek().litValue}, out.bits=${c.io.out.bits.peek().litValue}")
    //此时输出端口的valid应该是0,因为并没有有效的数据bits=0

    //Stage2
    c.io.in(1).valid.poke(true.B)   // Valid input 1
    c.io.in(1).bits.poke(42.U)      //从1端口进入数据42
    c.io.out.ready.poke(true.B)     //允许读出数据
    
    println(s"valid input 1:")
    println(s"\tin(0).ready=${c.io.in(0).ready.peek().litValue}, in(1).ready=${c.io.in(1).ready.peek().litValue}")
    //此时的io.in(0).ready=1，是因为io.in(0)的bits没有数据，只有io.in(1)的bits是有数据的，所以此时的ready就不是1了
    println(s"\tout.valid=${c.io.out.valid.peek().litValue}, out.bits=${c.io.out.bits.peek().litValue}")
    //valid是1，是因为两个输入端口之一有数据，bits应该是42

    //Stage3
    c.io.in(0).valid.poke(true.B)  //0端口接受数据
    c.io.in(0).bits.poke(43.U)     //0端口的数据流给43,1端口给的数据是42
   
    println(s"valid inputs 0 and 1:")
    println(s"\tin(0).ready=${c.io.in(0).ready.peek().litValue}, in(1).ready=${c.io.in(1).ready.peek().litValue}")
    //两个都为0,因为bits都是有数据的,并不能准备接受新的数据
    println(s"\tout.valid=${c.io.out.valid.peek().litValue}, out.bits=${c.io.out.bits.peek().litValue}")
    //valid=1,因为里面是有数据可以输出的.输出的数据流应该是低位优先,因此给43

    //Stage4
    c.io.in(1).valid.poke(false.B)  // Valid input 0
    
    println(s"valid input 0:")
    println(s"\tin(0).ready=${c.io.in(0).ready.peek().litValue}, in(1).ready=${c.io.in(1).ready.peek().litValue}")
    println(s"\tout.valid=${c.io.out.valid.peek().litValue}, out.bits=${c.io.out.bits.peek().litValue}")
}
```

```scala
Start:
	in(0).ready=0, in(1).ready=0
	out.valid=0, out.bits=0
valid input 1:
	in(0).ready=1, in(1).ready=1
	out.valid=1, out.bits=42
valid inputs 0 and 1:
	in(0).ready=1, in(1).ready=0
	out.valid=1, out.bits=43
valid input 0:
	in(0).ready=1, in(1).ready=0
	out.valid=1, out.bits=43
```