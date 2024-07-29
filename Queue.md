## `Queue`

Queue创建了一个`FIFO`(先入先出)队列，在两边都有`Decoupled`的接口，允许反压。元素的数据类型和数量都是可配置的。
用法如下：
```scala
// enq为队列的源，entries为队列的元素个数
val queue = Queue(enq: DecoupledIO, entries: Int)
```
下面这是一段实现`FIFO`的代码
```scala
class Fifo extends Module{
  val io = IO (new Bundle{
    val in = Flipped(Decoupled(UInt(8.W)))
    val out = Decoupled(UInt(8.W))
  })

  val queue = Queue(io.in, 2)  // 2-element queue
  //输入输出接口不算在队列的深度内
  io.out <> queue
}
```
```verilog
//verilog
module Queue(
  input        clock,
  input        reset,
  output       io_enq_ready,
  input        io_enq_valid,
  input  [7:0] io_enq_bits,
  input        io_deq_ready,
  output       io_deq_valid,
  output [7:0] io_deq_bits
);
```
另外一段示例代码
```scala
class QueueModule[T <: Data](ioType: T, entries: Int) extends MultiIOModule {
  val in = IO(Flipped(Decoupled(ioType)))
  val out = IO(Decoupled(ioType))
  out <> Queue(in, entries)
}
//一队列,输入是用decoupled in 输出是用decoupled out
```
### EnqueNow and ExpectDequeNow

| method           |                            description                             |
| ---------------- | :----------------------------------------------------------------: |
| enqueueNow       |     Add (enqueue) one element to a `Decoupled` input interface     |
| expectDequeueNow | Removes (dequeues) one element from a `Decoupled` output interface |



```scala
test(new QueueModule(UInt(9.W), entries = 200)) { c =>
    //ioType是UInt(9.W)这表示每个数据元素都是 9 位宽的无符号整数，队列的深度是200

    c.in.initSource()
    c.in.setSourceClock(c.clock)
    c.out.initSink()
    c.out.setSinkClock(c.clock)
    //
    val testVector = Seq.tabulate(200){ i => i.U }
    //创建一个包含 200 个元素的序列 testVector，每个元素都是一个 UInt 类型的无符号整数，表示从 0 到 199。
    testVector.zip(testVector).foreach { case (in, out) =>
      c.in.enqueueNow(in)
      c.out.expectDequeueNow(out)
    }
}
```
1. `initSource(   )`:  
功能：初始化一个输入数据源，用于向模块的输入端口注入数据。  
使用场景：通常在测试中使用，用来模拟数据的产生或输入的起始状态。例如，可以将一个输入端口标记为数据源后，之后可以通过 enqueueNow 等方法向这个端口推送数据。  
2. `initSink( )`:  
功能：初始化一个输出接收端，用于从模块的输出端口接收数据。  
使用场景：同样是在测试中使用，用来准备接收从输出端口流出的数据。通过 expectDequeueNow 等方法，可以验证从输出端口取出的数据是否与预期一致。
3. `foreach`:  
   通常用于对集合（如数组、列表、序列等）进行遍历
4. 用`Enque`放进去一个0，此时的0在队列中了，再用`Deque`取出来,此时Expect两个  数的大小应该相等。  
   取出来之后并不意味着0没了,而是还在队列中,只不过下次`Deque`的时候由于FIFO,出来的是1.
## EnqueueSeq and DequeueSeq

| method           | description                                                                                                                            |
| :--------------- | :------------------------------------------------------------------------------------------------------------------------------------- |
| enqueueSeq       | Continues to add (enqueue) elements from the `Seq` to a `Decoupled` input interface, one at a time, until the sequence is exhausted    |
| expectDequeueSeq | Removes (dequeues) elements from a `Decoupled` output interface, one at a time, and compares each one to the next element of the `Seq` |

> [!NOTE] 
> The `enqueueSeq` must finish before the `expectDequeueSeq` can begin. This example would fail if the `testVector`'s size is made larger than the queue depth, because the queue would fill up and not be able to complete the `enqueueSeq`.
```scala
test(new QueueModule(UInt(9.W), entries = 200)) { c =>
    c.in.initSource()
    c.in.setSourceClock(c.clock)
    c.out.initSink()
    c.out.setSinkClock(c.clock)
    //同样的初始化
    val testVector = Seq.tabulate(100){ i => i.U }

    c.in.enqueueSeq(testVector)
    c.out.expectDequeueSeq(testVector)
}
```
> [!WARNING]
> c.in.enqueueSeq(testVector)  
> c.out.expectDequeueSeq(testVector)  
> 最后两句，把testVector中的所有元素按顺序入队，然后按顺序出队，希望获得testVector的结果

```scala
testVector = Seq(
  0.U, 1.U, 2.U, 3.U, 4.U, 5.U, 6.U, 7.U, 8.U, 9.U,
  10.U, 11.U, 12.U, 13.U, 14.U, 15.U, 16.U, 17.U, 18.U, 19.U,
  20.U, 21.U, 22.U, 23.U, 24.U, 25.U, 26.U, 27.U, 28.U, 29.U,
  30.U, 31.U, 32.U, 33.U, 34.U, 35.U, 36.U, 37.U, 38.U, 39.U,
  40.U, 41.U, 42.U, 43.U, 44.U, 45.U, 46.U, 47.U, 48.U, 49.U,
  50.U, 51.U, 52.U, 53.U, 54.U, 55.U, 56.U, 57.U, 58.U, 59.U,
  60.U, 61.U, 62.U, 63.U, 64.U, 65.U, 66.U, 67.U, 68.U, 69.U,
  70.U, 71.U, 72.U, 73.U, 74.U, 75.U, 76.U, 77.U, 78.U, 79.U,
  80.U, 81.U, 82.U, 83.U, 84.U, 85.U, 86.U, 87.U, 88.U, 89.U,
  90.U, 91.U, 92.U, 93.U, 94.U, 95.U, 96.U, 97.U, 98.U, 99.U
)
```
另一个例子
```scala
test(new Module {
    val io = IO(new Bundle {
      val in = Flipped(Decoupled(UInt(8.W)))
      val out = Decoupled(UInt(8.W))
    })
    val queue = Queue(io.in, 2)  
    // 2-element queue, 相当于是FIFO
    io.out <> queue
  }) { c =>
    //Stage1
    c.io.out.ready.poke(false.B)    
    //此时不读故设置输出的out.ready=0.由于此时队列是空的，所以valid=0；
    c.io.in.valid.poke(true.B)      
    // 设置输入端口有有效数据进入，即 in.valid=1，由于队列不满，所以 in.ready=1
    c.io.in.bits.poke(42.U)         
    //设置bit流数据，但是时钟还没来，这个数据还没读进去,所以下面的bits读出来是0
    println(s"Starting:") 
    println(s"\tio.in: ready=${c.io.in.ready.peek().litValue}")  //
    println(s"\tio.out: valid=${c.io.out.valid.peek().litValue}, bits=${c.io.out.bits.peek().litValue}")
    c.clock.step(1)

    //Stage2
    c.io.in.valid.poke(true.B)      //让另一个元素进队,设置进入数据有效
    c.io.in.bits.poke(43.U)         //把数据流设置为43
    
    println(s"After first enqueue:")
    println(s"\tio.in: ready=${c.io.in.ready.peek().litValue}")     //由于此时的队伍非满,所以是1
    println(s"\tio.out: valid=${c.io.out.valid.peek().litValue}, bits=${c.io.out.bits.peek().litValue}") 
    //由于此时的队伍非空,所以是1,并且在时钟的上升沿已经读入了42,bits=42
    c.clock.step(1)

    //Stage3
    c.io.in.valid.poke(true.B)  //再次尝试读入一个元素
    c.io.in.bits.poke(44.U)     //把数据流设置为44
    c.io.out.ready.poke(true.B) //开启读
    
    println(s"On first read:") 
    println(s"\tio.in: ready=${c.io.in.ready.peek().litValue}") //已经此时的队列已经满了所以为0
    println(s"\tio.out: valid=${c.io.out.valid.peek().litValue}, bits=${c.io.out.bits.peek().litValue}")
    //因为此时的队列仍旧是非空的,所以为valid=1，由于第一个读出去的应该是42，结果是42
    c.clock.step(1)

    //Stage4
    c.io.in.valid.poke(false.B)  // 此时不写入了
    c.io.out.ready.poke(true.B)  // 此时往外读
    
    
    println(s"On second read:")
    println(s"\tio.in: ready=${c.io.in.ready.peek().litValue}") //由于此时的queue不是满的,应该为1
    println(s"\tio.out: valid=${c.io.out.valid.peek().litValue}, bits=${c.io.out.bits.peek().litValue}") //由于此时的队列是有元素的,所以valid=1,并且可以知道出去的应该是43
    c.clock.step(1)

    //Stage5
    println(s"On third read:")
    println(s"\tio.in: ready=${c.io.in.ready.peek().litValue}") //由于此时的队列空ready=1
    println(s"\tio.out: valid=${c.io.out.valid.peek().litValue}, bits=${c.io.out.bits.peek().litValue}") //由于此时的队列空valid=0,读出的数据应该就是最开始的数据
    c.clock.step(1)
}
```

```scala
Starting:
	io.in: ready=1
	io.out: valid=0, bits=0
After first enqueue:
	io.in: ready=1
	io.out: valid=1, bits=42
On first read:
	io.in: ready=0
	io.out: valid=1, bits=42
On second read:
	io.in: ready=1
	io.out: valid=1, bits=43
On third read:
	io.in: ready=1
	io.out: valid=0, bits=42
```

需要解释一下就是queue是寄存器,所以需要clock的边沿触发,但是arbiter不是寄存器,所以直接赋值就可以了