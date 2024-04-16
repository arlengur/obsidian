
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
- 
- that we have committed to using only pure functions
- a question naturally emerges
- Immersion
- dive in
- we need to know to get going
- brain-bending
- it's not crucial
- you internalize every single concept
- occurence
- as long as it doesn’t crash or hang
- code that spans multiple lines
- it permeates the functional pro- gramming style
- we’ll see how useful this capability really is
- how it permeates
- convention
- To advance to the next iteration
- it hinders good style.
- so long as the recursive call
- likewise
- it connotes
- may be elided
- this is something of a wart inherited from Java

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

def fib(n: Int): Int

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
`def isSorted[A](as: Array[A], ordered: (A,A) => Boolean): Boolean`