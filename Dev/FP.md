book https://drive.google.com/file/d/1E9SAZrEmDG8aBc_Va3FlI8xzdiodxMIm/view
Answers: https://github.com/ruivalentemaia/fpscala
https://github.com/fpinscala/fpinscala/blob/second-edition/answerkey/laziness/16.answer.md

Чтобы получить доступ к объекту внутри которого объявлен метод используется ключевое слово this

Функциональная структура данных обрабатывается с использованием только чистых функций. Неизменяема по определению
Примеры:
```scala
List() или Nil
3 или 4
a ++ b // конкатенация списков
```


Пример: односвязанный список

```scala
sealed trait List[+A]  
case object Nil extends List[Nothing]  
case class Cons[+A](head: A, tail: List[A]) extends List[A]  
  
object List {  
  def sum(ints: List[Int]): Int = ints match {  
    case Nil => 0  
    case Cons(x,xs) => x + sum(xs)  
  }  
  
  def product(ds: List[Double]): Double = ds match {  
    case Nil => 1.0  
    case Cons(0.0, _) => 0.0  
    case Cons(x,xs) => x * product(xs)  
  }  

  def apply[A](as: A*): List[A] =  
    if (as.isEmpty) Nil  
    else Cons(as.head, apply(as.tail: _*))  

  def append[A](a1: List[A], a2: List[A]): List[A] =
a1 match {
	case Nil => a2
	case Cons(h,t) => Cons(h, append(t, a2))
  }
}
```

Функция apply для объекта List называется вариативный, если она принимает 0 и больше аргументов типа А. Это синтаксический сахар для явной передачи списка элементов
```scala
def apply[A](as: A*): List[A] =
	if (as.isEmpty) Nil
	else Cons(as.head, apply(as.tail: _*))

List(1,2,3,4)
List("hi","bye")
```

Конструкция `_*` позволяет передать Seq в вариативный метод

Обмен данными - так как функциональные структуры неизменны, то при добавлении нового элемента в них нам не нужно копировать все содержимое мы его просто преиспользуем. Например, для добавления элемента в голову списка мы создаем новый список так `Cons(1,xs)` в этом случае мы переиспользуем имеющийся список xs.

Другой пример функциональной структуры данных Vector имеет постоянное время доступа к элементам, обновлению, head, tail, init, добавления в начало или конец.

## Улучшение выведения типов для функций высших порядков
У нас есть функция 
```scala
def dropWhile[A](l: List[A], f: A => Boolean): List[A]

val xs: List[Int] = List(1,2,3,4,5)
val ex1 = dropWhile(xs, (x: Int) => x < 4)
```

для ее вызова нам нужно указывать тип элементов анонимной функции, но так как первый аргумент содержит список Int, то анонимная функция должна принимать Int, чтобы помочь компилятору вывести тип нужно каррировать dropWhile
```scala
def dropWhile[A](as: List[A])(f: A => Boolean): List[A] =
as match {
	case Cons(h,t) if f(h) => dropWhile(t)(f)
	case _ => as
}

val xs: List[Int] = List(1,2,3,4,5)
val ex1 = dropWhile(xs)(x => x < 4)
```

в такой версии dropWhile возвращает функцию которая принимает функцию f и в этом случае нет необходимости указывать тип элементов анонимной функции компилятор сделает это сам

## Обобщение функций высшего порядка
Функции суммы и произведения очень похожи
```scala
def sum(ints: List[Int]): Int = ints match {
	case Nil => 0
	case Cons(x,xs) => x + sum(xs)
}

def product(ds: List[Double]): Double = ds match {
	case Nil => 1.0
	case Cons(x, xs) => x * product(xs)
}
```

Они различаются только возвращаемыми значениями, операцией применяемой к элементам и типом элементов списка, можно вынести эти значения в аргументы функций:
```scala
def foldRight[A,B](as: List[A], z: B)(f: (A, B) => B): B =
as match {
	case Nil => z
	case Cons(x, xs) => f(x, foldRight(xs, z)(f))
}

def sum2(ns: List[Int]) = foldRight(ns, 0)((x,y) => x + y)
def product2(ns: List[Double]) = foldRight(ns, 1.0)(_ * _)
```

Причем возвращаемый тип не должен быть таким же как у элементов списка

Посчитаем длину списка
```scala
def length[A](as: List[A]): Int = foldRight(as, 0)((_, acc) => acc + 1 )
```

Версия с хвостовой рекурсией
```scala
def foldLeft[A,B](as: List[A], z: B)(f: (B, A) => B): B = l match
  case Nil => z
  case Cons(h,t) => foldLeft(t, f(z,h), f

def sumViaFoldLeft(l: List[Int]) = foldLeft(l, 0, _ + _)
def productViaFoldLeft(l: List[Double]) = foldLeft(l, 1.0, _ * _)
def lengthViaFoldLeft[A](l: List[A]): Int = foldLeft(l, 0, (acc,_) => acc + 1)

def reverse[A](l: List[A]): List[A] = foldLeft(l, List[A](), (acc,h) => Cons(h,acc))

def appendViaFoldRight[A](l: List[A], r: List[A]): List[A] =
  foldRight(l, r, Cons(_,_))

// делает один список из списка списков
def concat[A](l: List[List[A]]): List[A] =
  foldRight(l, Nil:List[A], append)

// увелитчивает каждый элемент списка на 1
def incrementEach(l: List[Int]): List[Int] =
  foldRight(l, Nil:List[Int], (h,t) => Cons(h+1,t))

def doubleToString(l: List[Double]): List[String] =
  foldRight(l, Nil:List[String], (h,t) => Cons(h.toString,t))
  
def doubleToString(l: List[Double]): List[String] =
  foldRight(l, Nil:List[String], (h,t) => Cons(h.toString,t))

def filter[A](l: List[A], f: A => Boolean): List[A] =
  foldRight(l, Nil: List[A], (h, t) => if f(h) then Cons(h, t) else t)
  
def flatMap[A,B](l: List[A], f: A => List[B]): List[B] =
  concat(map(l, f))
  
def filterViaFlatMap[A](l: List[A], f: A => Boolean): List[A] =
  flatMap(l, a => if f(a) then List(a) else Nil)
  
def zipWith[A,B,C](a: List[A], b: List[B], f: (A,B) => C): List[C] = (a,b) match
  case (Nil, _) => Nil
  case (_, Nil) => Nil
  case (Cons(h1, t1), Cons(h2, t2)) => Cons(f(h1, h2), zipWith(t1, t2, f))
```

## Двоичное дерево
```scala
sealed trait Tree[+A]
case class Leaf[A](value: A) extends Tree[A]
case class Branch[A](left: Tree[A], right: Tree[A]) extends Tree[A]

def size[A](t: Tree[A]): Int = t match
  case Leaf(_) => 1
  case Branch(l, r) => 1 + l.size + r.size

def maximum(t: Tree[Int]): Int = t match
  case Leaf(n) => n
  case Branch(l, r) => l.maximum.max(r.maximum)

def depth(t: Tree[Int]): Int = t match
  case Leaf(_) => 0
  case Branch(l, r) => 1 + (l.depth.max(r.depth))

def map[B](t: Tree[Int])(f: A => B): Tree[B] = t match
  case Leaf(a) => Leaf(f(a))
  case Branch(l, r) => Branch(l.map(f), r.map(f))
  
def fold[A, B](t: Tree[A])(f: A => B, g: (B,B) => B): B = t match
  case Leaf(a) => f(a)
  case Branch(l, r) => g(l.fold(f, g), r.fold(f, g))

def sizeViaFold: Int = 
  fold(a => 1, 1 + _ + _)

def depthViaFold: Int = 
  fold(a => 0, (d1,d2) => 1 + (d1 max d2))

def mapViaFold[B](f: A => B): Tree[B] = 
  fold(a => Leaf(f(a)), Branch(_,_))
```

# Обработка ошибок без исключений
Почему исключения ломают ссылочную прозрачность
```scala
def failingFn(i: Int): Int = {
	val y: Int = throw new Exception("fail!")
	try {
		val x = 42 + 5
		x + y
	}
	catch { case e: Exception => 43 }
}
```

Если вызвать failingFn то получим `java.lang.Exception: fail!` а, если, следуя логике ссылочной прозрачности, мы заменим `у` значением, на которое оно ссылается
```scala
def failingFn2(i: Int): Int = {
	try {
		val x = 42 + 5
		x + ((throw new Exception("fail!")): Int)
	}
	catch { case e: Exception => 43 }
}
```

Если вызвать failingFn(12), то получим 43

Итак исключения
- ломают ссылочную прозрачность
- вносят зависимость от контекста (в зависимости от места использования исключение может быть выброшено или нет)
- не типобезопасные. Тип failingFn, Int => Int ничего не говорит о том какое исключение может произойти

Алтернативы исключениям

Рассмотрим частичную функцию, которая неопределена для пустого списка
```scala
def mean(xs: Seq[Double]): Double =
	if (xs.isEmpty)
		throw new ArithmeticException("mean of empty list!")
	else xs.sum / xs.length
```

# Option
```scala
sealed trait Option[+A] {  
  def map[B](f: A => B): Option[B] = this match {  
    case Some(a) => Some(f(a))  
    case None => None  
  }  
  def flatMap[B](f: A => Option[B]): Option[B] = map(f).getOrElse(None)  
  def getOrElse[B >: A](default: => B): B = this match { // В супер тип для А (для ковариантности)  
    case Some(a) => a  
    case None => default  
  }  
    
  def orElse[B >: A](ob: => Option[B]): Option[B] = map(Some(_)).getOrElse(None) // ob переменная по имени  
  def filter(f: A => Boolean): Option[A] = flatMap(a => if (f(a)) Some(a) else None)  
}
case class Some[+A](get: A) extends Option[A]
case object None extends Option[Nothing]

def mean(xs: Seq[Double]): Option[Double] =
	if (xs.isEmpty) None
	else Some(xs.sum / xs.length)

def variance(xs: Seq[Double]): Option[Double] = 
  mean(xs).flatMap(m => mean(xs.map(x => math.pow(x - m, 2))))

// поднимает функцию в контекст
def lift[A,B](f: A => B): Option[A] => Option[B] = _ map f

// поднимает функцию 2-х переменных в контекст
def map2[A,B,C](a: Option[A], b: Option[B])(f: (A, B) => C): Option[C] =  
  a.flatMap(aa => b.flatMap(bb => Some(f(aa, bb))))

// через for-comprehension
def map2[A,B,C](a: Option[A], b: Option[B])(f: (A, B) => C):
Option[C] =
	for {
		aa <- a
		bb <- b
	} yield f(aa, bb)

def Try[A](a: => A): Option[A] =
	try Some(a)
	catch { case e: Exception => None }

def sequence[A](as: List[Option[A]]): Option[List[A]] =
  as match
    case Nil => Some(Nil)
    case h :: t => h.flatMap(hh => sequence(t).map(hh :: _))

def sequence_1[A](as: List[Option[A]]): Option[List[A]] =  
  as.foldRight[Option[List[A]]](Some(Nil))((a, acc) => map2(a, acc)(_ :: _))

def traverse[A, B](as: List[A])(f: A => Option[B]): Option[List[B]] =
  as match
    case Nil => Some(Nil)
    case h::t => map2(f(h), traverse(t)(f))(_ :: _)

def traverse_1[A, B](as: List[A])(f: A => Option[B]): Option[List[B]] =
  as.foldRight[Option[List[B]]](Some(Nil))((h,t) => map2(f(h),t)(_ :: _))

def sequenceViaTraverse[A](as: List[Option[A]]): Option[List[A]] =
  traverse(as)(x => x)
```

# Either
```scala
sealed trait Either[+E, +A] {
	def map[B](f: A => B): Either[E, B] = 
	  this match
	    case Right(a) => Right(f(a))
	    case Left(e) => Left(e)
	  
	def flatMap[EE >: E, B](f: A => Either[EE, B]): Either[EE, B] =
	  this match
	    case Left(e) => Left(e)
	    case Right(a) => f(a)
	
	def orElse[EE >: E, AA >: A](b: => Either[EE, AA]): Either[EE, AA] =
	  this match
	    case Left(_) => b
	    case Right(a) => Right(a)
	
	def map2[EE >: E, B, C](b: Either[EE, B])(f: (A, B) => C): Either[EE, C] =
	  for
	    a <- this
	    b1 <- b
	  yield f(a,b1)
}
case class Left[+E](value: E) extends Either[E, Nothing]
case class Right[+A](value: A) extends Either[Nothing, A]

def Try[A](a: => A): Either[Exception, A] =
	try Right(a)
	catch { case e: Exception => Left(e) }

def traverse[E,A,B](as: List[A])(f: A => Either[E, B]): Either[E, List[B]] = 
  as match
    case Nil => Right(Nil)
    case h :: t => f(h).map2(traverse(t)(f))(_ :: _)

def traverse_1[E,A,B](as: List[A])(f: A => Either[E, B]): Either[E, List[B]] = 
  as.foldRight[Either[E,List[B]]](Right(Nil))((a, b) => f(a).map2(b)(_ :: _))

def sequence[E,A](as: List[Either[E,A]]): Either[E,List[A]] = 
  traverse(as)(x => x)
```

Пример
```scala
case class Person(name: Name, age: Age)
sealed class Name(val value: String)
sealed class Age(val value: Int)

def mkName(name: String): Either[String, Name] =
	if (name == "" || name == null) Left("Name is empty.")
	else Right(new Name(name))

def mkAge(age: Int): Either[String, Age] =
	if (age < 0) Left("Age is out of range.")
	else Right(new Age(age))

def mkPerson(name: String, age: Int): Either[String, Person] =
mkName(name).map2(mkAge(age))(Person(_, _))
```

# Строгость и ленивость
Функция называется нестрогой, если она может не вычислять один или несколько своих аргументов (аргументы которые не вычисляются называются аргументами по имени)
Функция называется строгой, если она вычисляет все свои аргументы  (аргументы которые вычисляются называются аргументами по значению)

Примеры нестрогих функций
`&&` вычисляет второй аргумент, если первый `true`
`||` вычисляет второй аргумент, если первый `false`
`if` в зависимости от условия вычисляет, то что находится в блоке if или в блоке else

```scala
def if2[A](cond: Boolean, onTrue: () => A, onFalse: () => A): A =
	if (cond) onTrue() else onFalse()

if2(a < 22,
	() => println("a"),
	() => println("b")
)
```

Тут `() => A` функция принимающая 0 аргументов и возвращающая `A` это алиас для типа `Function0[A]` (ее называют преобразователь) Такая функция вычисляется каждый раз когда к ней обращаются значение не кешируется)

```scala
sealed trait Stream[+A] {
	def toListRecursive: List[A] = this match
	  case Cons(h,t) => h() :: t().toListRecursive
	  case Empty => Nil
	
	def toList: List[A] =
	  @annotation.tailrec
	  def go(ll: LazyList[A], acc: List[A]): List[A] = ll match
	    case Cons(h, t) => go(t(), h() :: acc)
	    case Empty => acc.reverse
	  go(this, Nil)
	// версия без reverse с использованием mutable list, но так как ListBuffer не выходит за рамки toListFast, то эта функция тоже чистая
	def toListFast: List[A] =
	  val buf = new collection.mutable.ListBuffer[A]
	  @annotation.tailrec
	  def go(ll: LazyList[A]): List[A] = ll match
	    case Cons(h, t) =>
	      buf += h()
	      go(t())
	    case Empty => buf.toList
	  go(this)

	def take(n: Int): LazyList[A] = this match
	  case Cons(h, t) if n > 1 => cons(h(), t().take(n - 1))
	  case Cons(h, _) if n == 1 => cons(h(), empty)
	  case _ => empty
	
	@annotation.tailrec
	final def drop(n: Int): LazyList[A] = this match
	  case Cons(_, t) if n > 0 => t().drop(n - 1)
	  case _ => this

	def takeWhile(f: A => Boolean): LazyList[A] = this match
	  case Cons(h,t) if f(h()) => cons(h(), t().takeWhile(f))
	  case _ => empty

	def exists(p: A => Boolean): Boolean = this match {
		case Cons(h, t) => p(h()) || t().exists(p)
		case _ => false
	}

	def foldRight[B](z: => B)(f: (A, => B) => B): B =
		this match {
			case Cons(h,t) => f(h(), t().foldRight(z)(f))
			case _ => z
		}

	def exists(p: A => Boolean): Boolean = 
		foldRight(false)((a, b) => p(a) || b)

	def forAll(p: A => Boolean): Boolean =
		foldRight(true)((a,b) => p(a) && b)

	def takeWhile_1(p: A => Boolean): LazyList[A] =
		foldRight(empty)((a, b) => if p(a) then cons(a, b) else empty)

	def headOption: Option[A] =
		foldRight(None: Option[A])((h, _) => Some(h))

	def map[B](f: A => B): LazyList[B] =
	  foldRight(empty[B])((a, acc) => cons(f(a), acc))
	
	def filter(f: A => Boolean): LazyList[A] =
	  foldRight(empty[A])((a, acc) => if f(a) then cons(a, acc) else acc)
	
	def append[A2>:A](that: => LazyList[A2]): LazyList[A2] =
	  foldRight(that)((a, acc) => cons(a, acc))
	
	def flatMap[B](f: A => LazyList[B]): LazyList[B] =
	  foldRight(empty[B])((a, acc) => f(a).append(acc))
}
case object Empty extends Stream[Nothing]
case class Cons[+A](h: () => A, t: () => Stream[A]) extends Stream[A]

object Stream {
	def cons[A](hd: => A, tl: => Stream[A]): Stream[A] = {
		lazy val head = hd
		lazy val tail = tl
		Cons(() => head, () => tail)
	}
	def empty[A]: Stream[A] = Empty
	def apply[A](as: A*): Stream[A] =
		if (as.isEmpty) empty else cons(as.head, apply(as.tail: _*))
	def headOption: Option[A] = this match {
		case Empty => None
		case Cons(h, t) => Some(h()) // явно вычисляем значение h()
	}
}
```

Смарт-конструктор cons позволяет кэшировать значения head и tail после первого обращения, а также он удобен тем что Скала при выведении типов конструкторов данных использует подтипы то есть если мы создаем Stream при помощи Cons и Empty, то и тип у них будет соответствующий, а если при помощи cons, то тип будет Stream

Ленивость позволяет нам отделить описание выражения от его вычисления
```scala
def exists(p: A => Boolean): Boolean = this match {
	case Cons(h, t) => p(h()) || t().exists(p)
	case _ => false
}
```

Функция `||` нестрогая по второму аргументу, а потому 
- когда в списке будет найдено значение удовлетворяющее предикату вычисление будет остановлено и вернется true, 
- а tail имеющий тип lazy val не будет вычислен

Так как за один так работы вычисляется значение только одного элемента Stream, то функции применяемые к Stream не создают локальных копий Stream (что существенно сокращает потребление памяти при работе со Stream)
```scala
Stream(1,2,3,4).map(_ + 10).filter(_ % 2 == 0).toList
cons(11, Stream(2,3,4).map(_ + 10)).filter(_ % 2 == 0).toList

Stream(2,3,4).map(_ + 10).filter(_ % 2 == 0).toList
cons(12, Stream(3,4).map(_ + 10)).filter(_ % 2 == 0).toList

12 :: Stream(3,4).map(_ + 10).filter(_ % 2 == 0).toList
12 :: cons(13, Stream(4).map(_ + 10)).filter(_ % 2 == 0).toList

12 :: Stream(4).map(_ + 10).filter(_ % 2 == 0).toList
12 :: cons(14, Stream().map(_ + 10)).filter(_ % 2 == 0).toList

12 :: 14 :: Stream().map(_ + 10).filter(_ % 2 == 0).toList
12 :: 14 :: List()
```

поэтому мы можем реализовать find через filter вернув первый найденный элемент, если таковой имеется
```scala
def find(p: A => Boolean): Option[A] = filter(p).headOption
```

# Бесконечный Stream
```scala
val ones: Stream[Int] = Stream.cons(1, ones)

ones.take(5).toList // List(1, 1, 1, 1, 1)
ones.exists(_ % 2 != 0) // true
ones.map(_ + 1).exists(_ % 2 == 0)
ones.takeWhile(_ == 1)
ones.forAll(_ != 1) // stack overflow
```

Примеры
```scala
def continually[A](a: A): LazyList[A] =
  lazy val single: LazyList[A] = cons(a, single)
  single

def from(n: Int): LazyList[Int] =
  cons(n, from(n + 1))

val fibs =
  def go(current: Int, next: Int): LazyList[Int] =
    cons(current, go(next, current + next))
  go(0, 1)

def unfold[A, S](state: S)(f: S => Option[(A, S)]): LazyList[A] =
  f(state) match
    case Some((h,s)) => cons(h, unfold(s)(f))
    case None => empty
```

unfold - корекурсивная функция или защищенная рекурсия (производит данные), не нужно завершать пока она продуктивна, а продуктивна она пока функция f завершается

Рекурсивная функция потребляет данные, завершается на меньших входных данных

```scala
val fibsViaUnfold: LazyList[Int] =
  unfold((0,1)):
    case (current, next) =>
      Some((current, (next, current + next)))

def fromViaUnfold(n: Int): LazyList[Int] =
  unfold(n)(n => Some((n, n + 1)))

def continuallyViaUnfold[A](a: A): LazyList[A] =
  unfold(())(_ => Some((a, ())))

// could also of course be implemented as constant(1)
val onesViaUnfold: LazyList[Int] =
  unfold(())(_ => Some((1, ())))

def mapViaUnfold[B](f: A => B): LazyList[B] =
  unfold(this):
    case Cons(h, t) => Some((f(h()), t()))
    case _ => None

def takeViaUnfold(n: Int): LazyList[A] =
  unfold((this, n)):
    case (Cons(h, t), 1) => Some((h(), (empty, 0)))
    case (Cons(h, t), n) if n > 1 => Some((h(), (t(), n-1)))
    case _ => None

def takeWhileViaUnfold(f: A => Boolean): LazyList[A] =
  unfold(this):
    case Cons(h, t) if f(h()) => Some((h(), t()))
    case _ => None

def zipAll[B](that: LazyList[B]): LazyList[(Option[A], Option[B])] =
  unfold((this, that)):
    case (Empty, Empty) => None
    case (Cons(h1, t1), Empty) => Some((Some(h1()) -> None) -> (t1() -> Empty))
    case (Empty, Cons(h2, t2)) => Some((None -> Some(h2())) -> (Empty -> t2()))
    case (Cons(h1, t1), Cons(h2, t2)) => Some((Some(h1()) -> Some(h2())) -> (t1() -> t2()))

def zipWith[B,C](that: LazyList[B])(f: (A,B) => C): LazyList[C] =
  unfold((this, that)):
    case (Cons(h1, t1), Cons(h2, t2)) =>
      Some((f(h1(), h2()), (t1(), t2())))
    case _ => None

// special case of `zipWith`
def zip[B](that: LazyList[B]): LazyList[(A,B)] =
  zipWith(that)((_,_))

def zipWithAll[B, C](that: LazyList[B])(f: (Option[A], Option[B]) => C): LazyList[C] =
  LazyList.unfold((this, that)):
    case (Empty, Empty) => None
    case (Cons(h, t), Empty) => Some(f(Some(h()), Option.empty[B]) -> (t(), empty[B]))
    case (Empty, Cons(h, t)) => Some(f(Option.empty[A], Some(h())) -> (empty[A] -> t()))
    case (Cons(h1, t1), Cons(h2, t2)) => Some(f(Some(h1()), Some(h2())) -> (t1() -> t2()))

def zipAllViaZipWithAll[B](s2: LazyList[B]): LazyList[(Option[A],Option[B])] =
  zipWithAll(s2)((_,_))
```

```scala
/*
`s.startsWith(s2)` when corresponding elements of `s` and `s2` are all equal, until the point that `s2` is exhausted. If `s` is exhausted first, or we find an element that doesn't match, we terminate early. Using non-strictness, we can compose these three separate logical steps--the zipping, the termination when the second lazy list is exhausted, and the termination if a nonmatching element is found or the first lazy list is exhausted.
*/
def startsWith[A](prefix: LazyList[A]): Boolean =
  zipAll(prefix).takeWhile(_(1).isDefined).forAll((a1, a2) => a1 == a2)

/*
The last element of `tails` is always the empty `LazyList`, so we handle this as a special case, by appending it to the output.
*/
def tails: LazyList[LazyList[A]] =
  unfold(this):
    case Empty => None
    case l @ Cons(_, t) => Some((l, t()))
  .append(LazyList(empty))

def hasSubsequence[A](s: Stream[A]): Boolean = tails exists (_ startsWith s)

/*
The function can't be implemented using `unfold`, since `unfold` generates elements of the `LazyList` from left to right. It can be implemented using `foldRight` though.
The implementation is just a `foldRight` that keeps the accumulated value and the lazy list of intermediate results, which we `cons` onto during each iteration. When writing folds, it's common to have more state in the fold than is needed to compute the result. Here, we simply extract the accumulated list once finished.
*/
def scanRight[B](init: B)(f: (A, => B) => B): LazyList[B] =
  foldRight(init -> LazyList(init)): (a, b0) =>
    // b0 is passed by-name and used in by-name args in f and cons. So use lazy val to ensure only one evaluation...
    lazy val b1 = b0
    val b2 = f(a, b1(0))
    (b2, cons(b2, b1(1)))
  ._2

Stream(1,2,3).scanRight(0)(_ + _).toList // List(6,5,3,0)
```

# Генератор случайных чисел с побочным эффектом
```scala
val rng = new scala.util.Random
rng.nextDouble
rng.nextInt
rng.nextInt(10) // случайное число от 0 до 9
```

# Генератор случайных чисел чисто функциональный
```scala
trait RNG {  
  def nextInt: (Int, RNG)  
}  
  
case class SimpleRNG(seed: Long) extends RNG {  
  def nextInt: (Int, RNG) = {  
    val newSeed = (seed * 0x5DEECE66DL + 0xBL) & 0xFFFFFFFFFFFFL  
    val nextRNG = SimpleRNG(newSeed)  
    val n = (newSeed >>> 16).toInt  
    (n, nextRNG)  
  }  
}  
  
val rng = SimpleRNG(42)  
println(rng.nextInt)
```

Генератор чисел от 0 до Int.maxValue включительно
```scala
// Since `Int.Minvalue` is 1 smaller than `-(Int.MaxValue)`,
// it suffices to increment the negative numbers by 1 and make them positive.
// This maps Int.MinValue to Int.MaxValue and -1 to 0.
def nonNegativeInt(rng: RNG): (Int, RNG) =
  val (i, r) = rng.nextInt
  (if i < 0 then -(i + 1) else i, r)
```

Генератор чисел от 0 до 1, не включая 1
```scala
def double(rng: RNG): (Double, RNG) =
  val (i, r) = nonNegativeInt(rng)
  (i / (Int.MaxValue.toDouble + 1), r)
```

Еще
```scala
def intDouble(rng: RNG): ((Int, Double), RNG) =
  val (i, r1) = rng.nextInt
  val (d, r2) = double(r1)
  ((i, d), r2)

def doubleInt(rng: RNG): ((Double, Int), RNG) =
  val ((i, d), r) = intDouble(rng)
  ((d, i), r)

def double3(rng: RNG): ((Double, Double, Double), RNG) =
  val (d1, r1) = double(rng)
  val (d2, r2) = double(r1)
  val (d3, r3) = double(r2)
  ((d1, d2, d3), r3)
```

Список случайных чисел
```scala
// A simple recursive solution
def ints(count: Int)(rng: RNG): (List[Int], RNG) =
  if count <= 0 then
    (List(), rng)
  else
    val (x, r1)  = rng.nextInt
    val (xs, r2) = ints(count - 1)(r1)
    (x :: xs, r2)

// A tail-recursive solution
def ints2(count: Int)(rng: RNG): (List[Int], RNG) =
  def go(count: Int, r: RNG, xs: List[Int]): (List[Int], RNG) =
    if count <= 0 then
      (xs, r)
    else
      val (x, r2) = r.nextInt
      go(count - 1, r2, x :: xs)
  go(count, rng, List())
```

Наши функции имеют вид `RNG => (A, RNG)` такие функции называются переходы или действия состояний (state actions) так как они переводят RNG из одного состояния в другое
Создадим тайп-алиас
```scala
type Rand[+A] = RNG => (A, RNG)

// тогда
val int: Rand[Int] = _.nextInt

// напишем DSL (domain-specific language)
def unit[A](a: A): Rand[A] = rng => (a, rng)

def map[A,B](s: Rand[A])(f: A => B): Rand[B] =
	rng => {
		val (a, rng2) = s(rng)
		(f(a), rng2)
	}

// пример map
def nonNegativeEven: Rand[Int] = 
	map(nonNegativeInt)(i => i - i % 2)

val double: Rand[Double] = 
	map(nonNegativeInt)(i => i / (Int.MaxValue.toDouble + 1))

def map2[A, B, C](ra: Rand[A], rb: Rand[B])(f: (A, B) => C): Rand[C] =
  rng0 =>
    val (a, rng1) = ra(rng0)
    val (b, rng2) = rb(rng1)
    (f(a, b), rng2)

def both[A,B](ra: Rand[A], rb: Rand[B]): Rand[(A,B)] =
	map2(ra, rb)((_, _))

val randIntDouble: Rand[(Int, Double)] =
	both(int, double)

val randDoubleInt: Rand[(Double, Int)] =
	both(double, int)

def sequence[A](rs: List[Rand[A]]): Rand[List[A]] =
  rs.foldRight(unit(Nil: List[A]))((r, acc) => map2(r, acc)(_ :: _))

def _ints(count: Int): Rand[List[Int]] =
  sequence(List.fill(count)(int))
```