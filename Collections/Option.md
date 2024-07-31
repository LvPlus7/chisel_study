## Option

```scala
val map = Map("a" -> 1)
val a = map("a")
println(a)
val b = map("b")
println(b)
```

此时会出现警告是因为，在映射关系中查找不到b，换成map.get()返回的是一个option，如果没有查找到，就返回none，而不是error

```scala
val map = Map("a" -> 1)
val a = map.get("a")
println(a)
val b = map.get("b")
println(b)
```
