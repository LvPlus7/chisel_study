
## Fork and Join

| method | description                                                                                                                                                   |
| :----- | :------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| fork   | launches a concurrent code block, additional forks can be run concurrently to this one via the .fork appended to end of the code block of the preceeding fork |
| join   | re-unites multiple related forks back into the calling thread                                                                                                 |

fork并行执行多个分支，join用来合并多个分支的结果

```scala
import chisel3._

class MyModule extends Module {
  val io = IO(new Bundle {
    val inputA = Input(UInt(8.W))
    val inputB = Input(UInt(8.W))
    val output = Output(UInt(8.W))
  })

  // 在这里使用 fork-join 结构
  fork {
    // 第一个分支
    val resultA = io.inputA + 1.U
    io.output := resultA
  } join {
    // 第二个分支
    val resultB = io.inputB * 2.U
    io.output := resultB
  }
}
```

需要采用其他方法确定在端口赋值的时候不会发生冲突
