Apply это как Applicative но без pure то есть у него есть Functor + ap

Для него доступны
```scala
val a: Map[Int, Boolean] = Map(2 -> true, 3 -> false)  
val b: Map[Int, String] = Map(2 -> "foo", 3 -> "bar")  
  
(a *> b)  
(a, b).tupled  
(a, b).mapN { (b,s) =>  
b.toString + s  
}
```

в пао сбер разрешаю делать тому-то то-то
разблокировка карты
открытие 
получение
полукчение выписок и справок

Пример:
```scala
import cats.effect.{ExitCode, IO, IOApp}  
import cats.implicits._  
  
object Test extends IOApp {  
	final case class ProductId(value: String) extends AnyVal  
	  
	final case class Product(id: ProductId, name: String)  
	  
	final case class Inventory(productId: ProductId, amount: Int)  
	  
	val getProducts: IO[List[Product]] = IO.println("Loading products...").as(  
		List(  
			Product(ProductId("PRO1"), "Bread"),  
			Product(ProductId("PRO2"), "Milk"),  
			Product(ProductId("PRO3"), "Coffee")  
		)  
	)  
	val getInventory: IO[List[Inventory]] = IO.println("Loading inventory...").as(  
		List(  
			Inventory(ProductId("PRO1"), 4),  
			Inventory(ProductId("PRO2"), 2),  
			Inventory(ProductId("PRO2"), 4),  
			Inventory(ProductId("P3"), 1)  
		)  
	)  
	  
	def run(args: List[String]): IO[ExitCode] = {  
		val res = for {  
			products <- getProducts.map(_.groupByNel(_.id.value).mapValues(_.head))  
			inventory <- getInventory.map(_.groupByNel(_.productId.value).mapValues(_.reduceMap(_.amount)))  
		} yield {  
			val productsWithInventory: Map[String, (Product, Int)] = products  
				.map {  
					case (productId, product) =>  
					(productId, (product, inventory.get(productId)))  
				}  
				.collect {  
					case (k, (p, Some(i))) => (k, (p, i))  
				}  
			productsWithInventory  
		}  
		res.flatMap { map =>  
			map.toList.fmap(_.toString).traverse(IO.println(_))  
		}  
	}.as(ExitCode.Success)  
}
```

## Полугруппа (Semigroup)
```java
trait Semigroup[A] {
  def combine(x: A, y: A): A
}
```

Должна удовлетворять закону ассоциативности:
combine(x, combine(y, z)) = combine(combine(x, y), z)
для любых x, y, z типа А

Пример:
```java
case class Id(id: Int)  
implicit val semiId = new Semigroup[Id] {  
	def combine(x: Id, y: Id): Id = Id(x.id + y.id)  
}  
  
def sumId(id1: Id, id2: Id)(implicit sum: Semigroup[Id]): Id = sum.combine(id1, id2)  
  
implicit class SumId(a: Id) {  
	def |+|(b: Id)(implicit sum: Semigroup[Id]) = sum.combine(a, b)  
}  
  
println(sumId(Id(1), Id(2)))  
println(Id(1) |+| Id(2))
```

## Моноид (Monoid)
Идея: размножение типа
Это полугруппа с определенным на ней нулевым элементом
```java
trait Monoid[A] extends Semigroup[A] {
  def empty: A
}

// или

trait Monoid[A] {
  def empty: A
  def combine(x: A, y: A): A
}
```

Должен удовлетворять законам:
- Ассоциативности: combine(x, combine(y, z)) = combine(combine(x, y), z), для любых x, y, z типа А
- Нулевой элемент: combine(x, empty) == combine(empty, x) == x, для любого x типа А

Сложение
```scala
import cats.implicits._  

println(1 |+| 3 |+| Monoid[Int].empty)
println("foo" |+| "bar" |+| Monoid[String].empty)

case object X  
println(List(X, X, X) |+| List(X, X, X))
```

Map
```scala
import cats.syntax.all._  
val maps = List(  
	Map("a" -> List(1,2,3)),  
	Map("b" -> List(4,5,6)),  
	Map("a" -> List(7,8,9), "b" -> List(10,11,12))  
)  
maps.combineAll.foreach(println)
```

foldMap = map + combine
```scala
import cats.syntax.all._  
println(List("foo", "bar").map(_.length).combineAll) // 6
println(List("foo", "bar").foldMap(_.length)) // 6

// группировка по длине
val maps = List(  
"foo",  
"bar",  
"hello",  
"a"  
)  
println(maps.foldMap(s => Map(s.length -> NonEmptyList.one(s))))
// Map(5 -> NonEmptyList(hello), 1 -> NonEmptyList(a), 3 -> NonEmptyList(foo, bar))

// подсчет количества
val maps = List(  
"foo",  
"foo",  
"bar",  
"hello",  
"foo",  
"a"  
)  
println(maps.foldMap(s => Map(s -> 1)))
// Map(a -> 1, foo -> 3, hello -> 1, bar -> 1)
```

Произведение
```scala
import cats.syntax.all._  
  
implicit val monoidIntProduct = new Monoid[Int] {  
	def empty: Int = 1  
	def combine(x: Int, y: Int): Int = x * y  
}  
println(5 |+| 10) // 50
```

foldLeft & State & traverse
```scala
case class MyState(even: List[Int], odd: List[Int], div5: List[Int])  

// v1 можно перепутать s & withEvenOrOdd
println((1 to 10).toList.foldLeft(MyState(Nil, Nil, Nil)) { (s, next) =>  
	val withEvenOrOdd =  
		if (next % 2 == 0) s.copy(even = next :: s.even)  
		else s.copy(odd = next :: s.odd)  
	if (next % 5 == 0)  
		withEvenOrOdd.copy(div5 = next :: withEvenOrOdd.div5)  
	else withEvenOrOdd  
})

// v2 вынесем логику в отдельные методы
// В зависимости от полученного значения next обновляет состояние  
def step1(next: Int): MyState => MyState = s =>  
	if (next % 2 == 0) s.copy(even = next :: s.even) else s.copy(odd = next :: s.odd)  
  
def step2(next: Int): MyState => MyState = s =>  
	if (next % 5 == 0) s.copy(div5 = next :: s.div5) else s  
  
println((1 to 10).toList.foldLeft(MyState(Nil, Nil, Nil)) { (s, next) =>  
	step1(next)  
		.andThen(step2(next))  
		.apply(s)  
})

// v3 перепишем с применением State монады
// runS просто возвращает State
// на каждой итерации foldLeft мы зщапускаем runS, чтобы делать это 
// единожды можно использовать traverse
def step1_v2(next: Int): State[MyState, Unit] = State.modify { s =>  
	if (next % 2 == 0) s.copy(even = next :: s.even) else s.copy(odd = next :: s.odd)  
}  
def step2_v2(next: Int): State[MyState, Unit] = State.modify { s =>  
	if (next % 5 == 0) s.copy(div5 = next :: s.div5) else s  
}  
println((1 to 10).toList.foldLeft(MyState(Nil, Nil, Nil)) { (s, next) =>  
	step1_v2(next).flatMap(_ => step2_v2(next)).runS(s).value  
})
```