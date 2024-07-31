## Class
```scala
class Students {
    var name = "None"
    def register(n: String) = name = n
}
```
以上的代码实现了在`Students`类里面定义一个`register`方法用来注册学生的名字
```scala
class Students {
    private var name = "None"
    def register(n: String) = name = n
    def display() = println(name)
}
//实例
scala> val stu = new Students
stu: Students = Students@75063bd0
//借用register方法去定义名字
scala> stu.register("Bob")
//这里就是不能直接访问私有成员
scala> stu.name
<console>:13: error: variable name in class Students cannot be accessed in Students
       stu.name
           ^
//借助方法去访问私有成员
scala> stu.display
Bob
```
### Abstract

```scala
abstract class MyAbstractClass {
  def myFunction(i: Int): Int
  val myValue: String
}
class ConcreteClass extends MyAbstractClass {
  def myFunction(i: Int): Int = i + 1
  val myValue = "Hello World!"
}
// Uncomment below to test!
//val abstractClass = new MyAbstractClass() // Illegal! Cannot instantiate an abstract class
val concreteClass = new ConcreteClass()      // Legal!
```

不能实例化抽象类，只能实例化继承于抽象类的类，在这里`MyAbstractClass`就是抽象类，`ConcreteClass`就是延伸于抽象类的类

### Trait
trait与抽象类非常相似，因为它们可以定义未实现的值。然而，它们有两个不同之处:

一个类可以从多个trait继承

一个trait不能有构造函数参数
```scala
trait HasFunction {
  def myFunction(i: Int): Int
}
//其中的一个trait不能有构造函数
trait HasValue {
  val myValue: String
  val myOtherValue = 100
}
class MyClass extends HasFunction with HasValue {
  override def myFunction(i: Int): Int = i + 1
  val myValue = "Hello World!"
}
// Uncomment below to test!
// val myTraitFunction = new HasFunction() // Illegal! Cannot instantiate a trait
// val myTraitValue = new HasValue()       // Illegal! Cannot instantiate a trait
val myClass = new MyClass()                // Legal!
```

*class MyClass extends trait1 with trait2 with...*  
用来定义继承于特质的类  
不能直接实例化特质

### 单例类Object
类似于没有类，而直接出来一个对象
```scala
object MyObject {
  def hi: String = "Hello World!"
  def apply(msg: String) = msg
}
println(MyObject.hi)
println(MyObject("This message is important!")) 
// equivalent to MyObject.apply(msg)  使用了apply方法
```


### 伴生对象

当类和对象具有相同的名称并且定义在同一个文件中时，该对象称为伴生对象。当在类/对象名称前使用new时，它将实例化类。如果你不使用new，它将引用对象:
```scala
//定义的单例对象
object Lion {
    def roar(): Unit = println("I'M AN OBJECT!")
}
//使用相同名字定义类
class Lion {
    def roar(): Unit = println("I'M A CLASS!")
}
new Lion().roar()
Lion.roar()
```
```
*** 结果 ***
I'M A CLASS!
I'M AN OBJECT!
```
```scala
object Animal {
    val defaultName = "Bigfoot"
    //定义隐私变量去跟踪数量
    private var numberOfAnimals = 0
    def apply(name: String): Animal = {
        numberOfAnimals += 1
        new Animal(name, numberOfAnimals)
    }
    def apply(): Animal = apply(defaultName)
}


class Animal(name: String, order: Int) {
  def info: String = s"Hi my name is $name, and I'm $order in line!"
}

val bunny = Animal.apply("Hopper") // Calls the Animal factory method
println(bunny.info)
val cat = Animal("Whiskers")       // Calls the Animal factory method
println(cat.info)
val yeti = Animal()                // Calls the Animal factory method
println(yeti.info)

```