# Физическое время

Instant - момент времени на timeline
Duration - продолжительность события между Instant (в секундах)

Примеры:
- Instant - Instant = Duration
- Duration + Duration = Duration
- Instant + Duration = Instant
- Duration * number = Duration

# Гражданское время

- Year
- month 1-12
- day of month 1-31
- hour 0-23
- minute 0-59
- second 0-59

Datetime - (дата и время точка на календаре)
Period (2 month 1 week 3 days) - время между точками на календаре

Примеры:
- Datetime - Datetime = Period
- Period + Period = Period
- Period + Datetime = Datetime
- Period * number = Period

Классы: LocalDateTime, LocalDate, LocalTime

TemporalAmount реализуют Duration и Period
# Time zone 
преобразует Instant в Datetime и наоборот
UTC offset = 00:00

Классы: OffsetDateTime и OffsetTime
# ZonedDateTime 
включает Физическое и Гражданское время

Хранит все выше перечисленные классы

Примеры:
```scala
Instant.now() // время в UTC
Instant.now().atZone(ZoneId.of("Europe/Moscow")) // преобразовали во время в Москве

LocalDateTime.now() // текущее время
LocalDateTime.now().atZone(ZoneId.of("Europe/Moscow")).toInstant // преобразовали в UTC
LocalDateTime.now().atZone(ZoneId.of("UTC")).toInstant // сохранили текущее время как UTC

Duration.ofDays(1) // PT24H
Period.ofDays(1) // P1D
```