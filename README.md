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

<summary><strong>Задание 1: Диапазон дат в данных</strong></summary>

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
































</details>
