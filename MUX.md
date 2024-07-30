## Muxes

分为PriorityMux以及Mux1H

### Priority Mux

```scala
test(new Module {
    val io = IO(new Bundle {
      //两个bool的向量，两个8为宽度的数据作为数据位，1个8为宽度的数据作为输出  
      val in_sels = Input(Vec(2, Bool()))
      val in_bits = Input(Vec(2, UInt(8.W)))
      val out = Output(UInt(8.W))
    })

    io.out := PriorityMux(io.in_sels, io.in_bits)

  }) { c =>

    c.io.in_bits(0).poke(10.U)//端口0设置为10
    c.io.in_bits(1).poke(20.U)//端口1设置为20

    // Select higher index only
    c.io.in_sels(0).poke(false.B)
    c.io.in_sels(1).poke(true.B)
    println(s"in_sels=${c.io.in_sels(0).peek().litValue}, out=${c.io.out.peek().litValue}")

    // Select both - arbitration needed 
    c.io.in_sels(0).poke(true.B)
    c.io.in_sels(1).poke(true.B)
    println(s"in_sels=${c.io.in_sels(0).peek().litValue}, out=${c.io.out.peek().litValue}")

    // Select lower index only
    c.io.in_sels(0).poke(true.B)
    c.io.in_sels(1).poke(false.B)
    println(s"in_sels=${c.io.in_sels(0).peek().litValue}, out=${c.io.out.peek().litValue}")
}
```
```scala
in_sels=0, out=20
in_sels=1, out=10
in_sels=1, out=10
```

这个例子就是为了说明PriorityMux在多端口有效的时候，优先取低顺位的数据

###  Mux1H

Mux1H就是一个有效的多路选择器，只有一个选择信号为高时候，输出信号有效，其他情况下，输出信号都是无效的。  
Mux1H的名称中的“1”表示选择信号的位宽度为1，而“H”表示“hot”，即在输入端口中只有一个信号是高电平（1）

```scala
test(new Module {

    val io = IO(new Bundle {
      val in_sels = Input(Vec(2, Bool()))
      val in_bits = Input(Vec(2, UInt(8.W)))
      val out = Output(UInt(8.W))
    })

    io.out := Mux1H(io.in_sels, io.in_bits)
  }) { c =>

    c.io.in_bits(0).poke(10.U)
    c.io.in_bits(1).poke(20.U)

    // Select index 1
    c.io.in_sels(0).poke(false.B)
    c.io.in_sels(1).poke(true.B)
    println(s"in_sels=${c.io.in_sels(0).peek().litValue}, out=${c.io.out.peek().litValue}")

    // Select index 0
    c.io.in_sels(0).poke(true.B)
    c.io.in_sels(1).poke(false.B)
    println(s"in_sels=${c.io.in_sels(0).peek().litValue}, out=${c.io.out.peek().litValue}")

    // Select none (invalid)
    c.io.in_sels(0).poke(false.B)
    c.io.in_sels(1).poke(false.B)
    println(s"in_sels=${c.io.in_sels(0).peek().litValue}, out=${c.io.out.peek().litValue}")

    // Select both (invalid)
    c.io.in_sels(0).poke(true.B)
    c.io.in_sels(1).poke(true.B)
    println(s"in_sels=${c.io.in_sels(0).peek().litValue}, out=${c.io.out.peek().litValue}")
}
```


```scala
in_sels=0, out=20
in_sels=1, out=10
in_sels=0, out=0
in_sels=1, out=30
```
```scala
// Declare a vector of 4 bits width
val myVec = VecInit(Seq(0.U, 3.U, 5.U, 7.U))

// Create a 2-bit select signal
val select = Wire(UInt(2.W))
select := 2.U

// Use Mux1H to select the third input of myVec
val result = Mux1H(select, myVec)

// Print the selected result
printf("Result: %d\n", result)

//*** 结果 ***
Result:5
```
------

### MuxLookup

MuxLookup是一种多路复用器（Mux）电路。它根据选择信号的值从一组`键-值对`中选择一个输出，并且当输入数据量大时它比MuxCase更易于使用。

代码先欠着