title: Лабораторная работа №8: Веб-скрейпинг новостей сайта ITMO News

**ФИО**: Ян Ломаченко Станиславович

**ИСУ**: 505115

**Группа**: P3120

Ссылка на **[Google Colab](https://colab.research.google.com/drive/1x7JWkhTmFqPCo2_bu3tOPUoTt2_XjBEW?usp=sharing)**<br><br>


## 1. Цель работы

Целью работы было освоить базовые методы веб-скрейпинга данных из HTML-страниц на примере сайта новостей Университета ИТМО.

В рамках работы необходимо было получить данные с сайта:

https://news.itmo.ru/ru

и сохранить результаты в формате CSV.

---

## 2. Постановка задачи

По заданию требовалось:

1. Сделать копию исходного ноутбука в Google Colab.
2. Открыть доступ к ноутбуку.
3. В верхней ячейке указать фамилию, имя и группу.
4. Спарсить общий набор данных по новостям:
   - идентификатор новости;
   - название новости;
   - дату размещения;
   - URL на страницу конкретной новости.
5. Для каждой новости из датасета дополнительно спарсить:
   - идентификатор;
   - название новости;
   - дату размещения;
   - количество просмотров;
   - текст новости;
   - теги.
6. Сохранить общий CSV-файл рядом с ноутбуком.
7. Сохранить CSV-файл с подробным содержимым новостей в папку `news_content`.
8. Выполнить все ячейки ноутбука.
9. Сохранить и закрепить версию Colab.
10. Предоставить ссылку на готовый Colab-файл.

---

## 3. Используемые инструменты и библиотеки

Для выполнения работы использовались следующие библиотеки Python:

```python
import os
import re
import time
import random
import requests
import pandas as pd

from bs4 import BeautifulSoup
````

### Назначение библиотек

| Библиотека      | Назначение                                                  |
| --------------- | ----------------------------------------------------------- |
| `os`            | Работа с папками и путями к файлам                          |
| `re`            | Извлечение ID новости из URL с помощью регулярных выражений |
| `time`          | Создание задержек между запросами                           |
| `random`        | Генерация случайных пауз между запросами                    |
| `requests`      | Загрузка HTML-страниц сайта                                 |
| `pandas`        | Формирование таблиц и сохранение CSV-файлов                 |
| `BeautifulSoup` | Парсинг HTML-разметки страниц                               |

---

## 4. Общая логика решения

Вся работа была разделена на несколько этапов:

1. Сначала были получены списки заголовков новостей и ссылок на страницы новостей.
2. Затем для каждой ссылки выполнялся отдельный запрос к странице новости.
3. Из URL новости извлекался числовой идентификатор.
4. Из HTML-страницы новости извлекались:

   * дата публикации;
   * количество просмотров;
   * точный заголовок;
   * основной текст;
   * теги.
5. После сбора данных формировались два датасета:

   * общий датасет;
   * датасет с подробным содержимым новостей.
6. Оба датасета сохранялись в CSV-файлы.

---

## 5. Структура итоговых файлов

В результате выполнения кода создаются два CSV-файла.

### 5.1. Общий CSV-файл

Файл:

```text
general_news.csv
```

Содержит данные из пункта 3 задания:

| Колонка                                 | Описание                                |
| --------------------------------------- | --------------------------------------- |
| `Идентификатор новости`                 | Числовой ID новости, извлечённый из URL |
| `Название новости`                      | Название новости                        |
| `Дата её размещения`                    | Дата публикации новости                 |
| `URL на страницу с конкретной новостью` | Полная ссылка на страницу новости       |

---

### 5.2. CSV-файл с содержимым новостей

Файл:

```text
news_content/news_content.csv
```

Содержит данные из пункта 4 задания:

| Колонка                 | Описание                      |
| ----------------------- | ----------------------------- |
| `Идентификатор`         | Числовой ID новости           |
| `Название новости`      | Название новости              |
| `Дата размещения`       | Дата публикации               |
| `Количество просмотров` | Количество просмотров новости |
| `Текст`                 | Основной текст новости        |
| `Теги`                  | Теги новости                  |

---

## 6. Создание файлов и папок

В коде заранее задаются имена файлов и папки:

```python
GENERAL_CSV = 'general_news.csv'
CONTENT_DIR = 'news_content'
CONTENT_CSV = os.path.join(CONTENT_DIR, 'news_content.csv')
```

Затем создаётся папка для подробного CSV-файла:

```python
os.makedirs(CONTENT_DIR, exist_ok=True)
```

Параметр `exist_ok=True` нужен для того, чтобы код не выдавал ошибку, если папка `news_content` уже существует.

---

## 7. Извлечение идентификатора новости

Идентификатор новости находится в конце URL.

Пример URL:

```text
https://news.itmo.ru/ru/education/cooperation/news/14792/
```

Из него необходимо получить:

```text
14792
```

Для этого была написана функция:

```python
def get_news_id(url):
    """
    Достает числовой ID новости из URL.
    """
    match = re.search(r'/(\d+)/?$', url)

    if match:
        return int(match.group(1))

    return None
```

### Как работает регулярное выражение

```python
r'/(\d+)/?$'
```

| Часть выражения | Значение                                       |
| --------------- | ---------------------------------------------- |
| `/`             | Перед ID должен быть символ `/`                |
| `\d+`           | Одна или несколько цифр                        |
| `(...)`         | Сохранить найденные цифры в отдельную группу   |
| `/?`            | После числа может быть `/`, но он необязателен |
| `$`             | Конец строки                                   |

Таким образом, функция достаёт ID из URL и возвращает его как целое число.

---

## 8. Анализ HTML-структуры страницы новости

На этапе работы мы отдельно посмотрели HTML-разметку нескольких страниц новостей ITMO.

Было выявлено, что структура страницы новости выглядит примерно так:

```html
<article>
    <div class="news-info-wrapper">
        <time datetime="2026-04-06T14:27:02+03:00">
            6 Апреля 2026, 14:27
            <span class="timezone">UTC+3</span>
            <span class="icon eye">7250</span>
        </time>
    </div>

    <h1>Название новости</h1>

    <div class="content js-mediator-article">
        <p class="lead">
            <strong>Вводный абзац новости...</strong>
        </p>

        <div class="post-content">
            <p>Основной текст новости...</p>
            <p>Следующий абзац...</p>
        </div>
    </div>
</article>
```

Также теги новости находятся в ссылках вида:

```html
<a href="/ru/search/?tag=...">Поступление</a>
```

После анализа HTML стало понятно, какие точные CSS-селекторы нужно использовать.

---

## 9. Точные CSS-селекторы, использованные в работе

После анализа HTML были выбраны следующие селекторы:

| Данные                    | CSS-селектор                      |
| ------------------------- | --------------------------------- |
| Заголовок новости         | `article h1`                      |
| Дата новости              | `article time`                    |
| Количество просмотров     | `article time span.icon.eye`      |
| Основной контейнер текста | `div.content.js-mediator-article` |
| Лид новости               | `p.lead`                          |
| Основной текст            | `div.post-content`                |
| Теги                      | `a[href*="/ru/search/?tag="]`     |

Именно это позволило отказаться от неточного поиска через набор разных `possible_selectors`.

---

## 10. Извлечение даты новости

Сначала предполагалось, что дата находится в блоке:

```html
<div class="time">
```

Но анализ HTML показал, что такого блока на страницах нет.

Реальная дата находится внутри тега:

```html
<article>
    <time>
        6 Апреля 2026, 14:27
        <span class="timezone">UTC+3</span>
        <span class="icon eye">7250</span>
    </time>
</article>
```

Поэтому была написана функция:

```python
def get_news_date(soup):
    """
    Достает дату новости из article time, не удаляя просмотры из HTML.
    """
    time_tag = soup.select_one('article time')

    if time_tag:
        date_text = next(time_tag.stripped_strings, None)

        if date_text:
            return date_text

    return 'Дата не найдена'
```

### Важный момент

Ранее была версия функции, которая удаляла вложенные теги `span` из `time`.
Но количество просмотров тоже находится внутри `span`, поэтому при таком подходе просмотры удалялись из HTML до их извлечения.

Проблемная логика выглядела так:

```python
for inner_tag in time_tag.find_all(['span', 'a']):
    inner_tag.decompose()
```

После этого блок с просмотрами:

```html
<span class="icon eye">7250</span>
```

удалялся из дерева `soup`, и функция для просмотров уже не могла его найти.

Поэтому финальная версия функции `get_news_date()` ничего не удаляет, а просто берёт первый текстовый фрагмент внутри `time`.

---

## 11. Извлечение количества просмотров

Количество просмотров на сайте ITMO News находится не в отдельном блоке с текстом «просмотров», а внутри тега:

```html
<span class="icon eye">7250</span>
```

Например:

```html
<time datetime="2026-04-06T14:27:02+03:00">
    6 Апреля 2026, 14:27
    <span class="timezone">UTC+3</span>
    <span class="icon eye">7250</span>
</time>
```

Здесь число `7250` — это количество просмотров.

Для извлечения была написана функция:

```python
def get_views(soup):
    """
    Достает количество просмотров из:
    <span class="icon eye">7250</span>
    """
    views_tag = soup.select_one('article time span.icon.eye')

    if views_tag:
        views_text = views_tag.get_text(strip=True)

        if views_text.isdigit():
            return int(views_text)

    return None
```

Функция ищет элемент:

```python
article time span.icon.eye
```

Затем извлекает из него текст и преобразует его в целое число.

---

## 12. Извлечение текста новости

Основной текст новости находится в блоке:

```html
<div class="content js-mediator-article">
```

Внутри него есть:

1. Вводный абзац:

```html
<p class="lead">...</p>
```

2. Основной текст:

```html
<div class="post-content">
    <p>...</p>
    <ul>
        <li>...</li>
    </ul>
</div>
```

Для извлечения текста была написана функция:

```python
def get_text(soup):
    """
    Достает текст новости из div.content.js-mediator-article.
    """
    content = soup.select_one('div.content.js-mediator-article')

    if content is None:
        return ''

    for tag in content.select('div.poster, div.copyinfo, figure, figcaption, script, style'):
        tag.decompose()

    text_parts = []

    lead = content.select_one('p.lead')

    if lead:
        lead_text = lead.get_text(' ', strip=True)

        if lead_text:
            text_parts.append(lead_text)

    post_content = content.select_one('div.post-content')

    if post_content:
        for element in post_content.find_all(['h1', 'h2', 'h3', 'h4', 'p', 'li']):
            text = element.get_text(' ', strip=True)

            if not text:
                continue

            if 'Фото:' in text and len(text) < 150:
                continue

            if 'Источник:' in text and len(text) < 150:
                continue

            text_parts.append(text)

    return ' '.join(text_parts)
```

### Что делает эта функция

1. Находит основной контейнер новости.
2. Удаляет из него изображения, подписи к изображениям, скрипты и стили.
3. Извлекает лид новости.
4. Извлекает основной текст из блока `post-content`.
5. Берёт не только абзацы `<p>`, но и заголовки `<h1>`, `<h2>`, `<h3>`, `<h4>`, а также элементы списков `<li>`.
6. Убирает короткие подписи к фото и источникам.
7. Склеивает все части текста в одну строку.

---

## 13. Почему учитывалась вариативность текста

В задании было отдельно указано:

> Текст — не забыть про вариативность верстки.

На страницах ITMO News текст может быть оформлен по-разному:

* обычными абзацами `<p>`;
* заголовками `<h1>`, `<h2>`, `<h3>`, `<h4>`;
* списками `<ul>` и `<li>`;
* блоками цитат `<blockquote>`;
* внутренними блоками с `id`, например `link1`, `link2`, `link3`.

Поэтому код не ограничивается только тегом `<p>`, а собирает данные из разных текстовых элементов:

```python
post_content.find_all(['h1', 'h2', 'h3', 'h4', 'p', 'li'])
```

Это делает извлечение текста более полным.

---

## 14. Извлечение тегов новости

Теги на странице представлены ссылками на поиск по тегу.

Пример:

```html
<a href="/ru/search/?tag=Поступление">Поступление</a>
```

Для извлечения тегов была написана функция:

```python
def get_tags(soup):
    """
    Достает теги из ссылок /ru/search/?tag=...
    """
    tag_links = soup.select('a[href*="/ru/search/?tag="]')

    tags = []

    for link in tag_links:
        tag_text = link.get_text(' ', strip=True)

        if tag_text and tag_text not in tags:
            tags.append(tag_text)

    return ', '.join(tags)
```

Функция ищет все ссылки, у которых в `href` есть:

```text
/ru/search/?tag=
```

Затем достаёт текст каждой ссылки и сохраняет его в список.

Чтобы теги не повторялись, используется проверка:

```python
if tag_text and tag_text not in tags:
```

В итоговый CSV теги записываются одной строкой через запятую.

Пример:

```text
Поступление, День открытых дверей, Абитуриенты
```

---

## 15. Основной цикл сбора данных

После определения всех функций создаются два списка:

```python
news_data = []
content_data = []
```

`news_data` используется для общего датасета.

`content_data` используется для подробного датасета.

Основной цикл проходит по спискам `headers` и `urls`:

```python
for i, (header, url) in enumerate(zip(headers, urls)):
```

Здесь:

* `headers` — список названий новостей;
* `urls` — список ссылок на новости;
* `zip(headers, urls)` объединяет название и ссылку в пару;
* `enumerate(...)` добавляет номер текущей новости.

---

## 16. Начальные значения для каждой новости

Для каждой новости задаются значения по умолчанию:

```python
news_id = get_news_id(url)
news_date = 'Дата не найдена'
views = None
text = ''
tags = ''
```

Это нужно для того, чтобы даже в случае ошибки в итоговый датасет можно было добавить строку с частично заполненными данными.

---

## 17. Задержка между запросами

Перед запросом к каждой странице выполняется случайная пауза:

```python
time.sleep(random.uniform(0.5, 1.5))
```

Это означает, что код ждёт случайное количество секунд от `0.5` до `1.5`.

Такой подход используется, чтобы не отправлять слишком много запросов к сайту подряд.

---

## 18. Загрузка HTML-страницы

Для загрузки страницы новости используется:

```python
response = requests.get(
    url,
    headers={'User-Agent': 'Mozilla/5.0'},
    timeout=20
)
```

Здесь:

| Параметр                                | Назначение                               |
| --------------------------------------- | ---------------------------------------- |
| `url`                                   | Ссылка на страницу новости               |
| `headers={'User-Agent': 'Mozilla/5.0'}` | Имитация запроса от обычного браузера    |
| `timeout=20`                            | Максимальное ожидание ответа — 20 секунд |

Затем выполняется проверка ответа:

```python
response.raise_for_status()
```

Если сервер вернул ошибку, например `404` или `500`, программа перейдёт в блок `except`.

---

## 19. Парсинг HTML через BeautifulSoup

После успешного запроса HTML-код страницы передаётся в BeautifulSoup:

```python
soup_news = BeautifulSoup(response.text, 'html.parser')
```

После этого можно искать элементы страницы с помощью:

```python
select_one()
select()
find()
find_all()
```

В этой работе основным способом поиска были CSS-селекторы:

```python
soup.select_one('article time')
soup.select_one('article h1')
soup.select('a[href*="/ru/search/?tag="]')
```

---

## 20. Получение данных со страницы новости

Внутри цикла вызываются функции:

```python
news_date = get_news_date(soup_news)
```

Достаёт дату публикации.

```python
title_tag = soup_news.select_one('article h1')
```

Ищет точный заголовок новости на странице.

```python
views = get_views(soup_news)
```

Достаёт количество просмотров.

```python
text = get_text(soup_news)
```

Достаёт основной текст новости.

```python
tags = get_tags(soup_news)
```

Достаёт теги новости.

---

## 21. Уточнение заголовка новости

Изначально заголовок берётся из списка `headers`.

Но на самой странице новости он может быть точнее, поэтому код ищет заголовок внутри страницы:

```python
title_tag = soup_news.select_one('article h1')

if title_tag:
    page_title = title_tag.get_text(' ', strip=True)

    if page_title:
        header = page_title
```

Если заголовок найден, переменная `header` обновляется.

---

## 22. Обработка ошибок

Весь запрос и парсинг страницы находятся внутри блока:

```python
try:
    ...
except requests.exceptions.RequestException as e:
    ...
except Exception as e:
    ...
```

Если возникает ошибка при запросе к сайту, выполняется:

```python
except requests.exceptions.RequestException as e:
    print(f"Ошибка при запросе {url}: {e}")
```

Если возникает любая другая ошибка, например при парсинге, выполняется:

```python
except Exception as e:
    print(f"Ошибка при парсинге {url}: {e}")
```

Это важно, потому что одна проблемная страница не должна останавливать весь процесс сбора данных.

---

## 23. Формирование общего датасета

После обработки страницы в список `news_data` добавляется словарь:

```python
news_data.append({
    'Идентификатор новости': news_id,
    'Название новости': header,
    'Дата её размещения': news_date,
    'URL на страницу с конкретной новостью': url
})
```

Этот словарь соответствует одной строке будущего файла `general_news.csv`.

---

## 24. Формирование подробного датасета

В список `content_data` добавляется второй словарь:

```python
content_data.append({
    'Идентификатор': news_id,
    'Название новости': header,
    'Дата размещения': news_date,
    'Количество просмотров': views,
    'Текст': text,
    'Теги': tags
})
```

Этот словарь соответствует одной строке будущего файла:

```text
news_content/news_content.csv
```

---

## 25. Вывод прогресса

Каждые 10 новостей выводится сообщение:

```python
if (i + 1) % 10 == 0:
    print(f"Обработано {i + 1} новостей...")
```

Это удобно, потому что сбор данных может занимать некоторое время.

---

## 26. Преобразование данных в таблицы

После завершения цикла списки словарей преобразуются в таблицы `pandas`.

Общий датасет:

```python
general_df = pd.DataFrame(news_data)
```

Подробный датасет:

```python
content_df = pd.DataFrame(content_data)
```

---

## 27. Удаление дубликатов

Так как одна и та же новость может попасться несколько раз, из таблиц удаляются дубликаты.

Для общего датасета:

```python
general_df = general_df.drop_duplicates(subset=['Идентификатор новости'])
```

Для подробного датасета:

```python
content_df = content_df.drop_duplicates(subset=['Идентификатор'])
```

Дубликаты определяются по ID новости.

---

## 28. Сохранение общего CSV-файла

Общий датасет сохраняется командой:

```python
general_df.to_csv(GENERAL_CSV, index=False, encoding='utf-8-sig')
```

Параметры:

| Параметр               | Значение                                 |
| ---------------------- | ---------------------------------------- |
| `GENERAL_CSV`          | Имя файла `general_news.csv`             |
| `index=False`          | Не сохранять технический индекс pandas   |
| `encoding='utf-8-sig'` | Корректная кодировка для русского текста |

---

## 29. Сохранение подробного CSV-файла

Подробный датасет сохраняется командой:

```python
content_df.to_csv(CONTENT_CSV, index=False, encoding='utf-8-sig')
```

Файл сохраняется по пути:

```text
news_content/news_content.csv
```

Это соответствует требованию задания сохранить данные из пункта 4 в папке `news_content`.

---

## 30. Проверка результата

После сохранения файлов выводятся сообщения:

```python
print(f"Общий файл успешно записан: {GENERAL_CSV}")
print(f"Файл с содержимым новостей успешно записан: {CONTENT_CSV}")
```

Также отображаются первые строки обеих таблиц:

```python
display(general_df.head())
display(content_df.head())
```

Это позволяет быстро проверить, что данные собраны корректно.

---

## 31. Итоговая структура ноутбука

В итоговом Colab-ноутбуке должны быть следующие части:

1. Markdown-ячейка с фамилией, именем и группой.
2. Ячейка с импортом библиотек.
3. Ячейка с кодом для получения списков `headers` и `urls`.
4. Ячейка с функциями для парсинга страниц новостей.
5. Ячейка с основным циклом сбора данных.
6. Ячейка с сохранением CSV-файлов.
7. Ячейка с проверкой результата.

---

## 32. Итоговый код основной части

```python
import os
import re
import time
import random
import requests
import pandas as pd

from bs4 import BeautifulSoup


GENERAL_CSV = 'general_news.csv'
CONTENT_DIR = 'news_content'
CONTENT_CSV = os.path.join(CONTENT_DIR, 'news_content.csv')

os.makedirs(CONTENT_DIR, exist_ok=True)


def get_news_id(url):
    """
    Достает числовой ID новости из URL.
    """
    match = re.search(r'/(\d+)/?$', url)

    if match:
        return int(match.group(1))

    return None


def get_news_date(soup):
    """
    Достает дату новости из article time, не удаляя просмотры из HTML.
    """
    time_tag = soup.select_one('article time')

    if time_tag:
        date_text = next(time_tag.stripped_strings, None)

        if date_text:
            return date_text

    return 'Дата не найдена'


def get_views(soup):
    """
    Достает количество просмотров из:
    <span class="icon eye">7250</span>
    """
    views_tag = soup.select_one('article time span.icon.eye')

    if views_tag:
        views_text = views_tag.get_text(strip=True)

        if views_text.isdigit():
            return int(views_text)

    return None


def get_text(soup):
    """
    Достает текст новости из div.content.js-mediator-article.
    """
    content = soup.select_one('div.content.js-mediator-article')

    if content is None:
        return ''

    for tag in content.select('div.poster, div.copyinfo, figure, figcaption, script, style'):
        tag.decompose()

    text_parts = []

    lead = content.select_one('p.lead')

    if lead:
        lead_text = lead.get_text(' ', strip=True)

        if lead_text:
            text_parts.append(lead_text)

    post_content = content.select_one('div.post-content')

    if post_content:
        for element in post_content.find_all(['h1', 'h2', 'h3', 'h4', 'p', 'li']):
            text = element.get_text(' ', strip=True)

            if not text:
                continue

            if 'Фото:' in text and len(text) < 150:
                continue

            if 'Источник:' in text and len(text) < 150:
                continue

            text_parts.append(text)

    return ' '.join(text_parts)


def get_tags(soup):
    """
    Достает теги из ссылок /ru/search/?tag=...
    """
    tag_links = soup.select('a[href*="/ru/search/?tag="]')

    tags = []

    for link in tag_links:
        tag_text = link.get_text(' ', strip=True)

        if tag_text and tag_text not in tags:
            tags.append(tag_text)

    return ', '.join(tags)


news_data = []
content_data = []

print("Начинается сбор данных для каждой новости. Это может занять некоторое время...")

for i, (header, url) in enumerate(zip(headers, urls)):
    news_id = get_news_id(url)
    news_date = 'Дата не найдена'
    views = None
    text = ''
    tags = ''

    try:
        time.sleep(random.uniform(0.5, 1.5))

        response = requests.get(
            url,
            headers={'User-Agent': 'Mozilla/5.0'},
            timeout=20
        )

        response.raise_for_status()

        soup_news = BeautifulSoup(response.text, 'html.parser')

        news_date = get_news_date(soup_news)

        title_tag = soup_news.select_one('article h1')

        if title_tag:
            page_title = title_tag.get_text(' ', strip=True)

            if page_title:
                header = page_title

        views = get_views(soup_news)
        text = get_text(soup_news)
        tags = get_tags(soup_news)

    except requests.exceptions.RequestException as e:
        print(f"Ошибка при запросе {url}: {e}")

    except Exception as e:
        print(f"Ошибка при парсинге {url}: {e}")

    news_data.append({
        'Идентификатор новости': news_id,
        'Название новости': header,
        'Дата её размещения': news_date,
        'URL на страницу с конкретной новостью': url
    })

    content_data.append({
        'Идентификатор': news_id,
        'Название новости': header,
        'Дата размещения': news_date,
        'Количество просмотров': views,
        'Текст': text,
        'Теги': tags
    })

    if (i + 1) % 10 == 0:
        print(f"Обработано {i + 1} новостей...")


print("Сбор данных завершен. Начинается запись в CSV.")

general_df = pd.DataFrame(news_data)
general_df = general_df.drop_duplicates(subset=['Идентификатор новости'])

content_df = pd.DataFrame(content_data)
content_df = content_df.drop_duplicates(subset=['Идентификатор'])

general_df.to_csv(GENERAL_CSV, index=False, encoding='utf-8-sig')
content_df.to_csv(CONTENT_CSV, index=False, encoding='utf-8-sig')

print(f"Общий файл успешно записан: {GENERAL_CSV}")
print(f"Файл с содержимым новостей успешно записан: {CONTENT_CSV}")

print("\nПервые строки общего датасета:")
display(general_df.head())

print("\nПервые строки датасета с контентом:")
display(content_df.head())
```

---

## 33. Полученные результаты

В результате выполнения работы были получены два файла.

### Общий файл

```text
general_news.csv
```

Он содержит базовую информацию о новостях:

```text
ID новости
Название
Дата размещения
URL
```

### Подробный файл

```text
news_content/news_content.csv
```

Он содержит расширенную информацию:

```text
ID новости
Название
Дата размещения
Количество просмотров
Текст
Теги
```

---

## 34. Соответствие требованиям задания

| Требование                                        | Статус                     |
| ------------------------------------------------- | -------------------------- |
| Сделана копия ноутбука в Google Colab             | Выполнено                  |
| В верхней ячейке указаны ФИО и группа             | Выполнено                  |
| Спарсен идентификатор новости                     | Выполнено                  |
| Спарсено название новости                         | Выполнено                  |
| Спарсена дата размещения                          | Выполнено                  |
| Спарсен URL новости                               | Выполнено                  |
| Для каждой новости спарсен идентификатор          | Выполнено                  |
| Для каждой новости спарсено название              | Выполнено                  |
| Для каждой новости спарсена дата                  | Выполнено                  |
| Для каждой новости спарсено количество просмотров | Выполнено                  |
| Для каждой новости спарсен текст                  | Выполнено                  |
| Учтена вариативность текста                       | Выполнено                  |
| Для каждой новости спарсены теги                  | Выполнено                  |
| Общий CSV сохранён рядом с ноутбуком              | Выполнено                  |
| Подробный CSV сохранён в папке `news_content`     | Выполнено                  |
| Ячейки выполнены                                  | Выполнено                  |


---

## 35. Особенности и важные решения

В ходе работы были приняты следующие важные решения:

1. Использовать точные CSS-селекторы после анализа HTML.
2. Не удалять вложенные элементы из тега `time`, чтобы не потерять просмотры.
3. Извлекать дату как первый текстовый фрагмент внутри `time`.
4. Извлекать просмотры из `span.icon.eye`.
5. Извлекать текст из `div.content.js-mediator-article`.
6. Учитывать не только абзацы, но и заголовки и списки.
7. Удалять из текста подписи к фото и источникам.
8. Сохранять теги одной строкой через запятую.
9. Удалять дубликаты по ID новости.
10. Сохранять CSV в кодировке `utf-8-sig` для корректного отображения русского текста.

---

## 36. Вывод

В ходе лабораторной работы был реализован веб-скрейпер для сайта ITMO News.

С помощью библиотек `requests` и `BeautifulSoup` были загружены и разобраны HTML-страницы новостей. Из каждой новости были извлечены основные и дополнительные данные: идентификатор, заголовок, дата публикации, URL, количество просмотров, текст и теги.

Полученные данные были структурированы в виде таблиц с помощью библиотеки `pandas` и сохранены в CSV-файлы. Общая информация о новостях была сохранена в файл `general_news.csv`, а подробное содержимое новостей — в файл `news_content/news_content.csv`.

Работа позволила закрепить навыки:

* отправки HTTP-запросов;
* анализа HTML-разметки;
* поиска элементов с помощью CSS-селекторов;
* обработки текста;
* извлечения числовых данных из URL;
* формирования табличных данных;
* сохранения результатов в CSV;
* организации файловой структуры проекта.

Итоговый код соответствует требованиям задания и позволяет автоматически собрать датасет новостей с сайта ITMO News.



