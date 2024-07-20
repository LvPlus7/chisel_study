## scala literals scala的字面量
```scala
//Int Float Double Char 
2 + 3
5.0 / 2
"hello"
//很少使用分号来表示句子的结束
```
## var可变 val不可变 （更推荐使用后者）
```scala
var mutX = 0
val constX = 42
```
## chisel 类型
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
使用数字+ W 方式来表示宽
```scala
3.U(4.W)
```
## 使用其他进制来表示
```scala
"hff".U
"o377".U
"b1111_1111".U //此处chisel会自动推算宽度
```
## 一大堆的运算符
### 位运算符
```scala
val and = a & b		// 按位与
val or = a | b		// 按位或
val xor = a ^ b		// 按位异或
val not = ~a		// 按位取反

val shiftleft = a << b
val shiftright = a >> b
//移位运算符不能用于bool的chisel类型
```
### 算数运算符
```scala
val add = a + b		// 加法
val sub = a - b		// 减法
val neg = -a		// 取相反数
val mul = a * b		// 乘法
val div = a / b		// 除法
val mod = a % b		// 取余
```

### 逻辑运算符
针对bool类型计算，有&& || ！
### 比较运算符
操作数为UInt、SInt，返回bool
```scala
> < <= >=
=== //等于
=/= //不等于

//为了让scala中的==  !=仍然可用
```
### 位字段操作符
```scala
val xLSB = x(0)		// 提取x的最低位
val xTopNibble = x(15, 12)	// 假设x是16位的，提取x的高4位
val usDebt = Fill(3, "hA".U)	// "hAAA".U 复制了3次
val float = Cat(sign, exponent, mantissa)		// 拼接三个向量
```
### MUX
val y = Mux(sel, a, b)

sel为布尔类型的值，为0的时候输出a，为1的时候输出b
## 寄存器
```scala
//初始化   定义了一个宽度为8位的寄存器
val reg = RegInit(0.U(8.W))
reg := d
val q = reg
//以上的三行代码实现一个8位的寄存器
//命名的时候通常在后面标注reg用来区分
```
## Bundle Vec
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
## Wire Reg IO 
UInt、SInt和Bits都是Chisel类型，它们本身是不直接表示硬件的，只有把它们封装成为Wire、Reg或IO才会生成电路
### Wire
在chisel中只使用val来描述电路
```scala
val number = Wire(UInt())
val reg = Reg(SInt())
//在创建硬件对象的时候使用=，给硬件对象重新赋值的时候使用:=
number := 10.U
reg := value - 3.U
```
### IO
IO用于声明一个模块的接口
```scala
val io = IO(new Bundle {
    val in_a = Input(UInt(8.W))
    val out_b = Output(UInt(8.W))
})
//输入输出接口也是一组信号的捆绑，所以里面用到了bundle
```
