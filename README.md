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


С `touch` преимущественно ищут контент для взрослых и поздравления с днём рождения,  
а с `Desktop`, помимо контента для взрослых, также ищут учебные материалы, картинки и оформление рабочего стола.

---

<summary><strong>Задание 4: Динамика трафика в течение дня</strong></summary>

📌 Для каждого часа суток рассчитано количество поисковых запросов на каждой платформе.
Анализ трафика помогает понять, в какие временные промежутки пользователи наиболее активно обращаются к поиску изображений и как меняется интенсивность запросов в зависимости от времени суток.


```python

sutki = """
SELECT strftime('%H', timestamp) as hour, COUNT(timestamp) as kolvo_zaprosov FROM queries
GROUP BY hour
"""
sutki_zaprosy = pd.read_sql(sutki, conn)
display(sutki_zaprosy.style.hide(axis="index"))

```
![Распределение трафика запросов по часам](https://drive.google.com/uc?export=view&id=1milp8jX6RK4v2Rrb4vJcHbhGO--7pjVM)


### Распределение трафика запросов по часам 

![График распределения трафика в течение дня](https://drive.google.com/uc?export=view&id=1btlBv6oautOoR5Eun40JFPPxOE6d_ieR)


График запросов начинает расти с раннего утра, активно увеличивается уже с **3–4 часов**,  
когда в **Приморском крае** и **Сибири** начинается активная деятельность и рабочее время.  

С **7 до 17 часов** количество запросов держится на стабильном высоком уровне,  
формируя *«плато»* дневной активности. Этот период совпадает с **активным рабочим временем**  
в **Центральной России**, на **Урале**, **Юге** и в **Северо-Западном регионе**,  
где проживает основная часть населения страны.  

В **18:00** наблюдается **резкий всплеск**, что, вероятно, связано с окончанием рабочего дня или учёбы.  

После 18 часов трафик начинает постепенно снижаться, достигая **минимума** к ночи.  
Такое распределение можно объяснить тем, что большинство пользователей активнее всего  
используют поиск в рабочее и учебное время.

---


<summary><strong>Задание 5: Тематики запросов по платформам</strong></summary>

📌 Для каждой тематики был рассчитан процент её встречаемости среди всех запросов на **мобильных устройствах** и **компьютерах**. Затем вычислена разница долей между платформами, чтобы выявить тематики с наибольшим контрастом.
Для анализа различий между платформами были выделены ключевые категории запросов:  
**🔞 Контент 18+**, **🎮 Игры**, **🛒 Покупки**, **📚 Учёба**, **🎶 Музыка**, **🎂 Поздравления**, **👥 Развелечения** , **🌐 Соцсети**, **📱 Обои**, **✝️ Религия**, **🚗 Авто**, **🐶 Домашние животные** и **другие**.


```python

category = """
WITH kolvo_platforms as 
(SELECT query, platform, COUNT(query) OVER(PARTITION BY platform) as vsego FROM queries),

tema as 
(SELECT query,
CASE
    WHEN LOWER(query) LIKE '%таблиц%' OR LOWER(query) LIKE '%алфавит%' OR LOWER(query) LIKE '%задани%' OR LOWER(query) LIKE '%школ%' THEN 'Учёба'
    WHEN LOWER(query) LIKE '%фильм%' OR LOWER(query) LIKE '%сериал%' OR LOWER(query) LIKE '%актер%' OR LOWER(query) LIKE '%актрис%' THEN 'Развлечения'
    WHEN LOWER(query) LIKE '%поздравлени%' OR LOWER(query) LIKE '%открытк%' THEN 'Поздравления'
    WHEN LOWER(query) LIKE '%одноклассник%' OR LOWER(query) LIKE '%вконтакт%' OR LOWER(query) LIKE '%телеграм%' THEN 'Соцсети'
    WHEN LOWER(query) LIKE 'xxx%' OR LOWER(query) LIKE '%секс%' OR LOWER(query) LIKE '%порн%' THEN 'Контент 18+'
    WHEN LOWER(query) LIKE 'игр%' OR LOWER(query) LIKE '%roblox%' OR LOWER(query) LIKE '%маинкрафт%' THEN 'Игры'
    WHEN LOWER(query) LIKE 'продукт%' OR LOWER(query) LIKE '%цен%' OR LOWER(query) LIKE '%одежд%' THEN 'Покупки'
    WHEN LOWER(query) LIKE 'клип%' OR LOWER(query) LIKE '%слуша%' OR LOWER(query) LIKE '%песн%' OR LOWER(query) LIKE '%музык%' THEN 'Музыка'
    WHEN LOWER(query) LIKE '%библия%' OR LOWER(query) LIKE '%икон%' OR LOWER(query) LIKE '%молитв%' THEN 'Религия'
    WHEN LOWER(query) LIKE '%kia%' OR LOWER(query) LIKE '%bmw%' OR LOWER(query) LIKE '%audi%' THEN 'Авто'
    WHEN LOWER(query) LIKE '%кошк%' OR LOWER(query) LIKE '%корг%' OR LOWER(query) LIKE '%собак%' THEN 'Домашние животные'
    WHEN LOWER(query) LIKE '%обо%' OR LOWER(query) LIKE '%картинк%' OR LOWER(query) LIKE '%фон%' THEN 'Обои'
    ELSE 'Другие'
END as tematika,
platform, vsego FROM kolvo_platforms),

tema_vsego as 
(SELECT platform, tematika, count(tematika) as kolvo_tema, vsego FROM tema
GROUP BY platform, tematika, vsego)

SELECT platform, tematika, ROUND(100.0 * kolvo_tema / vsego, 2) as procent FROM tema_vsego
ORDER BY platform, tematika
"""

categories = pd.read_sql(category, conn)
display(categories.style.hide(axis="index"))

```

![Контраст тематики запросов на `desktop` и `touch`](https://drive.google.com/uc?export=view&id=119e__tXdZ2wkQN75HHJVM_FCBqFIBlTX)

### График долей тематик поисковых запросов по платформам

![Тематики запросов — различия между desktop и touch](https://drive.google.com/uc?export=view&id=1Z48iq9PdL4-GmBEzCQdvde548snxkeFh)

- **С мобильных устройств (Touch)** заметно выше доля запросов, связанных с:
  - 🎂 **Поздравлениями** — вероятно, пользователи быстро ищут открытки и поздравления в мессенджерах;
  - 🔞 **Контентом 18+** — такие запросы чаще поступают с телефонов;
- **С компьютеров (Desktop)** чаще ищут:
  - 📚 **Учебные материалы** — таблицы, алфавит, школьные задания и т.п.;
- Большая часть запросов попадает в категорию **«Другие»**, но и здесь видно, что телефоны доминируют по общему количеству.


</details>

<details> 

<summary><strong>Выводы</strong></summary>
