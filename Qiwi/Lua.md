Lua (луна) — интерпритируемый язык программирования, свободно распространяемый, с открытым исходным кодом. Разработан в Бразилии. Язык с неявным динамическим определением типов данных.

Позволяет делать параллельное присваивание:
`a, b = b, a`

В Lua восемь основных типов:
- nil (неопределенный), бозначает отсутствие пригодного значения.
- boolean (логический)
- number (числовой), целый или вещественный
- string (строковый), массивы символов. Строки могут содержать любые 8-битные символы, включая ноль ('\0'). Строки неизменяемы. Строковые литералы могут записываться в одинарных или двойных кавычках, служебные символы помещаются в них в стандартной для C нотации с ведущим обратным слэшем. Многострочные литералы ограничиваются двумя подряд открывающимися и двумя подряд закрывающимися квадратными скобками.
- function (функция), допускается присваивание, передача функции в параметре и возврат функции как значения
- userdata (пользовательские данные, полученные на другом языке)
- thread (поток)
- table (таблица)

Online interpreter: https://onecompiler.com/lua

```lua
print('Hello, World!')

local a = 5 -- локальная переменная
b = 7 -- глобальная переменная
print(a)
```

Локальные переменные используются в одном скрипте, глобальные между скриптами

Две точки говорит, что соединяем строку
`print('Hello, World! ' .. 5)`

Сравнения и логические операторы
 `<, >, <=, >=, ==, ~= | !=, and, or, not`

Циклы
```lua
for i=1,10 do
  print(i)
end

local k = 5
while k > 0 do
 print(k)
 k = k - 1
end
```

Условные операторы
```lua
if a == 24 then 
	print(a)
else -- or elseif
	print(a .. ' != 24')
end
```

Таблицы
```lua
local tabledata = {'1str', '2str', 3, 4}
for r in pairs(tabledata) do
  print(r)
end

for r,inf in pairs(tabledata) do
  print(r) -- номер строки
  print(inf) -- значение в строке
end

local tabledata2 = {['tt'] = 'rr', '1str', '2str', 3, 4} --

for r,inf in pairs(tabledata2) do
  print(r .. ' ' .. inf)
end

local tabledata3 = {{1, 2}, '1str', '2str', 3, 4} -- вложенная таблица
```

Функции
```lua
function test()
 print('func')
end

test()
```

Встроенные функции
- tonumber - преобразует строку в число

Компилятор который собирает библиотеку Луа влияет на:
- работу стека
- организацию сборщика мусора
- распределение памяти

# Заметки
- печать таблицы в киви: `logger.info(jsondump(txn))`

# Скрипты
```lua
local res = list.name("pan").key(12).get()
local res = list.name({"20", "10", "pan"}).key(12).get()
local res = list.prefix("20:10").name("pan").key(12).get()

local res = list.prefix("").name("pan").key(12).get() 
local res = list.name({"pan"}).key(12).get() -- не находит

local uniq_item = unique_item.name("pan").of("12").get()
local uniq_item = unique_item.name({"20", "10", "pan"}).of("12").get()
local uniq_item = unique_item.prefix("20:10").name("pan").of("12").exist("goo")

local res = list.prefix("20:10").name("pqan").key(112).get()

-- сет
[
    {
        "name": "20:10:unique_recipients",
        "key": "boo",
        "value": "{\"goo\": true}"
    }
]


logger.info(jsondump(res))
return scoring.verdict.BLOCK
```

