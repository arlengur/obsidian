pycharm - IDE

Интерпретируемый язык программирования т.е. выполняет код строка за строкой, поэтому ошибка может быть найдена только в момент исполнения кода, в отличие от этого компилируемые языки программирования такой код даже не скомпилируют.

Чувствителен к регистру.

Динамически типизированный т.е. тип переменной может меняться по ходу программы.

Операции сравнения можно объединять в цепочку: `10 <= a < 100`

```python
print("Hello") # Hello

my_val = 40 # int
my_float = 3.14 # float
print(f"Hello {my_val}") # Hello 40

int(5/4) # 1 - integer number
```

# Операторы

|   |   |
|---|---|
|//|целочисленное деление|
|%|остаток от деления|
|\*\*|возведение в степень|
|=|присваивание|
|+=|оператор приращения сложения|
|-=|оператор приращения вычитания|
|\*=|оператор приращения умножения|
|/=|оператор приращения вещественного деления|
|//=|оператор приращения целочисленного деления|
|%=|оператор приращения остатка от деления|
|\*\*=|оператор приращения возведения в степень|

Множественное присваивание в одной строке: 
```
x1, x2, x3 = False, True, True
```
# Типы данных (неизменяемые)

|   |   |
|---|---|
|int|целые числа|
|float|вещественные (0.5 или 1e-1)|
|bool|логические|
|str|строки|

# Преобразование типов

|   |   |
|---|---|
|type(x)|вывести тип объекта|
|int(x)|преобразование к целому числу|
|float(x)|преобразование к числу с плавающей точкой|

# Логические операции и операции сравнения

|   |
|---|
|or|
|and|
|not|
|>|
|>=|
|<|
|<=|
|\==|
|!=|

# Ввод данных

|   |   |
|---|---|
|input()<br>input('input data')|читает строку с клавиатуры|
|s = input()<br>a, b = input().split|записать в переменную пользовательский ввод|
|a = int(input())|прочитать строку с клавиатуры и преобразовать в число|
|print('Hello')<br>print(a, b)|вывод данных|

# Условные операторы

```python
if cond_1:
 code_1
 elseif cond_2: // or elif
  code_2
else:
 code_3
```

# Строки
Строка - это список символов.
```python
‘str’
“str”
‘’’big line’’’
“””big line”””
```

## Операции со строками

```python
‘str’ + ‘2’ // ‘str2’
‘str’ * 3 // ‘strstrstr’
len(‘str’) // 3
‘str’ == “str” // True
‘abc’ < ‘ac’ // True
‘abc’ > ‘ab’ // True

genome = 'ATGG'
genome[0] // A
genome[-1] // G
genome[1:4]
genome[:4] // сначала до 4
genome[4:] // от 4 до конца
genome[-4:] // 4 с конца и до конца
genome[1:-1]
genome[1:-1:2] // откуда и докуда и шаг
genome[::-1] // символы в обратном порядке
genome.upper()
genome.lower()
genome.find('c') // первое вхождение (индекс) c в genome либо -1
if 'c' in genome: // проверка вхождения в строку
genome.replace('c', 'C') //  заменить все вхождения с на С
genome.count('C') // сколько раз строка С встречается в строке genome
```
# Комментарий

```
# single line
‘’’ multiple line ‘’’
“””multiple line”””
```

# Циклы

## while

```
while cond:
 code
```

## for
```python
for var in 2,3,5:
 code

for var in range(n):
 code

a, b =(int(i) for var in input().split())

genome = 'ATGG'
for c in genome:
 print(c)

x = [1, 2, 3]
for i in x:
 print(i)

break # прерывает выполнение цикла
continue # переход к следующей итерации цикла
```

# range
range - функция возвращающая последовательность

```python
range(10) # 0,1..9
range(2,5) # 2,3,4
range(2,15,4) # 2,6,10,14
range(start=0, to, step=1)
```

# Списки
Являются изменяемой структурой данных
```python
list = [1,2,3,4] # list
list[0] # 1 - first elem
list[-1] # 4 - last elem
list[0:2] # [1, 2] - first two elems
list[:2] # [1, 2] - first two elems
list[1:3] # [1, 2] - last two elems
list[1:] # [1, 2] - all elems after second elem
sum(list) # 10 - list sum
len(list) # 4 - list length
list.append(5) # add value to list

x = [1, 2, 3]
y = [4, 5, 6]
x + y # [1, 2, 3, 4, 5, 6]
x * 2 # [1, 2, 3, 1, 2, 3]
x += 5 # [1, 2, 3, 5]
x.insert(1, 7) # [1, 7, 2, 3]
x.remove(7)# удаляет первое вхождение переданного значения
del x[0]# удалить элемент по индексу
if 3 in x: 
	print(‘yes’)
if 3 not in x:
	print(‘yes’)
	
if 3 not x:
	print(‘yes’)
x.sort() # сортировка

ind = x.index(3) # возвращает индекс элемента или ValueError
x.min()
x.max()
x.reverse()
```
## Генерация списков
```python
a=[0]*5
a=[0 for i in range(5)]
a=[i*i for i in range(5)]
a=[int(i) for i in input().split()]
```

## Двумерный список

```python
a=[[1,2,3],[4,5,6],[7,8,9]]
```

## Генерация двумерного списка 
```python
n=3
a=[[0]*n]*n // создали строку из n элементов и скопировали ссылку на нее n раз
a[0][0]=5 // сделает первый столбец такой матрицы равным 5
a=[[0]*n for i in range(n)] // создает двумерный список
a=[[0 for j in range(n)] for i in range(n)] // создает двумерный список
```

# Функции

```python
def func_name(params):
 func_body
 return
```

Функция должна быть объявлена ранее первого вызова:
`funcVal = func_name(param)`

Процедура - функция, которая ничего не возвращает.

Функция с произвольным числом параметров:
```python
def min(*a):
 m = a[0]
 for x in a:
  if m > x:
   m = x
 return m
  
min(4,5)
min([4,5])
```

## Параметр по умолчанию
`def func_name(param = 1) `- функция с значением параметра по умолчанию

## Передача параметра по имени
`func_name(param = 1)` - функция с передачей параметра по имени

## Локальные переменные
Локальные переменные - переменные указанные внутри функции и доступные только внутри нее.

Объекты связанные с локальными переменными можно изменить, если переданный объект является изменяемым значением:
```python
def append_zero(xs):
 xs.append(0)

a=[]
append_zero(a)
print(a) #[0]
```

## Глобальные переменные
Глобальная переменная - переменная объявленная вне всех функций.

Если внутри функции используется локальная переменная с таким же именем как у глобальной переменной, то внутри функции будет доступна только локальная версия переменной.

# Множества
```python
s = set() # создание пустого множества
basket = {'apple', 'banana'}
'orange' in basket # Проверка нахождения элемента в множестве
```

## Операции с множествами


|   |   |
|---|---|
|add(el)|добавление элемента|
|remove(el)|удаление элемента, если элемента нет, то ошибка|
|discard(el)|удаление элемента независимо от его нахождения в множестве|
|clear()|очищение множества|
|len(s)|число элементов в множестве|
|for x in basket:<br> print(x)|перебор элементов множества|

# Словарь
Ассоциативный массив, изменяемый, ключи должны быть неизменяемыми элементами, элементами могут быть списки

```python
my_account = {
	"name": "Alex",
	"balance": 3000,
	"open_date": "12-05-2006"
}

my_account["balance"]
if "name" in my_account: # if record with key "name" exist
	print("found")
else:
	print("not found")
```

|   |   |
|---|---|
|dict(), {}|создание словаря|
|key in d<br>key not in|Проверка нахождения элемента в словаре|
|d[10] = value|добавление пары в словарь|
|d[key]|получение значения, ошибка при отсутствии|
|d.get(key)|получение значения, None при отсутствии|
|del d[key]|удалить элементы из словаря|
|d.keys()|возвращает множество ключей словаря|
|d.values()|множество значений словаря|
|d.items()|множество пар|
# Обработка исключений
```python
try:
	# do smth
except Exception:
	# do smth
```
# Чтение из файла
```python
inf = open('file_name', 'r')
inf.readline()
inf.close()

# при выходе из этой конструкции файл будет автоматически закрыт
with open('file_name') as inf:
 s1 = inf.readline()

inf.readline().strip() - убрать все служебные символы при чтении строки

import os
os.path.join('.', 'dir', 'file') # './dir/file'
```

## Построчное чтение файла
```python
with open('file_name') as inf:
 for line in inf:
  line = line.strip()
  print(line)
```

# Запись в файл

```python
ouf = open('file_name', 'w')
ouf.write('text\n')
ouf.write(str(25)) - записать число 25 в троку
ouf.close()
```

# Модули
Функции выделенные в отдельный файл
Имя файла = имя модуля + .py

```python
import mudule_name
mudule_name.foo()
```

## Импортирование модуля
```python
from mudule_name import foo
foo()

from mudule_name import *
foo()

from mudule_name import foo as my_foo
my_foo()
```
## Установка дополнительных модулей
`conda install requests`

## Модуль запуска внешних процессов
```python
subprocess
subprocess.call(["python", "-h"]) # запуск программы с параметром
```

# Библиотеки
pip install pyTelegramBotAPI  - ТГ бот
pip install requests - для отправки запросов
## string
`string.ascii_letters`

## random
`random.choice(string.ascii_letters)`
## hashlib
sha256 создает строку в 32 байта то есть 64 символа в шестнадцатеричном формате
```python
import hashlib
text = "Hello"
bynary = text.encode() # move to bynary array
hashlib.sha256(bynary).hexdigest() # hash from bynary array
```
## json
```python
import json
json.dumps(my_account) # move to line
json.dumps(data, sort_keys = True) # sort alphabetically
```

## NumPy
Библиотека для анализа данных

```python
a = array([2,3,4])
a = array([(1.5, 2, 3), (2, 3, 4)])
a.ndim - размерность массива
a.shape - размеры массива (число строк, столбцов)
a.size - общее количество элементо
zeros((3, 2)) - заполнение массива нулями
arange(10, 30, 5) - создание массива по заданному диапазону
linspace(0, 2, 9) - генерирует 9 чисел на отрезке от 0 до 2 с равным шагом
```



```python
import hashlib
import json
def dataToHash(data):
	json_data = json.dumps(data, sort_keys = True)
	bynary_data = json_data.encode()
	return hashlib.sha256(bynary_data).hexdigest()[:10]

def isValidHash(hash):
	return hash[0:4] == "0000"

# proof - доказательство работы (это число нужно подобрать, чттобы хэш начинался с нулей)
def isValidProof(block, proof):
	block_copy = block.copy()
	block_copy["proof"] = proof
	hash = dataToHash(block_copy)
	return isValidHash(hash) # начинается ли новый хэш с 4х нулей

# намайнить такое число, чтобы добавив его к блоку хэш начинался с 2х нулей
def mineProofOfWork(block):
	proof = 0
	while not isValidProof(block, proof):
		proof += 1
	return proof

genesis_block = {
	"from": "",
	"to": "",
	"amount": 0.0
}

genesis_block["proof"] = mineProofOfWork(genesis_block)

blockchain = [genesis_block]

def addNewBlock(account_from, account_to, amount):
	prev_block = blockchain[-1]
	prev_hash = dataToHash(prev_block)
	block = {
		"from": account_from,
		"to": account_to,
		"amount": amount,
		"prev_hash": prev_hash
	}
	proof = mineProofOfWork(block) # майнинг блока
	block["proof"] = proof
	blockchain.append(block)

addNewBlock("Alex", "Vasya", 2000)
addNewBlock("Alex", "Petya", 14)
addNewBlock("Alex", "Kolya", 88)
addNewBlock("Alex", "Nastya", 69420)
addNewBlock("Vasya", "Petya", 1000)
addNewBlock("Petya", "Nastya", 500)
addNewBlock("Petya", "Kolya", 500)

def validateBlockchain():
	prev_block = None
	for block in blockchain:
		if prev_block:
			actual_prev_hash = dataToHash(prev_block)
			recordered_prev_hash = block["prev_hash"]
			if not isValidHash(actual_prev_hash):
				print(f"Blockchain is invalid, proof of work is wrong!")
			if actual_prev_hash != recordered_prev_hash:
				print(f"Blockchain is invalid, expected {recordered_prev_hash}, actual {actual_prev_hash}")
			else:
				print(f"Valid hash {actual_prev_hash}")
		prev_block = block

def calcBalances():
	balances = {}
	for block in blockchain:
		if block["from"] in balances:
			balance_from = balances[block["from"]]
		else:
			balance_from = 0
	
		if block["to"] in balances:
			balance_to = balances[block["to"]]
		else:
			balance_to = 0

		balance_from -= block["amount"]
		balance_to -= block["amount"]
		
		balances[block["from"]] = balance_from
		balances[block["to"]] = balance_to
	print(balances)

```

