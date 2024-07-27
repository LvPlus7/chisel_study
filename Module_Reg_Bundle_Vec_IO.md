# 模块化
## 寄存器
```scala
//初始化   定义了一个宽度为8位的寄存器
val reg = RegInit(0.U(8.W))
reg := d
val q = reg
//以上的三行代码实现一个8位的寄存器
//命名的时候通常在后面标注reg用来区分（命名规则）
```
*take the input as the clock's rising edge*

### Reg

```scala
//这里是源文件
class RegisterModule extends Module {
  val io = IO(new Bundle {
    val in  = Input(UInt(12.W))
    val out = Output(UInt(12.W))
  })
  
  val register = Reg(UInt(12.W))
  register := io.in + 1.U
  io.out := register
}
//这里是tb文件
test(new RegisterModule) { c =>
  for (i <- 0 until 100) {
    c.io.in.poke(i.U)
    c.clock.step(1)
    c.io.out.expect((i + 1).U)
  }
}
println("SUCCESS!!")
```

> [!NOTE]
> 用 Reg( ) 去创建寄存器，在这里创建的12位的  
> step(n)表示时钟跳动n次，在chisel的程序里面不需要明显的写出来clock  
> Reg(UInt(12.W))是合法的，Reg(2.U)是非法的，因为在此处不能初始化值

### RegNext

```scala
//以上代码的缩短版
class RegNextModule extends Module {
  val io = IO(new Bundle {
    val in  = Input(UInt(12.W))
    val out = Output(UInt(12.W))
  })
  
  // register bitwidth is inferred from io.out,由输出直接推断，不用明显的定义Reg
  io.out := RegNext(io.in + 1.U)
}
```

### RegInit

创建一个有给定值的寄存器

```scala
val register = RegInit(0.U(12.W))  //创建一个宽为12位的全是0的寄存器
```

```scala
class MyShiftRegister(val init: Int = 1) extends Module {
  val io = IO(new Bundle {
    val in  = Input(Bool())
    val out = Output(UInt(4.W))                         //定义4位的
  })													
  val state = RegInit(UInt(4.W), init.U)                //把设定的初始值放在类定义的参数中
  val nextState = (state << 1) | io.in					//直接用运算符左移一位,或上in,就是新的状态了
  state := nextState
  io.out := state 
}

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

## Vec
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
## Wire
UInt、SInt和Bits都是Chisel类型，它们本身是不直接表示硬件的，只有把它们封装成为Wire、Reg或IO才会生成电路     
  
在chisel中只使用val来描述电路
```scala
val number = Wire(UInt())
val reg = Reg(SInt())
//在创建硬件对象的时候使用=，给硬件对象重新赋值的时候使用:=
number := 10.U
reg := value - 3.U
```

```scala
class LastConnect extends Module {
  val io = IO(new Bundle {
    val in = Input(UInt(4.W))
    val out = Output(UInt(4.W))
  })
  io.out := 1.U
  io.out := 2.U
  io.out := 3.U
  io.out := 4.U       //the last connect
}
//赋值的结果取决于最后一次赋值的值的大小
```

> [!NOTE]
> 取决于最后一次的赋值


## IO
IO用于声明一个模块的接口
```scala
val io = IO(new Bundle {
    val in_a = Input(UInt(8.W))
    val out_b = Output(UInt(8.W))
})
//输入输出接口也是一组信号的捆绑，所以里面用到了bundle
```

## Module

```scala
// Chisel Code: Declare a new module definition
class Passthrough extends Module {
  val io = IO(new Bundle {
    val in = Input(UInt(4.W))
    val out = Output(UInt(4.W))
  })
  io.out := io.in
}
```
  
  
```scala
module       
//是内建类，需要使用extend（对于所有的硬件模块都需要extend）
val io = IO(new Bundle { ？？？})  
```