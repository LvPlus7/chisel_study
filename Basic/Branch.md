# Branch
## Condition
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
> when不会返回值 if会返回值
## Switch 
```SCALA
switch (x) {
is( value1 ) {
// run if x === value1
}
is( value2 ) {
// run if x === value2
}
}
```