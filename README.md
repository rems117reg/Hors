# Hors

**Hors** — это Библиотека **.NET Core** для распознавания даты и времени в естественной речи на русском языке. Умеет понимать сложные 
конструкции с абсолютными и относительными датой и временем. В том числе временные периоды. [Хорс](https://ru.wikipedia.org/wiki/%D0%A5%D0%BE%D1%80%D1%81) — это славянский бог солнца. 

> Библиотека Hors лежит в основе работы [голосового навыка Мой Секретарь](https://dialogs.yandex.ru/store/skills/75bc5222-moj-sekretar) в Яндекс.Алисе.

## Установка
Через [NuGet](https://www.nuget.org/packages/Hors)
```bash
Install-Package Hors -Version 0.9.24
```

## Примеры работы
(_Текущая дата в примерах: 14.07.2019 3:40_)
```bash
> 26 марта в 11 вечера будет красивый закат.
>
> {0} будет красивый закат.
> [Type=Fixed, DateFrom=26.03.2019 23:00:00, StartIndex=0, EndIndex=20]
```
```bash
> Будет ли в сентябре 25 и 26 числа поход?
>
> Будет ли {0} {1} поход?
> [Type=Fixed, DateFrom=25.09.2019 0:00:00, DateTo=25.09.2019 23:59:59, StartIndex=9, EndIndex=33]
> [Type=Fixed, DateFrom=26.09.2019 0:00:00, DateTo=26.09.2019 23:59:59, StartIndex=9, EndIndex=33]
```
```bash
> Я был в гостях неделю 2 дня и час назад.
>
> Я был в гостях {0}.
> [Type=SpanBackward, DateFrom=05.07.2019 3:40:00, Span=-9.01:00:00, StartIndex=15, EndIndex=39]
```
```bash
> В следующий четверг с 9 утра до 6 вечера важный экзамен!
>
> {0} важный экзамен!
> [Type=Period, DateFrom=18.07.2019 9:00:00, DateTo=18.07.2019 18:00:00, StartIndex=0, EndIndex=40]
```
```bash
> Я был на даче в прошлые выходные
>
> Я был на даче {0}
> [Type=Period, DateFrom=06.07.2019 0:00:00, DateTo=07.07.2019 23:59:59, StartIndex=14, EndIndex=32]
```
```bash
> Позавчера в 6:30 состоялось совещание, а завтра днём будет хорошая погода.
>
> {0} состоялось совещание, а {1} будет хорошая погода.
> [Type=Fixed, DateFrom=12.07.2019 6:30:00, StartIndex=0, EndIndex=16]
> [Type=Fixed, DateFrom=15.07.2019 12:00:00, StartIndex=41, EndIndex=52]
```

## Использование
- На вход можно подать строку или `IEnumerable<string>`, если строка уже разбита на токены.
- Все числа должны быть числами, а не прописью.
- Вторым аргументом подаётся дата пользователя, относительно которой нужно измерять промежутки (например если пользователь скажет "день назад"). В большинстве ситуаций логично подавать просто текущую дату, но могут быть исключения.
- Третьим аргументом передаётся `int collapseDistance` — максимальное количество слов между двумя датами, при котором Hors будет пытаться объединить эта даты в одну, если возможно. Например: _завтра втреча с другом в 12_ превратится в _{завтра в 12} встреча с другом_. По-умолчанию `3`.

```CSharp
using Hors;

var hors = new HorsTextParser();
var result = hors.Parse("26 марта в 11 вечера будет красивый закат", DateTime.Now);

var text = result.Text; // -> string: "{0} будет красивый закат"
var date = result.Dates[0].DateFrom; // -> DateTime: 26.03.2019 23:00:00
```

Метод `Parse` возвращает объект [HorsParseResult](https://github.com/DenisNP/Hors/blob/master/Models/HorsParseResult.cs)

### HorsParseResult
Поле | Тип | Описание
-- | -- | --
`SourceText` | _string_ | Исходная строка без изменений
`Text` | _string_ | Исходная строка с полной пунктуацией, из которой убраны все токены, отвечающие за распознанные дату-время
`Tokens` | _List\<string\>_ | Текст, разбитый на токены с убранными знаками препинания. Вместо дат и времени будут токены `{x}`, где `x` — номер индекса токена в массиве `Dates`
`Dates` | _List\<DateTimeToken\>_ | Список распознанных дат и времён, см. ниже
`TextWithTokens` | _string_ | Исходная строка, где вместо всех распознанных участков даты и времени вставлен токен `{x}`, в котором `x` — номер индекса токена в массиве `Dates`
`CleanTextWithTokens` | _string_ | `TextWithTokens` после токенизации с убранными знаками препинания

### DateTimeToken
Поле | Тип | Описание
-- | -- | --
`Type` | _Enum_ | `Fixed` — фиксированная дата, обозначает какой-то конкретный день или момент внутри дня; `Period` — промежуток; `SpanForward` — обозначение периода времени со сдвигом вперед, например "через 1 час"; `SpanBackward` — период времени со сдвигом в прошлое, например "5 часов назад"
`DateFrom` | _DateTime_ | Дата начала периода. Для фиксированного момента дата этого момента.
`DateTo` | _DateTime_ | Дата конца периода. Если фиксированный момент — целый день, то `DateFrom` и `DateTo` будут, соответственно, началом и концом дня. Если зафиксирован конкретный час, то они будут равны.
`Span` | _TimeSpan_ | Величина периода смещения для `Type = SpanForward` и `Type = SpanBackward`. При этом в `DateFrom` лежит дата с этим смещением относительно даты пользователя.
`HasTime` | _bool_ | Показывает, было ли для этого токена установлено время в процессе ввода дат, или только день
`StartIndex` | _int_ | Индекс начала подстроки с данной датой в исходной строке, с нуля
`EndIndex` | _int_ | Индекс конца подстроки с данной датой в исходной строке, **не включительно**
