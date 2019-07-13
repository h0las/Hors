# Hors

**Hors** — это Библиотека **.NET Core** для распознавания даты и времени в естественной речи на русском языке. Умеет понимать сложные 
конструкции с абсолютным и относительным датой и временем. В том числе временные периоды.

## Установка
Через NuGet
```bash
Install-Package Hors -Version 0.9.1
```

## Примеры работы
(_Текущая дата: 13.07.2019 3:40_)
```bash
> 26 марта в 11 вечера будет красивый закат
>
> {0} будет красивый закат
> [Type=Fixed, DateFrom=26.03.2019 23:00:00, DateTo=26.03.2019 23:00:00, Span=00:00:00]
```
```bash
> скорее всего в сентябре 25 и 26 числа произойдёт что-то
>
> скорее всего {0} {1} произойдёт что-то
> [Type=Fixed, DateFrom=25.09.2019 0:00:00, DateTo=25.07.2019 23:59:59, Span=00:00:00]
> [Type=Fixed, DateFrom=26.09.2019 0:00:00, DateTo=26.07.2019 23:59:59, Span=00:00:00]
```
```bash
> я ходил гулять неделю 2 дня и час назад
>
> я ходил гулять {0}
> [Type=SpanBackward, DateFrom=04.07.2019 2:40:00, DateTo=04.07.2019 2:40:00, Span=-9.01:00:00]
```
```bash
> в следующий четверг с 9 утра до 6 вечера важный экзамен
>
> {0} важный экзамен
> [Type=Period, DateFrom=18.07.2019 9:00:00, DateTo=18.07.2019 18:00:00, Span=00:00:00]
```
```bash
> я был на даче в прошлые выходные
>
> я был на даче {0}
> [Type=Period, DateFrom=06.07.2019 0:00:00, DateTo=07.07.2019 23:59:59, Span=00:00:00]
```
```bash
> позавчера в 6 30 состоялось совещание а завтра днём будет хорошая погода
>
> {0} состоялось совещание а {1} будет хорошая погода
> [Type=Fixed, DateFrom=11.07.2019 6:30:00, DateTo=11.07.2019 6:30:00, Span=00:00:00]
> [Type=Fixed, DateFrom=14.07.2019 12:00:00, DateTo=14.07.2019 12:00:00, Span=00:00:00]
```

## Использование
- На вход можно подать строку или `IEnumerable<string>`, если строка уже разбита на токены.
- Все числа должны быть числами, а не прописью
- Вторым аргументом подаётся дата пользователя, относительно которой нужно измерять промежутки (например "день назад")

```CSharp
using Hors;

var hors = new HorsTextParser();
var result = hors.Parse("26 марта в 11 вечера будет красивый закат", DateTime.Now);

Console.WriteLine(result.Text); // -> {0} будет красивый закат
```

Метод `Parse` возвращает объект [HorsParseResult](https://github.com/DenisNP/Hors/blob/master/Models/HorsParseResult.cs)

### HorsParseResult
Поле | Тип | Описание
-- | -- | --
`Text` | _string_ | Исходная строка, где вместо всех распознанных участков даты и времени вставлен токен `{x}`, в котором `x` — номер индекса токена в массиве `Dates`
`Dates` | _List<DateTimeToken>_ | Список распознанных дат и времён, см. ниже

### DateTimeToken
Поле | Тип | Описание
-- | -- | --
`Type` | _Enum_ | `Fixed` — фиксированная дата, обозначает какой-то конкретный день или момент внутри дня; `Period` — промежуток; `SpanForward` — обозначение периода времени со сдвигом вперед, например "через 1 час"; `SpanBackward` — период времени со сдвигом в прошлое, например "5 часов назад"
`DateFrom` | _DateTime_ | Дата начала периода. Для фиксированного момента дата этого момента.
`DateTo` | _DateTime_ | Дата конца периода. Если фиксированный момент — целый день, то `DateFrom` и `DateTo` будут, соответственно, началом и концом дня. Если зафиксирован конкретный час, то они будут равны.
`Span` | _TimeSpan_ | Величина периода смещения для `Type = SpanForward` и `Type = SpanBackward`. При этом в `DateFrom` лежит смещение относительно текущей даты пользователя.
