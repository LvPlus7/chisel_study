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
### float numbers

```scala
var a = 3.2        //default double
var b = 1.3E4      //means 1.3 multiplies 10000
var c = 9.9f       //means float type
```

### string

```scala
val name ="ABC"
println(s"$name DEFG")
//插值字符串，在字符串前面加个s。s“…${表达式}…”其中的表达式可以直接在println中求值
```

### Dontcare

这是因为scala是一门静态语言需要知道这个变量的类型，即使有些变量对于结果的输出并不重要，这个时候直接用dontcare代替即可
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

//为了让scala中的==  !=仍然可用,scala中的==在比较的时候会发生类型的转换
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

## 条件语句
### if /else if /else   //用在纯软环境下

```scala
if
else
else if
//above can be used as c++
//special: use it as a value
//example below
```

```scala
val likelyCharactersSet = if (alphabet.length == 26)
    "english"
else 
    "not english"
println(likelyCharactersSet)
//result: English
```

### when / elsewhen/ otherwise   //用在Chisel的硬件环境下

```scala
when(someBooleanCondition) {
  // things to do when true
}.elsewhen(someOtherBooleanCondition) {
  // things to do on this condition
}.otherwise {
  // things to do if none of th boolean conditions are true
}
```

> [!NOTE]
> 
> when doesn't return a value .   if can return a value.

## Lists

```scala
val x = 7
val y = 14
val list1 = List(1, 2, 3)
val list2 = x :: y :: y :: Nil       
// An alternate notation for assembling a list,组装起来
val list3 = list1 ++ list2           
// Appends the second list to the first list,两个数组组装起来，注意此处用的是++运算符，而不是+
val m = list2.length
val s = list2.size

val headOfList = list1.head          // Gets the first element of the list
val restOfList = list1.tail          // Get a new list with first element removed

val third = list1(2)                 // Gets the third element of a list (0-indexed)
```

```scala
x: Int = 7
y: Int = 14
list1: List[Int] = List(1, 2, 3)
list2: List[Int] = List(7, 14, 14)
list3: List[Int] = List(1, 2, 3, 7, 14, 14)
m: Int = 3
s: Int = 3
headOfList: Int = 1
restOfList: List[Int] = List(2, 3)
third: Int = 3
```

## For cycle 循环
比如此处写了tb，包含一个循环语句
```scala
test(new MAC) { c =>
  val cycles = 100
  import scala.util.Random
  for (i <- 0 until cycles) {
    //
    val in_a = Random.nextInt(16)
    val in_b = Random.nextInt(16)
    val in_c = Random.nextInt(16)
    c.io.in_a.poke(in_a.U)
    c.io.in_b.poke(in_b.U)
    c.io.in_c.poke(in_c.U)
    c.io.out.expect((in_a * in_b + in_c).U)
  }
}
```

> [!CAUTION]
>
> **i <- 0 until cycles**  ??????

## Arrow，箭头的使用

```scala
//->在创建Map的时候使用，用来表示一种映射
def states = Map("idle" -> 0, "coding" -> 1, "writing" -> 2, "grad" -> 3)
//<-在for循环中常使用，表示遍历
for(select <- 0 to 2) {
   c.io.select.poke(select.U)
   c.io.x.poke(x.S)
   c.io.fOfX.expect(poly(select, x).S)
}
//=>常用来表示匿名函数和match
def choice(c:String)={
    c match{
        case "a" => 90
        case "b" => 75
        case "c" => 60
        case _ => 59
    }
}
```