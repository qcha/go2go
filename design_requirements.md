# Требования к оформлению

Абсолютно запрещено употребление ругательств, мат.
Запрещены упоминания политических тем, а также оскорбления человека и животных по любым признакам.

## Название файла

Название файла, содержащего заметку, пишется на английском языке. В имени должна быть информация о том, что это за заметка и про какую тему будет идти речь.

Если в названии более одного слова, то вместо пробела слова связываются символом нижнего подчеркивания `_`.

Например:

`table_of_contents.md`

Заметки логически группируются по директориям.
Принцип группирования тот же, что и в группировке классов по пакетам.

Сначала выбираете область, к которой относится ваша статья, например, `web` или `go_core`.

Памятка:

| Директория        | Описание                                                 |
|-------------------|----------------------------------------------------------|
| algorithms        | Алгоритмы, структуры данных                              |
| build             | Все, что касается сборки и ci/cd                         |
| db                | Секция баз данных                                        |
| frameworks        | Фреймворки                                               |
| assets            | Хранилище картинок и ресурсов для статей                 |
| interview         | Задачи с собеседований                                   |
| go_core           | Основные темы Go                                         |
| logging           | Логгирование                                             |
| patterns          | Паттерны: архитектурные, программирования (GoF), проекта |
| testing           | Секция про тестирование                                  |
| web               | Веб и все, что с ним связано                             |
| other             | Все, что не вошло в другие разделы                       |

Обращайте внимание на подпапки в директориях: это поможет детальнее подойти к выбору места для заметки.

## Оформление заметки

Шаблон для оформления находится здесь: [Pull Request Template](./PULL_REQUEST_TEMPLATE.md)

Ниже даны пояснения по разделам.

### Заголовок

Каждая заметка должна иметь заголовок, оформленный в виде `h1`.
Заголовок должен коротко передавать суть заметки и содержать не более 3-4 слов.

Крайне нежелательно в заголовок выносить какие-то цифры, менять регистр как вздумается и использование табуляции.

После `h1` заголовка статьи следует блок `Введение`, чья задача - это ввести читателя в рассматриваемую тему или проблему. Либо добавить деталей о том, что за материал будет дальше изложен.

До введения следует расположить блок `ToC`: Table of Contents. Это блок оглавления и навигации.

### Основная часть

После заголовка начинается основная часть статьи, каждый логический блок выделяется через `h2`-заголовок.

Каждый блок может также дробиться, выделяя новые блоки через `h3` и так далее.

Каждый блок отделется переносом строки до и после.

Значимые и ключевые слова выделяются с помощью ``.

Желательно избегать сокращений, либо в начале явно читателю показать, что вы сокращаете некоторое имя и далее будет использоваться сокращенная форма.

Если рассматриваемый материал пересекается или ссылается на отдельную заметку будет не лишним дать на неё ссылку.

Необходимо делать скидку на то, что уровень читателя может разниться. Поэтому избегайте переосложнений и сленга.

Старайтесь давать примеры кода и оформлять их в виде:

\```java
// some code
\```.

### Заключение

В качестве заключения пишется краткий обзор рассмотренного материала, с выделением наиболее занчимых мест.

### Полезные ссылки

После заключения обязательно необходимо указать блок полезных ссылок, которые помогут либо в дальнейшем, более глубоком изучении материала, либо были использованы для основы статьи.

### Отдельная благодарность

В самом конце можно выделить блок для вынесения благодарностей для тех, кто помог в создании статьи: делал ревью, консультировал, предлагал материалы и т.д.

### Изображения

Изображения хранятся в директории `assets` репозитория. Под каждую статью выделяется своя директория. Структура должна повторять то, как расположена статья.

Например, если ваша статья - это `go_core/oop/interface.md`, то изображения для нее должны располагаться в директории `images/go_core/oop/interface`.

Изображения, используемые в заметках и статьях, хранятся в формате `png` или `jpg`.

Я использую [draw.io](https://www.draw.io/) в качестве редактора.

### В качестве примера

---

### Шаблон

```text
# Название

## Введение

## Основная часть

## Заключение

## Полезные ссылки

## Отдельная благодарность
```

## Оформление interview заметки

Это заметки, описывюащие вопросы и [задачи с интервью](./interview/).

Раздел interview состоит из:

| Директория        | Описание                                                 |
|-------------------|----------------------------------------------------------|
| algorithms        | Задачи на алгоритмы и leetcode                           |
| code_review       | Задачи на код ревью                                      |
| live_coding       | Задачи на написание кода                                 |
| questions         | Вопросы с собеседований                                  |

Вопросы и задачи в каждом разделе делятся по сложности.

| Сложность         | Описание                                                                                              |
|-------------------|-------------------------------------------------------------------------------------------------------|
| beginner          | Задачи и вопросы для самых начинающих, стажеров                                                       |
| easy              | Задачи и вопросы не требующие глубоких знаний, базовый уровень                                        |
| medium            | Задачи и вопросы, требующие более глубокого обдумывания, проработки краевых случаев                   |
| hard              | Сложные и нестандартные задачи и вопросы                                                              |

В каждом разделе есть intro.md файл с описанием раздела и списком задач, там же описывается принцип разделения по сложности.

### Шаблон interview задачи

Шаблон:

```text
# Заголовок

## Условие

### Примеры

## Решение

### Асимптотика решения (если это задача на алгоритмы)

Время: O(_)

Память: O(_)

```

Не забудьте добавить заметку в intro.md раздела!
