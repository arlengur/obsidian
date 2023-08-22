Ресурсы - это компоненты системы, доступность которых ограничена и выход за их границы ведет к недоступности, отказу системы.

Традиционный подход: получить ресурс -> использовать ресурс -> освободить ресурс.
```java 
val resource = acquireResource  
try{  
	use(resource)  
} finally {  
	releaseResource(resource)  
}  
```
Такой подход не позволяет делать композицию то есть когда ресурсов много, то код будет дублироваться. И тут приходится все делать в одном месте получать, использовать и освобождать, что может быть неудобно в больших проектах.

Функциональный подход
```java
def withResource[R, A](r: R)(release: R => Any)(use: R => A): A = {  
	try {  
		use(r)  
	} finally {  
		release(r)  
	}  
}
...
val result: List[String] = withResource(Source.fromFile("test.txt"))(_.close()){ r =>  
	r.getLines().toList  
}
```

Параллельный подход
```java
def acquireFutureResource: Future[Resource] = ???  
def use(resource: Resource): Future[Unit] = ???  
def releaseFutureResource(resource: Resource): Future[Unit] = ???  
  
// оператор ensuring, позволит корректно работать с ресурсами в контексте Future 
implicit class FutureOps[A](future: Future[A]){  
	def ensuring(finalizer: => Future[Any]): Future[A] = future.transformWith {  
		case Failure(exception) => finalizer.flatMap(_ => Future.failed(exception))  
		case Success(value) => finalizer.flatMap(_ => Future.successful(value))  
	}  
}
```
В обоих случаях мы выполняем финалайзер и возвращаем результат

```java
// Написать код, который получит ресурс, воспользуется им и освободит  
lazy val result2Future = acquireFutureResource.flatMap(r =>  
use(r).ensuring(releaseFutureResource(r))  
)
```

В Zio получение и освобождение ресурса может быть прервано

Если во время работы finalizer мы получим ошибку то ensuring вернет ошибку finalizer а не ошибку выполнения, мы теряем exception

## ZIO bracket 
гарантирует что прерывание не произойдет во время получения и освобождения ресурса
освобождение ресурса происходит сразу после использования
```java
object ZIO { 
	def bracket[R, E, A, B]( 
		acquire: ZIO[R, E, A], 
		release: A => URIO[R, Any], 
		use: A => ZIO[R, E, B]
	): ZIO[R, E, B] = ???
}
```

```java
object ZIO { 
	def bracketExit[R, E, A, B]( 
		acquire: ZIO[R, E, A], 
		release: (A, Exit[E, B]) => URIO[R, Any], 
		use: A => ZIO[R, E, B]
	): ZIO[R, E, B] = ??? 
}
```
где Exit - обертка над результатом
```java
trait ZIO[-R, +E, +A] { 
	def ensuring[R1 <: R](finalizer: URIO[R1, Any]): ZIO[R1, E, A] = ??? 
}
```
ensuring гарантирует что finalizer будет вызван после выполнения зио эффекта

- получение ресурса нельзя прервать 
- освобождение ресурса нельзя прервать 
- если получение случилось(acquire), то освобождение(release) случится сразу же, как закончится использование(use)

Пример:
```java
// реалтзовать ф-цию, которая будет описывать открытие файла с помощью ZIO эффекта  
def openFile(fileName: String): Task[BufferedSource] =  
	ZIO.effect(Source.fromFile(fileName))  
// реалтзовать ф-цию, которая будет описывать закрытие файла с помощью ZIO эффекта  
def closeFile(file: Source) =  ZIO.effect(file.close()).orDie  
  
// Написать эффект, котрый прочитает строчки из файла и выведет их в консоль  
def handleFile(file: Source): ZIO[Any, Throwable, List[Unit]] = ZIO.foreach(file.getLines().toList){ str =>  
	ZIO.effect(println(str))  
}  
  
/**  
* Написать эффект, который откроет 2 файла, прочитает из них строчки,  
* выведет их в консоль и корректно закроет оба файла  
*/  
val twoFiles = ZIO.bracket(openFile("test1.txt"))(closeFile){ f1 =>  
	ZIO.bracket(openFile("test2.txt"))(closeFile){ f2 =>  
		handleFile(f1) zipRight handleFile(f2)  
	}  
}
```

## ZManaged 
- дает композицию
- позволяет абстрагироваться от ресурсов (получение и закрытие ресурсов происходит отдельно от использования)
- закрытие ресурсов происходит в обратном порядке открытию
- получение и освобождение ресурсов можно делать параллельно
- zio можно привести к zmanaged

bracket - базовый оператор 
- не позволяет абстрагироваться от получения / закрытия ресурса 
- нет композиции

```java
final case class ZManaged[-R, +E, A]( 
	acquire: ZIO[R, E, A], 
	release: A => URIO[R, Any] 
)
```

Конструирование 
```java
def make[R, E, A](acquire: ZIO[R, E, A])(release: A => URIO[R, Any]): ZManaged[R, E, A] 
def fromEffect[R, E, A](fa: ZIO[R, E, A]): ZManaged[R, E, A] 
def fromAutoCloseable[R, E, A <: AutoCloseable](fa: ZIO[R, E, A]): ZManaged[R, E, A]
```

Комбинирование 
```java
def map[B](f: A => B): ZManaged[R, E, B] 
def flatMap[R1 <: R, E1 >: E, B](f: A => ZManaged[R1, E1, B]): ZManaged[R1, E1, B] 
def zip[R1 <: R, E1 >: E, B](that: ZManaged[R1, E1, B]): ZManaged[R1, E1, (A, B)]
```

Использование 
```java
def use[R1 <: R, E1 >: E, B](f: A => ZIO[R1, E1, B]): ZIO[R1, E1, B] 
def use_[R1 <: R, E1 >: E, B](f: ZIO[R1, E1, B]): ZIO[R1, E1, B] 
def useNow: ZIO[R, E, A] val // использует и сразу освобождает
useForever: ZIO[R, E, Nothing] // использует и не освобождает
```

Пример:
```java
final case class ZManaged[-R, +E, A](  
	acquire: ZIO[R, E, A],  
	release: A => URIO[R, Any]  
	){ self =>  

	def use[R1 <: R, E1 >: E, B](f: A => ZIO[R1, E1, B]): ZIO[R1, E1, B] =  
	acquire.bracket(release)(f)  
	
	def map[B](f: A => B): ZManaged[R, E, B] = ???  
	
	def flatMap[R1 <: R, E1 >: E, B](f: A => ZManaged[R1, E1, B]): ZManaged[R1, E1, B] = ???  
}
```

Создание и использование
```java
// написать эффект открывающий / закрывающий первый файл  
lazy val file1: ZManaged[Any, Throwable, BufferedSource] =  
	ZManaged.make(openFile("test1.txt"))(closeFile)  
  
lazy val file2 = ZManaged.make(openFile("test2.txt"))(closeFile)  
  
// Написать эффект, котрый восользуется ф-цией handleFile из блока про bracket для печати строчек в консоль  
lazy val printFile1 = file1.use(handleFile)
```

Комбинирование
```java
lazy val combined: ZManaged[Any, Throwable, (BufferedSource, BufferedSource)] = file1 zip file2  

// Паралельное открытие / закрытие  
lazy val combined2 = file1 zipPar file2  
  
// Написать эффект, который прочитает и выведет строчки из обоих файлов  
val combinedEffect = combined.use{  
	case (f1, f2) => handleFile(f1) zipRight handleFile(f2)  
}
```

Множество ресурсов
```java
lazy val fileNames: List[String] = ???  
  
def file(name: String): ZManaged[Any, IOException, Source] = ???  
  
// множественное открытие / закрытие  
lazy val files: ZManaged[Any, IOException, List[Source]] = ZManaged.foreach(fileNames){ n =>  
	file(n)  
}  
  
// паралельное множественное открытие / закрытие  
lazy val files2: ZManaged[Any, IOException, List[Source]] = ???  
  
// обработать N файлов  
lazy val r1: ZIO[Any, Throwable, List[String]] = files2.use{ l =>  
	ZIO.foreach(l){ r =>  
		ZIO.effect(r.getLines())  
	}.map(_.flatten)  
}
```

Конструирование
```java
lazy val eff1: Task[Int] = ???  

// Из эффекта  
lazy val m1 = ZManaged.fromEffect(eff1)  
  
type Transactor  
def mkTransactor(c: Config): ZManaged[Any, Throwable, Transactor] = ???  
  
// микс ZManaged и ZIO  
type Config  
val config: Task[Config] = ???  
  
lazy val m2 = for{  
	c <- config.toManaged_  
	t <- mkTransactor(c)  
} yield t
```

