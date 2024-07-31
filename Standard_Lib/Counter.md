## Counter 计数器

------

counter提供了一个计数器，有上界，溢出的时候会回到0，标准语法Counter(N)，N-1是他的上限，0-N-1共N个数  

------
一段示例代码
```scala
class Count extends Module{
  val io = IO (new Bundle{
    val en    = Input(Bool())
    val valid = Output(Bool())
    val out1   = Output(UInt(8.W))
    val out2   = Output(UInt(8.W))
  })

  val (a,b) = Counter(io.en, 233)
  io.out1   := a       //计数器的计数值
  io.valid := b        //达到最大值的指示信号

  val counter = Counter(3)     //最大计数到2
  when(io.en){
   counter.inc()               //调用inc方法，使计数器加1
  }
  io.out2 := counter.value
}
```
> [!NOTE] Counter1
> `Counter(io.en, 233) `创建了一个计数器，当 `io.en` 为 true 时启动计数，最大计数值为 233。a 是计数器的当前值，b 是一个信号，指示计数器是否达到最大值。io.out1 输出 a 的值，`io.valid` 输出 b 的值。

> [!NOTE] Counter2
> `Counter(3)` 创建了一个计数器，最大计数到 2（因为计数器从 0 开始）。

另一段示例代码
```scala
test(new Module {
    val io = IO(new Bundle {
      val count = Input(Bool())
      val out = Output(UInt(2.W))
    })
    val counter = Counter(3)  // 3-count Counter (outputs range [0...2])
    when(io.count) {
      counter.inc()
    }
    io.out := counter.value
    //使用.value来查看counter内部值
  }) { c =>
    //Stage1
    c.io.count.poke(true.B)
    println(s"start: counter value=${c.io.out.peek().litValue}")
    c.clock.step(1)
    //Stage2
    println(s"step 1: counter value=${c.io.out.peek().litValue}")
    c.clock.step(1)
    //Stage3
    println(s"step 2: counter value=${c.io.out.peek().litValue}")
    c.io.count.poke(false.B)
    c.clock.step(1)
    //Stage4
    println(s"step without increment: counter value=${c.io.out.peek().litValue}")
    c.io.count.poke(true.B)
    //Stage5
    c.clock.step(1)
    println(s"step again: counter value=${c.io.out.peek().litValue}")
    //使用litvalue是为了将具有宽度的值，返回一个字面值
}
```
```scala
Elaborating design...
Done elaborating.
start: counter value=0
step 1: counter value=1
step 2: counter value=2
step without increment: counter value=2
step again: counter value=0
test Helper_Anon Success: 0 tests passed in 6 cycles in 0.042299 seconds 141.85 Hz
```