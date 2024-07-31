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
-----