## Scala Literals 
```scala
//Int Float Double Char 
2 + 3
5.0 / 2
"hello"
//很少使用分号来表示句子的结束
```
### Float

```scala
var a = 3.2        //default double
var b = 1.3E4      //means 1.3 multiplies 10000
var c = 9.9f       //means float type
```

### String

```scala
val name ="ABC"
println(s"$name DEFG")
//插值字符串，在字符串前面加个s。s“…${表达式}…”其中的表达式可以直接在println中求值
```

### Dontcare

这是因为scala是一门静态语言需要知道这个变量的类型，即使有些变量对于结果的输出并不重要，这个时候直接用dontcare代替即可



## var可变 val不可变 （更推荐使用后者）
```scala
var mutX = 0
val constX = 42
```



## Chisel 类型
Bool是chisel的硬件类型，Boolean是scala的类型，两者不要搞混

UInt 无符号数

SInt 有符号数
```scala
0.B  //chisel false
true    //scala true
true.B  //chisel true
val myBool: Bool = true.B   //announce a true 

6   //scala6
6.U     //chisel uint 6
6.U(8.W)
val myUInt: UInt = 4.U
val myUInt8 = 4.U(8.W)
```
使用数字+ W 方式来表示宽度
```scala
3.U(4.W)
```



## 使用其他进制来表示
```scala
"hff".U
"o377".U
"b1111_1111".U //此处chisel会自动推算宽度
```

## 类型转换

UInt、SInt和Bool三个类包含四个方法：asUInt、asSInt、toBool和toBools

toUInt：将有符号数转换无符号数

toSInt：3bit的UInt值“b111”，其字面量是“7”，转换成SInt后字面量就变成了“-1”

toBool：会把1bit的“1”转换成Bool类型的true，“0”转换成false

toBools：如果位宽超过1bit，则用toBools转换成Bool类型的序列Seq[Bool]。
```scala
op1 := "b1110".U   //此时的值为e
op1.asSInt         //转换之后的值为-2
```

## Vec  向量
一组相同类型的信号的集合
```scala
val v = Wire(Vec(3, UInt(4.W)))
//定义vec的时候用到，向量中元素的个数，元素的类型以及宽度，使用的时候同样需要借助wire
v(0) := 1.U
v(1) := 3.U
v(2) := 5.U

val idx = 1.U(2.W)
val a = v(idx)
//从而wire充当了一个多路选择器的作用，通过索引去访问.......

val registerFile = Reg(Vec(32, UInt(32.W)))
//定义了32个32位寄存器
```
### MixedVec 混合向量

允许用户在同一向量中混合使用不同类型的元素

```SCALA
class MixedDataBundle extends Bundle {
  val intData = UInt(32.W)
  val boolData = Bool()
}

class ExampleModule extends Module {
  val io = IO(new Bundle {
    val mixedVec = MixedVec(Seq(UInt(8.W), Bool(), new MixedDataBundle()))
  })

  // 初始化 MixedVec
  io.mixedVec(0) := 255.U
  io.mixedVec(1) := true.B
  io.mixedVec(2).intData := 42.U
  io.mixedVec(2).boolData := false.B
}
```
---
```scala
val mixVec = Wire(MixedVec((1 to 10) map { i => UInt(i.W) }))
```

## Bundle 捆
Bundle用来将一堆信号形成一个捆绑包
```scala
class Channel() extends Bundle {
    val data = UInt(32.W)
    val valid = Bool()
}
//通过定义一个类来定义一个bundle，类拓展自bundle类，channel这个bundle就把data和valid的两个信号捆绑在了一起，如果要使用这个bundle的话，我们可以new一个Channel然后把它封装到一个Wire里面
val ch = Wire(new Channel())
ch.data := 123.U
ch.valid := true.B

val b = ch.valid

//此外bundle也可以作为一个整体来引用
val channel = ch
```

Bundle可以和UInt进行相互转换。  
Bundle类有一个方法asUInt，可以把所含的字段拼接成一个UInt数据，并且前面的字段在高位。例如：
```SCALA
class MyBundle extends Bundle {
   val foo = UInt(4.W)  // 高位
   val bar = UInt(4.W)  // 低位
}
val bundle = Wire(new MyBundle)
bundle.foo := 0xc.U
bundle.bar := 0x3.U
val uint = bundle.asUInt  // 12*16 + 3 = 195
```

## Bundle Vec的结合使用
```scala
//Vec可以包含Bundle
val bundle_in_vec = Wire(Vec(8,new Channel()))
//Bundle里面可以有Vec
class vec_in_bundle extends Bundle{
    val vector = Vec(8,UInt(8.W))
}

val initVal = Wire(new Channel())
initVal.data := 0.U
initVal.valid := false.B
//先通过wire类型定义一个变量，其实为bundle，然后去修改bundle内部的信号的值。
//最后把这变量的值送给RegInit
val channelReg = RegInit(initVal)


val assignWord = Wire(UInt(16.W))
class Split extends Bundle {
    val high = UInt(8.W)
    val low = UInt(8.W)
}
val split = Wire(new Split())
split.low := lowByte
split.high := highByte
assignWord := split.asUInt
//使用bundle进行部分赋值
```