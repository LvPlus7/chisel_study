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