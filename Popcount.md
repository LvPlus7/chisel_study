## Popcount

`PopCoun`t以`UInt`形式返回输入中的高(1)位数,就是返回二进制里面有多少个1
```SCALA
test(new Module {
    
    val io = IO(new Bundle {
      val in = Input(UInt(8.W))
      val out = Output(UInt(8.W))
    })
    io.out := PopCount(io.in)
  })
```
*** 结果 ***
- in=0b0, out=0
- in=0b1111, out=4
- in=0b11001010, out=4
- in=0b11111111, out=8
```scala
test(new Module {
  // Example circuit using PopCount
  val io = IO(new Bundle {
    val in = Input(UInt(8.W))
    val out = Output(UInt(8.W))
  })
  io.out := PopCount(io.in)
}) { c =>
  c.io.in.poke(0.U)            // 输入为 0b00000000
  println(s"in=0b${c.io.in.peek().litValue}, out=${c.io.out.peek().litValue}") // 输出为 0
  
  c.io.in.poke("b1111".U)      // 输入为 0b00001111
  println(s"in=0b${c.io.in.peek().litValue}, out=${c.io.out.peek().litValue}") // 输出为 4
  
  c.io.in.poke("b11001010".U)  // 输入为 0b11001010
  println(s"in=0b${c.io.in.peek().litValue}, out=${c.io.out.peek().litValue}") // 输出为 4
  
  c.io.in.poke("b11111111".U)  // 输入为 0b11111111
  println(s"in=0b${c.io.in.peek().litValue}, out=${c.io.out.peek().litValue}") // 输出为 8
  
  c.clock.step(1)  // 时钟步进一次，完成仿真
}
```