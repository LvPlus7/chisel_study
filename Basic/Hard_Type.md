## Hard
### Channel
#### 整体连接<>
```scala
class MyIO extends Bundle {
   val in = Input(Vec(5, UInt(32.W)))
   val out = Output(UInt(32.W))
}

......
   val io = IO(new Bundle {
       val x = new MyIO       
       val y = Flipped(new MyIO)
   })

   io.x <> io.y  // 相当于 io.y.in := io.x.in; io.x.out := io.y.out 
   ```
#### IO
IO用于声明一个模块的接口
```scala
val io = IO(new Bundle {
    val in_a = Input(UInt(8.W))
    val out_b = Output(UInt(8.W))
})
//输入输出接口也是一组信号的捆绑，所以里面用到了bundle
```

### Module

在Chisel里面是用一个自定义的类来定义模块的，这个类有以下三个特点：①继承自Module类。②有一个抽象字段“io”需要实现，该字段必须引用前面所说的端口对象。③在类的主构造器里进行内部电路连线。

```scala
class Mux2 extends Module {             //继承
  val io = IO(new Bundle{               //IO的实现
    val sel = Input(UInt(1.W))
    val in0 = Input(UInt(1.W))
    val in1 = Input(UInt(1.W))
    val out = Output(UInt(1.W))
  })
 
  io.out := (io.sel & io.in1) | (~io.sel & io.in0)}
  //电路的抽象连接
```
例化
```scala
class Mux4 extends Module {
  val io = IO(new Bundle {
    val in0 = Input(UInt(1.W))
    val in1 = Input(UInt(1.W))
    val in2 = Input(UInt(1.W))
    val in3 = Input(UInt(1.W))
    val sel = Input(UInt(2.W))
    val out = Output(UInt(1.W))
  })
  val m0 = Module(new Mux2)
  m0.io.sel := io.sel(0)
  m0.io.in0 := io.in0
  m0.io.in1 := io.in1
  val m1 = Module(new Mux2)
  m1.io.sel := io.sel(0)
  m1.io.in0 := io.in2
  m1.io.in1 := io.in3
  val m2 = Module(new Mux2)
  m2.io.sel := io.sel(1)
  m2.io.in0 := m0.io.out
  m2.io.in1 := m1.io.out
  io.out := m2.io.out
}
```
同时例化多个模块
```scala
val m = VecInit(Seq.fill(3)(Module(new Mux2).io)) 
  m(0).sel := io.sel(0)  // 模块的端口通过下标索引，并且路径里没有“io”
  m(0).in0 := io.in0
  m(0).in1 := io.in1
  m(1).sel := io.sel(0)
  m(1).in0 := io.in2
  m(1).in1 := io.in3
  m(2).sel := io.sel(1)
  m(2).in0 := m(0).out
  m(2).in1 := m(1).out
  io.out := m(2).out
```

### Wire
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

### Reg
```scala
//初始化   定义了一个宽度为8位的寄存器
val reg = RegInit(0.U(8.W))
reg := d
val q = reg
//以上的三行代码实现一个8位的寄存器
//命名的时候通常在后面标注reg用来区分（命名规则）
```
*take the input as the clock's rising edge*

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

#### RegNext
值在下个时钟周期更新
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

#### RegInit

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
#### 寄存器组
```scala
  val reg0 = RegNext(VecInit(io.a, io.a))
  val reg1 = RegNext(VecInit(io.a, io.a), VecInit(0.U, 0.U))
  val reg2 = RegInit(VecInit(0.U(8.W), 0.U(8.W)))
  val reg3 = Reg(Vec(2, UInt(8.W)))
  val reg4 = RegEnable(VecInit(io.a + 1.U, io.a + 1.U), VecInit(0.U(8.W), 0.U(8.W)), io.en)
  val reg5 = RegEnable(VecInit(io.a - 1.U, io.a - 1.U), io.en)
  val reg6 = ShiftRegister(VecInit(io.a, io.a), 3, VecInit(0.U(8.W), 0.U(8.W)), io.en)

```

- 4中 RegEnable: 这个函数创建一个有条件的寄存器，根据 io.en 的值来决定是否更新寄存器的值。
- 第一个参数是一个向量，包含两个元素，每个元素是 io.a + 1.U。
- 第二个参数是一个向量，用于指定在 io.en 为假时应该写入寄存器的值。
- io.en 是一个输入输出接口中的信号，用来控制寄存器更新的使能。  

[Enable](https://zhuanlan.zhihu.com/p/594022828)