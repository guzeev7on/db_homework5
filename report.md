- Подготовительный этап
    создали директорию, docker-compose.yml, поместили дата-сеты в директорию, 
    создали индекс
    ![alt text](image.png)
    ![alt text](image-1.png)
    ![alt text](image-2.png)

    импорты и установление связей
    ![alt text](image-3.png)
    ![alt text](image-4.png)

- Аналитические запросы
    топ-10 артистов по количеству коллабораций
    ![alt text](image-5.png)
    Самый жанрово-разнообразный артист по жанрам его коллабораторов
    ![alt text](image-6.png)

    время с индексом - 110 мс
    ![alt text](image-7.png)

    время без индекса - 124 мс
    ![alt text](image-8.png)

- Обогащение модели
    Обогатили модель сущностью `Country`, отражающая страны, в чартах которых присутствовал артист. Это позволит анализировать географическое распространение популярности исполнителей. Между узлами `Artist` и `Country` было введено отношение `CHARTED_IN`. 
    code:
    CREATE CONSTRAINT country_code_unique IF NOT EXISTS
    FOR (c:Country)
    REQUIRE c.code IS UNIQUE;

    MATCH (a:Artist)
    WITH a,
        CASE
        WHEN a.chart_hits_raw IS NULL OR trim(a.chart_hits_raw) = '[]' THEN []
        ELSE [entry IN split(replace(replace(replace(trim(a.chart_hits_raw), "[", ""), "]", ""), "'", ""), ",")
                | trim(entry)]
        END AS chart_entries
    UNWIND chart_entries AS entry
    WITH a, entry, split(entry, " (") AS parts
    WHERE size(parts) = 2
    WITH a,
        trim(parts[0]) AS country_code,
        toInteger(replace(parts[1], ")", "")) AS hits
    WHERE country_code <> "" AND hits IS NOT NULL
    MERGE (c:Country {code: country_code})
    MERGE (a)-[r:CHARTED_IN]->(c)
    SET r.hits = hits;

    ![alt text](image-9.png)
    ![alt text](image-10.png)

- Задание 3. Жанровая экосистема
    Постройте граф жанров (projection), где два жанра связаны, если между их представителями существует хотя бы одна коллаборация. Определите:
        Какие жанры наиболее "центральны" в современной музыке?
        Какие пары жанров имеют самую сильную связь?
        Есть ли изолированные жанры?

    постройка жанрового графа
    ![alt text](image-11.png)

    самые "пересекаемые" или "центральные" жанры
    ![alt text](image-12.png)

    пары жанров с самой сильной связью
    ![alt text](image-13.png)

    жанры одиночки
    ![alt text](image-14.png)

- Задание 4. Цепочка артистов
    связь находится часто через других популярных разностороннихартистов
    ![alt text](image-15.png)

    

