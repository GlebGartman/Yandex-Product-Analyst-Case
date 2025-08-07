<h3 align="center">Описание проекта</h3>
<p align="center">
Это кейс-задание, выполненное в рамках отбора на позицию продуктового аналитика в <strong>Яндексе</strong>.  
Основной акцент сделан на анализ пользовательских запросов в <strong>Яндекс Картинках</strong> с двух платформ: <strong>мобильных устройств (touch)</strong> и <strong>компьютеров (desktop)</strong>.  
Работа выполнена в <strong>Jupyter Notebook</strong> с использованием <strong>Python</strong>, <strong>SQL-запросов</strong> и библиотеки <strong>matplotlib</strong> для визуализации данных.
</p>

<p>В рамках анализа были выполнены следующие задачи:</p>

- **Расчёт количества запросов с текстом <strong>«ютуб»</strong> по каждой платформе**  
- **Построение <strong>топ‑10 самых частотных запросов</strong> для desktop и touch, выявление отличий**  
- **Анализ <strong>динамики поискового трафика в течение дня</strong> и объяснение различий**  
- **Выделение <strong>контрастных тематик запросов</strong>, доля которых заметно отличается на разных устройствах**  


<details>
<summary><strong>Подключение к SQLite из Jupyter Notebook</strong></summary>

SQLite — это встроенная в Python база данных, не требующая установки.
Для подключения в Jupyter Notebook используем модуль `sqlite3`

### Загрузка и подготовка данных

```python
# В файле нет названия колонок, поэтому задаем их вручную
df = pd.read_csv(r'C:\Users\1\Desktop\Курсы\Стажировки\Стажка яндекс\Продуктовый аналитик\data.tsv', 
                 sep="\t", 
                 header=None, 
                 names=["query", "timestamp", "platform"])

# Переведем столбец с датой в формат datetime
df["timestamp"] = pd.to_datetime(df["timestamp"], unit="s")

# Создаём соединение с SQLite в памяти
conn = sqlite3.connect(":memory:")

# Записываем DataFrame в таблицу queries
df.to_sql("queries", conn, index=False, if_exists="replace")

# Проверим содержимое таблицы
query = """
SELECT query, timestamp, platform
FROM queries
"""

df = pd.read_sql(query, conn)
df.head(10)

```

</details>


<details>
<summary><strong>Результаты</strong></summary>

<summary><strong>Задача 1: Диапазон дат в данных</strong></summary>

📅 Определите даты, охватываемые в предоставленном наборе данных.

📌 В таблице `queries` с данными пользовательских запросов содержатся следующие поля:

- `query` — текст запроса  
- `timestamp` — время запроса в формате UNIX  
- `platform` — тип устройства (desktop или touch)  

🕒 Поле `timestamp` было преобразовано в человекочитаемый формат даты и времени.  
📆 В результате были определены минимальная и максимальная даты, которые задают полный диапазон анализа.

### Код

```python
diapozon = """
SELECT MAX(timestamp) as maximum, MIN(timestamp) as minimum FROM queries
"""

interval = pd.read_sql(diapozon, conn)

max_date, min_date, = interval.iloc[0]
first_task = f'Дата начала: {min_date},  Дата конца: {max_date}'
print(first_task)

```

### Диапазон дат анализа

![Диапазон дат](https://drive.google.com/uc?export=view&id=1d9L5amoo38IKcBBDucV-2CKqRM7BxpQX)


---


<summary><strong>Задача 2: Количество запросов с текстом «ютуб» по каждой платформе</strong></summary>

📌 Необходимо посчитать, сколько раз в данных встречается слово **«ютуб»** в запросах, отдельно для каждой платформы (`desktop`, `touch`).

Для обработки использовался SQL-запрос с фильтрацией по ключевому слову с учётом регистра (через `LIKE`/`ILIKE` в PostgreSQL или `LOWER(...) LIKE` в SQLite).

### Код

```python

youtube = """
SELECT platform, count(query) as kolvo
FROM queries
WHERE query like '%ютуб%' or query like '%Ютуб%'
group by platform
"""

youtube_kolvo = pd.read_sql(youtube, conn)
display(youtube_kolvo.style.hide(axis="index"))

```

### Количество запросов с текстом «ютуб» по платформам

![Количество запросов с текстом «ютуб» по платформам](https://drive.google.com/uc?export=view&id=1Cd2fu0fyrN6vJV0hlDPcr4VZpePM32LS)

---


<summary><strong>Задача 3: Топ‑10 самых частотных запросов в каждой платформе (desktop и touch)</strong></summary>

📌 Вывести 10 наиболее часто встречающихся запросов отдельно для `desktop` и `touch`.  
Сравнить полученные списки и определить отличия в популярных запросах между платформами.

### Сначала выведем 10 наиболее частотных запросов для платформы `desktop`

```python

desktop_top = """
SELECT query, COUNT(query) as kolvo_zaprosov FROM queries
WHERE platform = 'desktop'
GROUP BY query
order by kolvo_zaprosov desc
LIMIT 10
"""

top_10_desktop = pd.read_sql(desktop_top, conn)
display(top_10_desktop.style.hide(axis="index"))

```

![Топ‑10 запросов — desktop](https://drive.google.com/uc?export=view&id=13x5rwIoIF3_OV7W_3b8-a_nCNNiEXoP-)

### График Топ-10 запросов - desktop
![Топ‑10 запросов — desktop](https://drive.google.com/uc?export=view&id=1jwjB96mqOZKCUOfit9ITIxO99I3ssDRO)

---


### Теперь выведем 10 наиболее частотных запросов для платформы `touch`.

```python

touch_top = """
SELECT query, COUNT(query) as kolvo_zaprosov FROM queries
WHERE platform = 'touch'
GROUP BY query
order by kolvo_zaprosov desc
LIMIT 10
"""

top_10_touch = pd.read_sql(touch_top, conn)
display(top_10_touch.style.hide(axis="index"))

```

![Топ‑10 запросов для touch](https://drive.google.com/uc?export=view&id=1QQQAcndZHC4bvADCUWtVYJ9-mt2qW94x)

### График Топ-10 запросов - touch

![Топ‑10 частых запросов на touch](https://drive.google.com/uc?export=view&id=1LhgKvPMCCO8v2YypaxdJFDjR013OYh-0)














</details>
