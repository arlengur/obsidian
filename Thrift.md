Apache Thrift бинарный протокол
- contract-first - на псевдоязыке пишется контракт из которого генерируется код для различных платформ
- Кроссплатформенный
- Обратно совместимый

```trift
struct AdvBreed {
	1: string id,
	2: string name,
	3: optional string title
}
```