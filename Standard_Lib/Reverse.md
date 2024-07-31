## Reverse

可以用来返回一个相反的数 0-1 1-0

```scala
test(new Module {
    // Example circuit using Reverse
    val io = IO(new Bundle {
      val in = Input(UInt(8.W))
      val out = Output(UInt(8.W))
    })
    io.out := Reverse(io.in)
  }) { c =>
    // Integer.parseInt is used create an Integer from a binary specification
    c.io.in.poke(Integer.parseInt("01010101", 2).U)
    println(s"in=0b${c.io.in.peek().litValue.toInt.toBinaryString}, out=0b${c.io.out.peek().litValue.toInt.toBinaryString}")

    c.io.in.poke(Integer.parseInt("00001111", 2).U)
    println(s"in=0b${c.io.in.peek().litValue.toInt.toBinaryString}, out=0b${c.io.out.peek().litValue.toInt.toBinaryString}")

    c.io.in.poke(Integer.parseInt("11110000", 2).U)
    println(s"in=0b${c.io.in.peek().litValue.toInt.toBinaryString}, out=0b${c.io.out.peek().litValue.toInt.toBinaryString}")

    c.io.in.poke(Integer.parseInt("11001010", 2).U)
    println(s"in=0b${c.io.in.peek().litValue.toInt.toBinaryString}, out=0b${c.io.out.peek().litValue.toInt.toBinaryString}")
}
```

```
Elaborating design...
Done elaborating.
in=0b1010101, out=0b10101010
in=0b1111, out=0b11110000
in=0b11110000, out=0b1111
in=0b11001010, out=0b1010011
test Helper_Anon Success: 0 tests passed in 2 cycles in 0.010021 seconds 199.59 Hz
```