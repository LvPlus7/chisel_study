
## Scala interactive Build Tool (sbt)
```verilog
//chisel项目源代码的组织结构
//project
//  src
//  |    main
//  |         scala
//  |                package
//  |                        sub_package
//  test
//  |    scala
//  |         package
//  target
//  generated

//src文件夹包含全部的源代码，
//main  test分别包含硬件源码和测试器代码
//generated用来保存生成的verilog文件
```
```scala
//在包中定义一个类
package mypack

import chisel3._

class Abc extends Module {
    val io = IO(new Bundle{})
}
//使用module abc就需要导入这个mypack包
import mypack._

class AbcUser extends Module {
    val io = IO(new Bundle{})
    val abc = new Abc()
}

class AbcUser2 extends Module {
    val io = IO(new Bundle{})
    val abc = new mypack.Abc()
}

import mypack.Abc

class AbcUser3 extends Module {
    val io = IO(new Bundle{})
    val abc = new Abc()
}

//用sbt编译
sbt run
//选择一个执行
sbt "runMain xxxxx.MyObject"
//包含了main函数,放在了源代码树的test文件夹
sbt "test:runMain xxxx.MyMainTest"
```
## getVerilog

object Hello：这定义了一个名为"Hello"的对象。  
在Scala中，object用于定义单例对象，这意味着你的程序中只会有一个该对象的实例。

extends App：这表示Hello对象扩展了App特质。  
在Scala中，App特质允许你定义程序的入口点，而无需显式编写main方法。

emitVerilog(new Hello())：这是调用了一个名为emitVerilog的方法，并将一个Hello类的新实例作为参数传递给它。  
这表明你正在使用某个包含名为emitVerilog的方法的库或框架，用于从使用Chisel编写的硬件描述生成Verilog代码。

```scala
// Hello World
import chisel3._

class Hello extends Module {
    val io = IO(new Bundle {
    val led = Output(UInt(1.W))   //定义了1位的输出位led
    })
    val CNT_MAX = (50000000 / 2 - 1).U   //定义计数的最大值
    val cntReg = RegInit(0.U(32.W))     //计数器
    val blkReg = RegInit(0.U(1.W))       //定义的是灯状态.亮灭亮灭

    cntReg := cntReg + 1.U
    when(cntReg === CNT_MAX) {
    cntReg := 0.U
    blkReg := ~blkReg
    }
    io.led := blkReg
}
//直接terminal得到verilog的代码
println(getVerilogString(new Hello()))
//把生成的verilog代码放在generated文件夹下面
object HelloOption extends App {
    emitVerilog(new Hello(), Array("--target-dir", "generated"))
}
//generated文件夹下除了*.v的Verilog文件，还有*.fir文件和*.anno.json文件
```
## Chisel Test

```scala
c.io.in.poke(  )
c.io.out.expect(  )
c.io.out.peek()
c.io.clock.step(n) //用于在测试模块中控制时钟
```