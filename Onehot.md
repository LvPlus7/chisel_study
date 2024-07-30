## One-Hot Coding Utilities

`Onehot`:使用N位状态寄存器对于N个状态进行编码
主要实现二进制代码与Onehot编码的互相转换

```scala
test(new Module {
    
    val io = IO(new Bundle {
      val in = Input(UInt(4.W))
      val out = Output(UInt(16.W))\
      //定义了4位输入,16位输出
    })
    io.out := UIntToOH(io.in)
  }) { c =>
    c.io.in.poke(0.U)
    println(s"in=${c.io.in.peek().litValue}, out=0b${c.io.out.peek().litValue.toInt.toBinaryString}")

    c.io.in.poke(8.U)
    println(s"in=${c.io.in.peek().litValue}, out=0b${c.io.out.peek().litValue.toInt.toBinaryString}")

    c.io.in.poke(15.U)
    println(s"in=${c.io.in.peek().litValue}, out=0b${c.io.out.peek().litValue.toInt.toBinaryString}")
}
```
```scala
Elaborating design...
Done elaborating.
in=0, out=0b1
in=1, out=0b10
in=8, out=0b100000000
in=15, out=0b1000000000000000
```
toInt转换为整数，toBinaryString转换为二进制字符串，可以用parseInt将二进制字符串反转换为二进制。

另一段代码展示了从OH编码转换为二进制的过程

```scala
test(new Module {
    // Example circuit using OHToUInt
    val io = IO(new Bundle {
      val in = Input(UInt(16.W))
      val out = Output(UInt(4.W))
    })
    io.out := OHToUInt(io.in)
}) { c =>
    c.io.in.poke(Integer.parseInt("0000 0000 0000 0001".replace(" ", ""), 2).U)
    println(s"in=0b${c.io.in.peek().litValue.toInt.toBinaryString}, out=${c.io.out.peek().litValue}")
    //parseInt用来将字符串解析为整数的Int形式，
    //后面的2表示是2进制类型的字符串

    c.io.in.poke(Integer.parseInt("0000 0000 1000 0000".replace(" ", ""), 2).U)
    println(s"in=0b${c.io.in.peek().litValue.toInt.toBinaryString}, out=${c.io.out.peek().litValue}")

    c.io.in.poke(Integer.parseInt("1000 0000 0000 0001".replace(" ", ""), 2).U)
    println(s"in=0b${c.io.in.peek().litValue.toInt.toBinaryString}, out=${c.io.out.peek().litValue}")

    // Some invalid inputs:
    // None high
    c.io.in.poke(Integer.parseInt("0000 0000 0000 0000".replace(" ", ""), 2).U)
    println(s"in=0b${c.io.in.peek().litValue.toInt.toBinaryString}, out=${c.io.out.peek().litValue}")

    // Multiple high
    c.io.in.poke(Integer.parseInt("0001 0100 0010 0000".replace(" ", ""), 2).U)
    println(s"in=0b${c.io.in.peek().litValue.toInt.toBinaryString}, out=${c.io.out.peek().litValue}")
}
```
```scala
Elaborating design...
Done elaborating.
in=0b1, out=0
in=0b10000000, out=7
in=0b1000000000000001, out=15
in=0b0, out=0
in=0b1010000100000, out=15
```
这段代码中第三个例子，15有0有，此时OHtoUInt自动忽略了低位，只取高位。此时相当于是无效的输入。