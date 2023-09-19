Это параметрический тип данных, который обязательно реализует две операции: 
- создание монады (unit или pure)
- функцию flatMap 
Применяются для связывания вычислений (однородных эффектов).

```scala
trait Monad[A] {
 // помещает значение в контекст
 def pure(a: A): Monad[A] 
 // применяет к значению функцию, возвращающую новое значение и контекст (монадическая функция)
 def flatMap(f: A => Monad[B]): Monad[B]
}
```

Законы, которые должны выполнять монады:
1. Left Identity: Применение функции к значению в монаде эквивалентно применению функции к значению: `pure(x).flatMap(f) == f(x)`
```scala
def incrementFunction(x: Int): Option[Int] = Some(x + 1)

Some(5).flatMap(incrementFunction) == incrementFunction(5)
```

2. Right Identity: Применение функции создания не меняет монаду: `m.flatMap(pure) == m`
```scala
def incrementFunction(x: Int): Option[Int] = Some(x + 1)

Some(5).flatMap(x => Some(x)) == Some(5)
```

3. Ассоциативность: уравнивает разные способы комбинации функций, `m.flatMap(f).flatMap(g) == m.flatMap(x => f(x).flatMap(g))`
```scala
def squareFunction(x: Int): Option[Int] = Some(x * x)
def incrementFunction(x: Int): Option[Int] = Some(x + 1)

val l = Some(5).flatMap(squareFunction).flatMap(incrementFunction)
val r = Some(5).flatMap(x => squareFunction(x).flatMap(incrementFunction))
assert(l == r)
```

Примеры: List, Exception, Future, Try, Either, Option, Set

Эти законы позволяют выполнять цепочки вычислений
```scala
def findHost(): Option[String] = Some("my.host.com") 
def findPort(): Option[Int] = Some(22) 

val address: Option[InetSocketAddress] = for { 
	host <- findHost() 
	port <- findPort() 
} yield new InetSocketAddress(host, port)

address.map(add => println("Address: " + add)).getOrElse(println("Error")) 
// Address : my.host.com/82.98.86.171:22
```

> [!note] 
> следует помнить что flatMap и map никогда не выполнится при отрицательных входных данных (для Option это None)
