# FieldFormatter

FieldFormatter - это протокол, который позволяет упростить создание формы для отображения информации по одной сущности.

Допустим, вы получаете от API объект с некоторым набором свойств (полей).
Каждое поле во входных данных должно быть преобразовано к объекту Field, описывающему это поле и содержащему отформатированные данные.

Вот описание класса Field:

```js
export class Field {
  key: string;
  itemMode?: FieldItemMode;
  title: string;
  kind: FieldKind;
  hint: string;
  value: FieldValue;
}
```

Описание полей:

| Название | Описание                                                                   |
| -------- | -------------------------------------------------------------------------- |
| key      | Ключ, по которому происходит обращение к полю. Название поля               |
| itemMode | Приоритет поля при отображении в виде элемента списка или карточки         |
| title    | Название поля в человекочитаемом виде, в котором оно может быть отображено |
| type     | Тип поля                                                                   |
| hint     | Описание поля, которое может выступать в качестве подсказки                |
| value    | Отформатированное значение                                                 |

## Тип FieldItemMode

Описание типа `FieldItemMode`:

```js
export enum FieldItemMode {
  FirstLine = "firstLine",
  SecondLine = "secondLine",
  Picture = "picture",
  Hidden = "hidden",
}
```

Поля, для которых указан приоритет FirstLine должны отображаться на первой строке в элементе списка.

Поля, для которых указан приоритет SecondLine должны отображаться на второй строке в элементе списка.

Поля, для которых указан приоритет Hidden не должны отображаться в короткой форме. По умолчанию, если не указано ничего, то предполагается, что это поле типа hidden.

Поля, для которых указан приоритет Picture должны отображаться в виде картинке слева в элементе списка.

Одни и те же поля могут быть представлены в разные объекты типа `Field`. Например, если во входных данных указаны фамилия, имя и отчество в разных полях, то на карточке имеет смысл отобразить их в одном поле. Для этого нужно добавить отдельное поле с типом itemMode=FirstLine и со значением равным конкатенации этих входных полей. Впрочем, тоже самое можно сделать и в случае обычного отображения.

Протокол не предполагает двойной и более вложенности. Если такая есть, то нужно указать ссылку на соответствующую сущность.

## Тип FieldKind

Упрощение, которое явялется целью протокола, заключается в том, что тип поля `FieldKind` - это перечислимый тип. Таким образом, вместо того, чтобы писать отображение кучи пришедших от сервера данных, вы пишете отображение для каждого значения этого типа и функцию рендеринга, в которой в цикле выбираете как отображать текущее поле в зависимости от поля `kind`. А если учесть, что зачастую нужно разрабатывать на несколько платформ, то это позволит "помножить" эффективность на количество этих платформ.

Описание типа `FieldKind`:

```js
export enum FieldType {
  Text = "text",
  Number = "number",
  List = "list",
  Object = "object", // converted to fields object
  Date = "date",
  DateTime = "dateTime",
  Email = "email",
  Paragraphs = "paragraphs",
  ObjectList = "objectList", // list of objects
  MediaList = "mediaList",
  AttachmentList = "attachmentList",
}
```

Для каждого типа данных `FieldKind` допустимо значение определенного типа в поле `value`:

| Тип поля       | Ожидаемый тип значения поля value |
| -------------- | --------------------------------- |
| Text           | string                            |
| Number         | number                            |
| List           | string[]                          |
| Object         | Field[]                           |
| Date           | string                            |
| DateTime       | string                            |
| Price          | string                            |
| Email          | string                            |
| Paragraphs     | string[]                          |
| ObjectList     | Field[]\[]                        |
| MediaList      | string[]                          |
| AttachmentList | string[]                          |

Значения, пришедшие от сервера должны быть преобразованы в значения соответствующего типа.

## Скалярные поля

Есть несколько скалярных типов:

- Text
- Number
- Date
- DateTime
- Price
- Email

Все они преобразуются в строки, за исключением типа `Number`.

Поля типа `Date`, `DateTime` и `Price` преобразуются в строки в соответствии с локализацией.

При форматировании поля `Price` необходимо учесть порядок расположения знака валюты, характеный для той или иной локали.

При форматировании поля `Date` необходимо отбросить время и отображать дату.

## Списковые поля

Есть несколько списковых типов:

- List
- Paragrathes
- Object
- ObjectList
- MediaList
- AttachmentList

List - это простой список, элементами которого являются строки. В HTML такой список может непосредсвенно быть отображен с помощью тегов `ul` , `li`.

Paragraphes - список абзацев. В HTML элементы этого списка должны быть обернуты в таг `p`.

Object - список полей вложенного объекта. При реализации отображения данного поля уместен **рекурсивный вызов функции рендеринга**.

ObjectList - список вложенных объектов. При реализации отображения элементов списка можно упростить их вид до списка с подстрочным текстом или списком "карточек". Возможно, так же, потребуется фолдинг.

MediaList - список медиа файлов (картинки).

AttachmentList - список файлов, не имеющих визуального представления (документы).
