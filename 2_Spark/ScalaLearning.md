
## [符号语法糖][1]
---

- if
```scala
if(布尔表达式){
   // 如果布尔表达式为 true 则执行该语句块
}else{
   // 如果布尔表达式为 false 则执行该语句块
}

```
---
## 函数
```scala
def functionName ([参数列表]) : [return type] = {
   function body
   return [expr]
}
```
- 匿名函数
```scala
var f = (x:Int) => x + 1
```
- 简化函数
```scala
// 定义超类
def f[A](x: A) = "[" + x.toString() + "]"
def f[String](x: String) = "[" + x.toString() + "]"
```
- 可变参数函数
```scala
def addInt(a: Int*): Int = {
  var l = 0
  for (i <- a) {
    l = l + i
  }
  return l
}
```
- 递归函数
```scala
def factorial(n: BigInt): BigInt = {
  if (n <= 1)
    1
  else
    n * factorial(n - 1)
}
```

- 偏应用函数
```scala
import java.util.Date

object Test {
   def main(args: Array[String]) {
      val date = new Date
      val logWithDateBound = log(date, _ : String) // 相当于函数赋值生成新函数

      logWithDateBound("message1" )
      Thread.sleep(1000)
      logWithDateBound("message2" )
      Thread.sleep(1000)
      logWithDateBound("message3" )
   }

   def log(date: Date, message: String)  = {
     println(date + "----" + message)
   }
}
```
- 高阶函数
```scala
object Test {
  def main(args: Array[String]) {

    println( apply( layout, 10) )

  }
  // 函数 f 和 值 v 作为参数，而函数 f 又调用了参数 v
  def apply(f: Int => String, v: Int) = f(v)

  def layout[A](x: A) = "[" + x.toString() + "]"

}
```


---
[1]: http://blog.csdn.net/bobozhengsir/article/details/13023023
