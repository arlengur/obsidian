- Нет системы сборки мусора

Видео
https://www.youtube.com/watch?v=XDv4I3_4Ubs&list=PL4_hYwCyhAvbeLzi699gqMUA4UaPkcdmJ
https://youtu.be/73ORmrnZtmQ?list=PLgG7lPwNdp556iIin-9eaJLlu7HL6YFv0

Док
https://doc.rust-lang.ru/book/ch02-00-guessing-game-tutorial.html

https://www.rust-lang.org/

Можно добавить его в переменные среды это .profile
`. "$HOME/.cargo/env"`
Проверить установку: `rustc --version` или `cargo --version`

Ex: main.rs
```rust
fn main() {
	println!("Hello"); // println! - макрос
}
```
Скомпилировать `rustc main.rs`
Запустить `./main`
Или можно скомпилировать и запустить командой `cargo run`
Можно собрать оптимизированный файл и без отладочной инорфмации командой `cargo build --release`

Чтобы запускать из Idea или VsCode установить плагин Rust и Toml

Файл `Cargo.toml` хранит зависимости проекта.

Чтобы создать проект из консоли `cargo new app`

## Комментарии
```rust
// Однострочный
/* Многострочный */
```

# Типы данных

## Integer 

| 8   | 16  | 32  | 64  | 128  | 32/64 | Кол-во бит                  |
| --- | --- | --- | --- | ---- | ----- | --------------------------- |
| i8  | i16 | i32 | i64 | i128 | isize | со знаком                   |
| u8  | u16 | u32 | u64 | u128 | usize | без знака (неотрицательные) |
isize - создать переменную с количеством бит соответствующим разрядности ОС.

Переменные
```rust
let age1 = 21; // неизменяемая
let mut age2 = 21; // изменяемая
let idx: usize = 1;  
let y = 92_000i64;  
let hex_octal_bin = 0xfff_fff + 0o777 + 0b1;  
println!("{idx} {y} {hex_octal_bin}")
```
По большому счету let это паттерн матчинг
## Float 

Всего 2 типа: `f32` и `f64`

Ex. 
```rust
let age: f64 = 879.567;
```

## String

Строки хранятся в куче (heap) в UTF8.

Безопасный способ 
```rust
let name1 = String::from("Arlen"); // 1 way
let name2 = "Arlen".to_string(); // 2 way
println!("{}", name1);
```

Небезопасный способ 
```rust
let name = "Arlen";
let name: &str = "Arlen";
```

Добавление к строке
```rust
let mut name2 = "Arlen".to_string();  
name2.push_str(" gur"); // add string
name2.push('R'); // add char
```

Конкатенация
```rust
let s1 = "hello".to_string();  
let s2 = " world".to_string();  
let res = s1 + &s2; // 1 way: отдали s1 во владение и добавили к ней строковый литерал
let res = format!("{s1} {s2}"); // 2 way: format не берет переменные во владение
println!("{res}");
```

Чтение элементов
```rust
s1.len() // выводит количество байт в строке (для кириллицы 2 байта на символ)

for b in "ЛОЛ".bytes() {  
    println!("{b}")  
}

for ch in "ЛОЛ".chars() {  
    println!("{ch}")  
}
```

Функции
```rust
"Let's go for a walk".split_whitespace() // разбить строку на слова
```

## Char

```rust
let symbol = 'A';  
println!("{}", symbol);
```

## Bool
Занимает 1 байт
```rust
let logic = true;  
println!("{}", logic);
```

# Константы
Можно объявить не функции main, не может принимать значение вычисления функции.
```rust
const NUM: i8 = 10;
```
# Арифметика
```rust
// + - * / %

// / % округляют до целых

// ! << >> | & битовые операции

/* функции:
 (-1).abs()
 0b0011010u8.count_ones() 2i32.pow(10)
 "".trim().parse().unwrap()
 */
```

## Нет неявного преобразования типов
```rust
let x: u16 = 1;  
let y: u32 = x; // mismatched types  
let y: u32 = x.into(); // надо явно привести тип
let z: u16 = y as u16; // as оператр явного преобразования

let x = i32::MAX;

let x = i32::MAX;  
let y = x.wrapping_add(1);  
assert_eq!(y, i32::MIN);  
  
let y = x.saturating_add(1);  
assert_eq!(y, i32::MAX);  
  
let (y, overflowed) = x.overflowing_add(1);  
assert!(overflowed);  
assert_eq!(y, i32::MIN);  
  
match x.checked_add(1) {  
    Some(y) => unreachable!(),  
    None => println!("overflowed")  
}

let not_a_number = std::f32::NAN;  
let inf = std::f32::INFINITY;  
8.5f32.ceil().sin().round().sqrt();
```
# Условные операторы
```rust
// > < >= <= == !=
// || и && ленивые

let name = String::from("Arlen");  
if name == "Arlen" {  
    println!("Hello, {}", name);  
} else if name == "John" {  
    println!("Hello, John!");  
} else {  
    println!("Hello, unknown!");  
}

let is_name = true;  
let res = if is_name {  
    "name"   
} else {  
    "Unknown"  
};  
println!("Hello, {}", res);
```

# Циклы

## loop
```rust
let mut i = 0;  
loop {  
    if i == 100 {  
        break;  
    }  
    println!("{}", i);  
    i += 1;  
}
```

## while
```rust
let mut i = 0;  
while i <= 100 {  
    println!("{}", i);  
    i += 1;  
}
```

## for
```rust
for i in 0..101 {  
    println!("{}", i);  
}
```

# match
```rust
let num = 1;  
match num {  
    0 => println!("{}", 0),  
    1..=50 => { // [1,50]
        println!("{}", num);  
    },  
    _ => println!("default {}", num)  
}
```

# Tuple
```rust
let pair = (1.0, 2);  
let left = pair.0; // shadowing
let right = pair.1;
```

# Unit
```rust
let void_res = println!("Hi");  
assert_eq!(void_res, ())
```

```rust
let trailing_comma = (  
    "Hi",  
    "World", // запятую можно не писать
    );
```

```rust
let x = 10;  
for i in 0..5 {  
    if x == 10 {  
        let x = 12;  
        println!("{i}");  
    }  
}
// 0 1 2 3 4
```

# Read input
```rust
use std::io;  
  
fn main() {  
    let mut name = String::new();  
    match io::stdin().read_line(&mut name) {  
        Ok(_) => println!("Hello, {name}"),  
        Err(e) => println!("Error {e}")  
    }  
}
```

# Массивы
```rust
let arr = [1,2,3,4];  
let arr: [i32; 4] = [1,2,3,4];
let arr = [1; 4]; // [1, 1, 1, 1]

println!("{:?}", arr)

arr[0]

for i in arr.iter() {  
    println!("{:?}", i)  
}
arr.len()
```

# Функции

Функции могут возвращать кортеж

```rust
fn mul(a: i32, b: i32) -> i32 {  
    a * b // выражение
    return a * b; // возвращение инструкции
}
```

Выражение - Его значение возвращается в вызывающую конструкцию (когда в конце нет ";")
Инструкция - Ее значение НЕ возвращается в вызывающую конструкцию (когда в конце есть ";")

В python и go есть сборщики мусора.
В C и C++ нет сборщиков мусора.

Данные хранятся в стеке (фиксированный размер) и куче (динамический размер).
В стеке хранятся 
- целые числа
- Bool
- Числа с плавающей точкой
- Символьный тип
- Кортежи с выше перечисленными типами

У каждого значения есть владелец и только один (`let num = 5` значение 5 принадлежит владельцу num). Когда владелец выходит из области видимости, то владелец отбрасывается и память занимаемая значением высвобождается.

Данные, которые хранятся в стеке копируются
```rust
let a = 1;
let b = a;
```

Данные, которые хранятся в куче перемещаются
```rust
let str1 = String::from("Arlen");
let str2 = str1; // str1 становится недействительной
let str2 = str1.copy(); // скопировали значение, str1 действительная

```

При передаче значения, которое хранится в куче, в функцию, функция забирает владение этим значением, чего не происходит в случае передачи значения из стека.
```rust
let str = String::from("Arlen");  
print_str(str);  
println!("{str}"); // error: borrow of moved value
  
let num = 9;  
print_num(num);  
println!("{num}");  
  
  
fn print_str(s: String) {  
    println!("{s}")  
}  
  
fn print_num(num: isize) {  
    println!("{num}")  
}
```

Значение из кучи можно предать по ссылке, тогда передачи владения значением не происходит
```rust
let str = String::from("Arlen");  
print_str(&str); 
fn print_str(s: &String) {  // передача неизменяемой ссылки
    println!("{s}")  
}

let mut str2 = String::from("Arlen");  
print_str(&mut str);  
fn print_str(s: &mut String) {  // передача изменяемой ссылки
    s.push_str(" is here")  
}
```

Можно создать только одну изменяемую ссылку
Можно создать много неизменяемых ссылок.
Нельзя создать одновременно изменяемую и неизменяемую ссылки.

```rust
let mut str = String::from("Arlen");  
  
let x = true;  
  
if x == true {  
    let ref1 = &mut str;  
    println!("{ref1}")  
}  
let ref2 = &mut str;  
println!("{ref2}");
```

# Срез
```rust
let str = String::from("Text here");  
println!("{}", &str[1..4]);  
println!("{}", &str[0..4]);  
println!("{}", &str[..4]);  
println!("{}", &str[0..]);  
println!("{}", &str[..]);

let arr = [1, 2, 3, 4, 5];  
println!("{:?}", &arr[1..4]);  
println!("{:?}", &arr[..]);
```

# Структуры
```rust
#[derive(Debug)]
struct Person {  
    name: String,  
    surname: String,  
    age: i32,  
    balance: f64  
}  
  
fn main() {  
    let p = Person {  
        name: "Kate".to_string(),  
        surname: "Tomson".to_string(),  
        age: 25,  
        balance: 108.08,  
    };  
  
    println!("{:#?}", p)  
}

let age = 37;  
let balance = 3.45;  
let p1 = Person {  
    name: "Kate".to_string(),  
    surname: "Tomson".to_string(),  
    age,  
    balance,  
};  
let p2 = Person {  
    name: "Kate".to_string(),  
    surname: "Tomson".to_string(),  
    ..p1  
};

struct Str(i32, String, f64); // кортежная структура
let obj = Str(2, "Hi".to_string(), 3.5);
```

Реализация методов структуры
```rust
#[derive(Debug)]
struct Triangle {  
    cat1: f64,  
    cat2: f64,  
}  
// methods
impl Triangle {  
    fn find_hyp(&self) -> f64 {  // self - ссылка на объект
        (self.cat1 * self.cat1 + self.cat2 * self.cat2).sqrt()  
    }  
}  
  
let tr1 = Triangle {  
    cat1: 8.0,  
    cat2: 10.0  
};  
  
println!("{}", tr1.find_hyp())
```

Связанные функции - функции, которые не принимают self (пр. String::from)
```rust
impl Triangle {  
    fn create_isc(cat: f64) -> Triangle {  
        Triangle {  
            cat1: cat,  
            cat2: cat  
        }  
    }  
}
...
let isc_tr = Triangle::create_isc(5.0);
println!("{:#?}", isc_tr);
```

# Перечисления
```rust
enum  Person {  
    Adult,  
    Underage,  
}  
let p = Person::Adult;  
match p {  
    Person::Adult => println!("Go!"),  
    Person::Underage => println!("Prohibited!");  
}

enum  Say {  
    Hi(String),  
}  
let say = Say::Hi("Hi".to_string());  
match say {  
    Say::Hi(s) => println!("{s}")  
}

enum Say {  
    Num(i32),  
}  
impl Say {  
    fn say_hi(&self) {  
        match self {  
            &Say::Num(num) => println!("{num}"),  
        }  
    }  
}  
let num = Say::Num(14);  
num.say_hi()
```

# Векторы
Хранит данные одного типа

```rust
let list = vec![1,2,3];  

println!("{:?}", &list);
println!("{:?}", &list[0..2]); // [1, 2]
println!("{}", &list[0]); // 1
println!("{:?}", &list.get(0)); // Some(1)

match list.get(0) {  
    None => {println!("absent")}  
    Some(el) => {println!("found {}", el)}  
}
```

```rust
let mut list = vec![1,2,3];  
list.push(9); // [1, 2, 3, 9]  
list.remove(2); // [1, 2, 9]

for el in &list {  
    println!("{}", el)  
}

for el in list.iter() {  // iter принимает ссылку, поэтому явно указывать не нужно
    println!("{}", el)  
}
```

```rust
let mut l: Vec<f64> = Vec::new();
```

В векторе можно хранить разные типы данных
```rust
#[derive(Debug)]  
enum Types {  
    Int(i32),  
    Float(f64),  
    Bool(bool),  
    Text(String)  
}  
  
let list = vec![  
    Types::Int(2),  
    Types::Bool(true)  
];  
println!("{:?}", &list);

match &list[0] {  
    Types::Int(i_type) => {  
        println!("{}", i_type)  
    }  
    Types::Float(f_type) => {  
        println!("{}", f_type)  
    }  
    Types::Bool(logic) => {  
        println!("{}", logic)  
    }  
    Types::Text(text) => {  
        println!("{}", text)  
    }  
}
```

# HashMap

Элементы хранятся неупорядоченно.

```rust
use std::collections::HashMap;  
  
let mut map = HashMap::new();
let n1 = "Ann".to_string();
let n2 = "Lisa".to_string();
map.insert("Arlen".to_string(), 10);  
map.insert("Tom".to_string(), 5);  
map.insert("Jane".to_string(), 8);  
map.insert(n1, 8); // передали владение
map.insert(&n2, 8); // передали по ссылке, но тогда все надо передавать по ссылке тк тип данных будет <&String, i32>
  
println!("{:?}", map);

// получение элементов
map["Arlen"]; // 1 way

match map.get("Arlen") {  // 2 way
    None => {  
        println!("Not found")  
    }  
    Some(v) => {  
        println!("{v}") // 10
    }  
}

// перебор элементов
for (n, v) in map {  
    println!("{n} has {v}")  
}

map.entry("Arlen1".to_string()).or_insert(9); // или возвращает значение или записывает новое

// изменение HashMap
for s in "Let's go for a walk a".split_whitespace() {  
    let count = map.entry(s).or_insert(0);  
    *count += 1  
}  
println!("{:?}", map);
```

# Обработка ошибок

Ошибки
- устранимые (result)
- неустранимые (panic)

Необрабатываемые ошибки
```rust
panic!("Error"); // вызовет ошибку

let list = vec![1, 2, 3];  
list[100]; // вызовет ошибку index out of bounds
```

Обработка обрабатываемых ошибок
```rust
let f = File::open("lol.txt");  
let f = match f {  
    Ok(f) => f,  
    Err(e) => panic!("{e}")  
};
```

Обработка ошибок с match
```rust
use std::fs::File;  
use std::io::ErrorKind;  
let path = "lol.txt";  
let f = File::open(path);  
let f = match f {  
    Ok(f) => f,  
    Err(e) => match e.kind() {  
        ErrorKind::NotFound => match File::create(path) {  
            Ok(f) => f,  
            Err(e) => panic!("Error creating file"),  
        },  
        other => panic!("Error {other}"),  
    },  
};
```

Обработка ошибок с unwrap и expect
```rust
let f = File::open(path).unwrap(); // возвращает File или panic
let f = File::open(path).expect("Error opening file"); // возвращает File или panic с преданным текстом
```

Распространение ошибок
```rust
use std::fs::File;  
use std::io::{Error, Read};  
  
let path = "lol.txt";  
  
match read_f_data(path) {  
    Ok(f) => println!("File: {f}"),  
    Err(e) => panic!("Error: {e}"),  
}  
  
fn read_f_data(path: &str) -> Result<String, Error> {  
    let f = File::open(path);  
    let mut f = match f {  
        Ok(f) => f,  
        Err(e) => return Err(e),  
    };  
  
    let mut data = String::new();  
  
    match f.read_to_string(&mut data) {  
        Ok(_) => Ok(data),  
        Err(e) => Err(e),  
    }  
}
```

Оператор ? возвращает или рещультат или возвращает ошибку в вызывающую функцию
Используется только в тех функциях которые возвращают Result

```rust
use std::fs::File;  
use std::io::{Error, Read};  
  
let path = "lol.txt";  
  
match read_f_data(path) {  
    Ok(f) => println!("File: {f}"),  
    Err(e) => panic!("Error: {e}"),  
}  
  
fn read_f_data(path: &str) -> Result<String, Error> {  
    let mut f = File::open(path)?;  
  
    let mut data = String::new();  
  
    f.read_to_string(&mut data)?;  
    Ok(data)  
}

// еще короче
fn read_f_data(path: &str) -> Result<String, Error> {  
    let mut data = String::new();  
    File::open(path)?.read_to_string(&mut data)?;  
    Ok(data)  
}
```

Файлы
```rust
use std::fs::File; 

let mut f1 = File::create("file").expect("file not found"); // write-only, открывает файл и стирает все что там было
let mut f2 = File::open("file").expect("Error opening file"); // read-only

f1.write_all("My message 3".as_bytes()).expect("Eror write to file"); // 1 way
f1.write_all(b"My message 3").expect("Eror write to file"); // 2 way

let mut file_data = String::new();  
f2.read_to_string(&mut file_data); // чтение из файла
println!("{file_data}");

use std::fs::OpenOptions;
let f3 = OpenOptions::new()  
    .read(true) // чтеине из файла
    .write(true) // запись в файл
    .create(true) // создать файл если не существует
    .open("file.txt")  
    .expect("error opening/creating fille");

use std::io::stdin;
let mut input = String::new();  
stdin().read_line(&mut input).expect("error getting user input"); // чтение из консоли

use std::fs::rename;
rename("old name", "new name").expect("eror renaming file"); // переимениование

remove_file("file").expect("error removing file"); // удаление
```

# Обобщения

Абстрактный заменитель типов данных

```rust
print_value("Hello");  
print_value(777);  
  
use std::fmt::Display;  
fn print_value<T: Display>(value: T) {  
    println!("{value}");  
}
```

Обобщения в структурах
```rust
struct Data<T, K> {  
    d1: T,  
    d2: K  
}  
impl<T, K> Data<T, K> {  
    fn get_data_1(&self) -> &T {  
        &self.d1  
    }  
    fn get_data_2(&self) -> &K {  
        &self.d2  
    }  
}  
  
let data = Data {  
    d1: 777,  
    d2: 2.5  
};  
  
println!("{} {}", data.d1, data.d2);  
println!("{} {}", data.get_data_1(), data.get_data_2());
```

Обобщения в перечислениях
```rust
enum Option<T> {  
    Some(T),  
    None  
}  
let opt = Some(23);

enum Result<T, E> {  
    Ok(T),  
    Err(E)  
}
```

Мономорфищзация - это когда rust заменяет обобщенный код на конкретный

Трейты или типажи для обычных типов
```rust
trait GetData {  
    fn get_value(&self) -> String;  
}  
impl GetData for i32 {  
    fn get_value(&self) -> String {  
        format!("Data is {}", *self)  
    }  
}  
  
let num = 12;  
println!("{}", num.get_value())
```

Трейты для структур
```rust
struct Message {  
    author: String,  
    text: String,  
}  
trait GetInfo {  
    fn get_info(&self) -> String;  
    fn hide_info(&self) {  
        println!("Nothing here")  
    }  
}  
impl GetInfo for Message {  
    fn get_info(&self) -> String {  
        format!("Message author: {}, text: {}", self.author, self.text)  
    }  
}  
let msg = Message {  
    author: "Arlen".to_string(),  
    text: "content".to_string()  
};  
println!("{}", msg.get_info());  
msg.hide_info();
```

Трейты как параметры функции
```rust
trait PringtLOL {  
    fn print_lol(&self) {  
        println!("LOL");  
    }  
}  
struct DDD {  
    num: i32  
}  
impl PringtLOL for DDD {}  
  
fn get_object(obj: impl PringtLOL) {  
    obj.print_lol();  
}  
  
let d = DDD {  
    num: 12  
};  
get_object(d);
```







# Tasks
- Что будет выведено?
```rust
let x = 10;  
for i in 0..5 {  
    if x == 10 {  
        println!("{i}");  
        let x = 12;  
    }  
}
```