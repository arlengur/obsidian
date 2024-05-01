Answers: https://github.com/ruivalentemaia/fpscala

- premise - предпосылка
- lead - приводить
- relearning - повторное обучение
- implication - последствие
- far-reaching - далеко идущий
- halt - остановка
- precise - точный
- beneficial - полезный
- tremendously - чрезвычайно
- gain - получать
- regain - восстановить
- exposure - контакт
- arguably - возможно
- overkill - перебор
- mourn - скорбеть, оплакивать
- elsewhere - в другом месте
- anticipate - ожидать
- make into - превратить в
- coalesce - объединять
- enormous - огромный
- emerge - возникать
- cohesive - единый, согласованный, целостный
- come - настать
	- you come to appreciate - ты начинаешь ценить
- necessitate - требуют
- pinpoint - точно определять
- relate - связывать, относиться
- solely - исключительно, только
- imply - предполагать
- somewhat - несколько, немного
- redundant - избыточный, излишний
- unless - если не
- nothing else - ничего больше
	- nothing else occurs - ничего больше не происходит
- whatever - независимо от
- constraint - ограничение
- violate - нарушать
- intact - нетронутый
- outcome - результат
- What’s more - более того
- in turn - в свою очередь
- throughout - везде, повсюду
- governing - управляющий
- composable - компануемый
- obtain - получить
- wart - бородавка
- commit - обязаться
	- we have committed to using functions - мы обязались использовать функции
- emerge - появляться, возникать
	- a question naturally emerges - естественно возникает вопрос
- Immersion - погружение
- dive in - погружаться в
- get going - начинать
- brain-bending - головокружительный
- crucial - киритческий
	- it's not crucial - это не критично
- internalize - усваивать
	- you internalize every single concept
- occurence - явление, частота
- as long as - до тех пор пока
	- as long as it doesn’t crash or hang - пока он не упадет или зависнет
- so long as - так как, пока
	- so long as the recursive call
- span - охватывать
	- code that spans multiple lines - код, занимающий несколько строк
- permeate - пронизывать
- convention - соглашение
- advance - продвигаться
	- To advance to the next iteration - Чтобы перейти к следующей итерации
- hinder - мешать
	- it hinders good style
- likewise - кроме того, аналогично
- connote - означать
- elide - игнорировать, пропускать
	- may be elided - может быть опущено
- universe - вселенная
- would go - пойдет
- puzzle - головоломка
- to puzzle together - собирать вместе
- keep going - продолжить, двигаться вперед
- one-liner - одна-строчка
- flavor - вкус, аромат
- employ - использовать
- preliminary - предварительный

over
- that may change over time - это может измениться со временем
- over the next several chapter - в следующих нескольких главах

- to get somebody to do something - убедить кого-то сделать что-то
- The intent is to get you thinking about programs - Цель в том, чтобы заставить вас думать о программах
- My goal is to speak English fluently - Моя цель состоит в том, чтобы говорить по английски свободно


Функция Высшего Порядка higher-order functions (HOFs) - функции которые принимают другие функции в своих аргументах и как результат могут возвращать функции

Чтобы получить доступ к объекту внутри которого объявлен метод используется ключевое слово this

```scala
def factorial(n: Int): Int = { 
	@annotation.tailrec
	def go(n: Int, acc: Int): Int =
		if (n <= 0) acc 
		else go(n-1, n*acc)
	go(n, 1) 
}
```

Фибоначчи
```scala
def fib (n:Int) : Int = {
	@annotation.tailrec
	def go(n:Int, prev:Int, acc:Int) : Int = {
		if (n <= 1) acc
		else go(n-1, acc, prev+acc)
	}
	go(n, 0, 1)
}
```

Полиморфизм в ООП это форма подтипов или отношений наследования
Параметрический полиморфизм когда мы хотим абстрагироваться от типа

Пример полиморфической функции
```
def findFirst[A](as: Array[A], p: A => Boolean): Int = {
	@annotation.tailrec
	def loop(n: Int): Int =  
		if (n >= as.length) -1 
		else if (p(as(n))) n 
		else loop(n + 1)
	loop(0)
}
```

findFirst(Array(7, 9, 13), (x: Int) => x == 9)

Array(7, 9, 13) - литерал массива, создает массив из 3х элементов
(x: Int) => x == 9 - функциональный литерал (анонимная функция)

Анонимная функция проверяющая равенство чисел
`(x: Int, y: Int) => x == y`
 это синтаксический сахар раскрывающийся в 
 ```
 val equality = new Function2[Int, Int, Boolean] { 
	 def apply(x: Int, y: Int) = x == y
}
```

тип `Function2[Int,Int,Boolean]` (индекс 2 указывает на количество аргументов) обычно записывают `(Int,Int) => Boolean`
Трейт Function2 имеет метод apply, поэтому когда мы вызываем equality(10, 20) происходит вызов equality.apply(10, 20)
Так как функции являются просто объектами, то мы называем их значениями первого класса (first-class values)

Проверить отсортирован ли массив
```scala
def isSorted[A](as: Array[A], ordered: (A,A) => Boolean) : Boolean = {
	@annotation.tailrec
	def loop(n:Int) : Boolean = 
		if(n >= as.length - 1) true
		else if (ordered(as(n),as(n+1))) loop(n+1)
		else false
	loop(0)
}
```

Следование типам для реализации
```scala
def partial1[A,B,C](a: A, f: (A,B) => C): B => C
def partial1[A,B,C](a: A, f: (A,B) => C): B => C = (b: B) => f(a, b)
// или так как мы уже сказали какой тип аргумента
def partial1[A,B,C](a: A, f: (A,B) => C): B => C = b => f(a, b)
```

Каррирование
```scala
def curry[A,B,C](f: (A, B) => C): A => (B => C)
def curry[A,B,C](f: (A, B) => C): A => (B => C) = (a: A) => ((b: B) => f(a, b))
```

Обратное каррирование
```scala
def uncurry[A,B,C](f: A => B => C): (A, B) => C
def uncurry[A,B,C](f: A => B => C): (A, B) => C = (a: A, b: B) => f(a)(b)
```

Кормпозиция функций
```scala
def compose[A,B,C](f: B => C, g: A => B): A => C
def compose[A,B,C](f: B => C, g: A => B): A => C = (a: A) => f(g(a))
```

Для композиции функций скала предоставляет метод compose определенный на Function1, и andThen делающий тоже самое
```scala
g compose f
f andThen g
```

