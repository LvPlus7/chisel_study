## Decoupled Interfaces
举例
```scala
class QueueModule[T <: Data](ioType: T, entries: Int) extends MultiIOModule {
  val in = IO(Flipped(Decoupled(ioType)))
  val out = IO(Decoupled(ioType))
  out <> Queue(in, entries)
}
```

1. Decoupled：给bit数据包裹上一层握手协议，decoupled 默认方向为输出，如果需要输入，可以加Flipped.  
2. Decoupled 可以直接调用Bundle或者Bits，Bundle 内的端口也应该用bits定义。  
3. Ready应该只取决于接收器是否能够接收数据，valid应该只取决于源是否有数据。只有在事务完成之后(在下一个时钟周期中)，这些值才应该更新。
```scala
val myChiselData = UInt(8.W)
val myDecoupled = Decoupled(myChiselData) //在这里包上了一层协议

*** 结果 ***
valid: Output(Bool)
ready: Input(Bool)
bits: Output(UInt(8.W))
```
> [!NOTE]
> https://zhuanlan.zhihu.com/p/594189325
> "知乎专栏Decoupled"




示例代码
```scala
class testIO extends Bundle {
val a = Bits(3.W)
val b = Bits(2.W)
}

//定义了两个信号的捆绑
class testmodule extends Module{
val io = IO( new Bundle{
   val test = Decoupled(new testIO)
})

io.test.valid := io.test.ready
//一般直接定义ready 和 valid相连接是为了说明对方可以接受,自己的数据就有效,对方愿意收,就立刻发
io.test.bits.a := 1.U
io.test.bits.b := 1.U
}

println(getVerilog(new testmodule))
```



```scala
class Producer extends Module {
  val io = IO(new Bundle {
    val out = Decoupled(UInt(8.W))
  })

  //定义out,是Decoupled接口
  val data = RegInit(0.U(8.W))

  //定义8位的寄存器,初始值为0
  io.out.valid := true.B  //自己的数据始终有效
  io.out.bits := data     //data寄存器的值作为数据流输出
  when(io.out.ready) {    //当接收端准备好接收数据时...
    data := data + 1.U
  }
}

class Consumer extends Module {
  val io = IO(new Bundle {
    val in = Flipped(Decoupled(UInt(8.W)))
  })

  //定义in,是Decoupled接口
  when(io.in.valid && io.in.ready) {

    //输入接口同时有效（valid）且准备好（ready）
    printf("Received data: %d\n", io.in.bits)

  }
}

class DecoupledExample extends Module {
  val io = IO(new Bundle {
    val out = Decoupled(UInt(8.W))
    val in = Flipped(Decoupled(UInt(8.W)))
    //在这里就是定义数据流
  })

  //在这里定义一个连接模块
  val producer = Module(new Producer)
  val consumer = Module(new Consumer)

  //用<>表示模块的连接
  producer.io.out <> io.out
  consumer.io.in <> io.in
}
//????????????????????????????????????????
//bits的连接????

```
> [!NOTE]
> ??????