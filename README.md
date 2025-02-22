# sql

## Создание и удаление таблиц
- число столбцов фиксировано и не может быть больше чем 1600
- по умолчению id нет. Это обычное поле
```sh
  CREATE TABLE films(
    title VARCHAR(100),   // символы до 100
    release_date: DATE,
    age_restriction INT,
    rating: REAL, // тип для хранения значений с плавающей точкой одинарной точности
    country VARCHAR(60),
);

 CREATE TABLE IF NOT EXISTS films(...
 DROP TABLE products;
```

Если не указать тип - то значение будем NULL
```sh
CREATE TABLE products(
  id           SERIAL UNIQUE NOT NULL      // порядок автоматический - 1,2,3,4 ... UNIQUE- уникальнео значение
  title        NOT NULL VARCHAR(100),
  subtitle     VARCHAR(100) DEFAULT "name default",
  created      DATE DEFAULT CURRENT_DATE,
  price        NOT NULL NUMERIC CHECK (price > 0),
  discount:    NOT NULL NUMERIC CHECK (discount > 0),

  CONSTRAIN valid_discount CHECK (price > discount)    // общее ограничение для всей таблицы + полный синтаксис
  UNIQUE (title, created) //общее ограничение для полей 
)


но, id SERIAL UNIQUE NOT NULL - это не совсем правильно. Подобные ограничения задаются через PK
id   PRIMARY KEY,

PRIMARY KEY (product_id, id) // или через общее обьявление


// FOREIGN KEY
product_id INTEGER REFERENCES products (product_id) // указать через поле

// или указать общее значение + запрещаем удалять
FOREIGN KEY (product_id) REFERENCES products (product_id) ON DELETE RESTRICT
// указать еще один ключ + при удалении одного - удаляется второе
FOREIGN KEY (user_id) REFERENCES user (id) ON DELETE CASCADE 
```

```sh
  INSERT INTO films VALUES('MATRIX', '1990-02-12', 18, 8.7, 'США'); // порядок имеет значение

//можно поменять порядлк
INSERT INTO films (release_date, title, rating, country) VALUES('1990-02-12', 'MATRIX', 18, 8.7, 'США');

//можно встаить много сразу
INSERT INTO films VALUES ('1990-02-12', 'MATRIX', 18, 8.7, 'США'), ('1990-02-12', 'MATRIX', 18, 8.7, 'США'), ('1990-02-12', 'MATRIX', 18, 8.7, 'США'), ;
```

### Изменение данных

```sh
ALTER TABLE orders ADD COLUMN date_created DATE;

ALTER TABLE orders DROP COLUMN date_created;
ALTER TABLE orders DROP COLUMN date_created CASCADE;

ALTER TABLE users ADD COLUMN user_id SERIAL PRIMARY KEY;

ALTER TABLE orders ALTER COLUMN user_id TYPE BIGINT;

ALTER TABLE orders RENAME COLUMN quontity to quontity2;

UPDATE PRODUCT SET price=5_500 where id=1; 
```

### Получение данных

```sh
  // ВЫБОРКА - ТАБЛИЦА - УСЛОВИЕ

  - SELECT * FROM FILM; //все поля
  - SELECT title, country FROM film;

 - SELECT title, country, rating/2 AS fire_start_rating FROM film;
 - SELECT * FROM film where release_date > "2000-01-01" AND age_restriction < 18;
 - SELECT * FROM film ORDER BY reting DESC; //ASC

- SELECT DISTINCT country FROM film; //DISTINCT - только уникальные значения

- SELECT title, ROUND((price-discoud) /price * 100, 2) as offer from products;

- SELECT * FROM colors, items where color_id = item_id;

- SELECT * FROM colors, items
WHERE LEFT(colors.title, 1)=LEFT(items.title, 1)      // LEFT - берет первую букву
```

### Изменение данных
- если не перечислять значения, то надо указывать в строгой последовательности

```sh
INSERT INTO products (title, price, discount) VALUES ('IPhone', 49_000, 44_000), ('IPhone PRO', 47_000, 45_000);

INSERT INTO products (title, price, discount) SELECT title || ' PRO', price *2, discount *2 from products; 
```


### Соединение таблиц

**JOIN** - берет данные и добавляет в текущую таблицу
**INNER JOIN** - в postgres это то же самое, что и JOIN (они возарвщают пересечение - если даннфе есть там и там). Пример - в фильмах есть фильм с Францией. А в таблице с фильмами Франции нет. Фильм с Францией в выборку не попадет

**LEFT JOIN** - если надо вывести все фильмы со странами, даже, если стран нет (все данные из исходной, даже, если в смежной данных нет)

**RIGHT JOIN** - в выборку попадут все значения из соединяемой таблицы

```sh
// надо указать условие - по чему обьединять
// если поменять местами SELECT * FROM counntry Join film - результат такой же (порядок только иной)

  SELECT * FROM film JOIN country ON film.country = country.name;
  SELECT title, country, currency FROM film JOIN country ON film.country = country.name;

  SELECT 
  order_id, order_date, total_amount, customer_name, email 
  from orders o 
  JOIN customers c 
  ON c.customer_id=o.customer_id
  order by order_date
  ;


// left JOIN - порядок FROM country LEFT JOIN film важент - так как country будет вставлена вся целиком
  SELECT title, name, currency FROM country LEFT JOIN film ON film.country = country.name;

// right JOIN 
 SELECT title, name, currency FROM country RIGHT JOIN film ON film.country = country.name;
```

LEFT JOIN

<img width="400" alt="image" src="https://github.com/user-attachments/assets/4e5031b8-5da9-4788-b77b-a5881ef43a28" />
<img width="400" alt="image" src="https://github.com/user-attachments/assets/75281be1-e0d3-4499-8ce7-4d5f12c4d6c7" />


Типы связей
<img width="300" alt="image" src="https://github.com/user-attachments/assets/e8e9406d-4be1-4492-bc87-881aecb0feff" />
<img width="300" alt="image" src="https://github.com/user-attachments/assets/63ef62b0-84e2-440c-8ccb-78454b1c88b3" />
<img width="300" alt="image" src="https://github.com/user-attachments/assets/448c9c7b-3ba8-4832-a40f-d5223cd1f922" />



### Псевдонимы (очень укарачивают записи)

```sh
 SELECT title, name, currency FROM country c RIGHT JOIN film f ON f.country = c.name;
```

### Агрегированные функции
- count, sum, avg, min/max

отличие WHERE от HAVING. Where - сначало выбирает, а затем группирует и вычисляет агрегатные ф-ии. HAVING отбирает строки групп после группировки и вычисление агрегатов
  ```sh
 SELECT max(rating) from film;  // max знаяение рейтинга

 SELECT country, max(rating), count(*) from film group by country;

SELECT country, max(rating) from film 
where release_date >= "1990-01-01"  // сначало фильтруем
GROUP BY country  // группируем по странам (если США 3 фильма, то будет 3)
HAVING count(*) >=2; // применяем агрегат где значение сгуппированных больше 2-х
```

### Обновление данных

```sh
  UPDATE film SET age_restriction = 18 where id =4;
```

### Удаление данных

```sh
  DELETE FROM film  where id =4;
```


## Первичные и внешние ключи

PK (первичный ключ) - это столбец или группа столбцов, который уникально идентифицируют каждую запись в таблице
FK (внешние ключи) - используется для достижения целостности ссылочных данных. Это столбец или группа столбцов таблицы, которые связвны с первичным ключем или уникальным ключем в другой таблице

```sh
// какую таблицу изменить и как именно
ALTER TABLE country
ADD PRIMARY KEY (name);
// после этого мы не сможем добавить дубликаты в name


// сейчас нельзя будет создать film без указания country c name из таблицы country
ALTER TABLE country
ADD FOREIGN KEY (country) REFERENCES country(name);
```

<img width="791" alt="image" src="https://github.com/user-attachments/assets/05aafc86-8c32-4d1f-8acd-df61a41abe23" />


## Индексы
Предположим, следующий запрос является очень популярным
- SELECT * FROM country where country="Italy";
Чтобы выполнить такой запрос необходимо посмотреть все записи - строка за строкой. И если записей 10 000 000 - то такое сканирование не эффективно
**Добавление индекса по полю country** существенно увеличит скорость поиска
Но, построение индексов может занимать время. И операция записи блокируется до окончания построения индекса
  
```sh
CREATE INDEX film_country_index
ON film (country);

DROP INDEX film_country_index;
```

## Транзакции (позволяет обьединять несколько операций обращения к БД в одну группу)
- все изменения видны только внутри транзакции (UPDATE первый ниже - если паралельно выполнить SELECT * from account - то будет первонаяальное значений)
- только после завершение транзакции (COMMIT) мы получим результат в базе
- либо выполняются вместе и полностью, либо совсем ничего (целостность транзакции)
  
  <img width="400" alt="image" src="https://github.com/user-attachments/assets/68c625c1-5c04-43f4-a996-dac914cde2e6" />
  <img width="400" alt="image" src="https://github.com/user-attachments/assets/006e10f2-6cc1-4ea7-8678-c455e3defd07" />

```sh
BEGIN;  // начало транзакцмм
UPDATE account SET balance = balance -1000 where name ="Andrey";
UPDATE account SET balance = balance +1000 where name ="Sam";
COMMIT;
```

## UNION (обьединение результатов 2-х запросов)
```sh
SELECT * FROM product
UNION
SELECT * FROM product

SELECT * FROM product
UNION ALL        // ALL добавить дубликаты
SELECT * FROM product


SELECT * FROM product
INTERSECT    // одинаковые записи из 2-х таблиц
SELECT * FROM product_new
```
## WITH (для упрощение запросов. создает временную таблицу)

## Типы данных
- числовые SMALLINT, INTEGER, BIGINT(8байт), REAL(с плавающей запятой и одинарной точности), DOUBLE PRECISION ((с плавающей запятой и двойной точности - до 15 знаков)), NUMERIC
- символьные CHAR(n, если короче n-заполняются пробелами), VARCHAR(n, не дополняет пробелами), TEXT(без длины)
- Дата и время DATE (ISO8601, '2025-01-01'), TIME (HH:MI:SS[.MI]), TIME WITH TIMEZONE, TIMESTAMP (DATE+TIME), INTERVAL
- Логический TRUE, FALSE, NULL(not defined)

Операторы и функции
- логические AND, OR, NOT
- сравнения > < >= <= = <> , предикаты "BETWEEN 1 AND 3", "NOT BETWEEN", "IS NULL", "IS FALSE"
- строковые функции CONCAT (||),
- функции DATE ()

## Механизи управления конкурентным доступом
(когда 2 пользователя одновременно пытаются прочитать и изменить или прочитать одни и теже данные)

### Изоляция транзакций
- каким образом изменения одной транзакции видны всем остальным транзакциям в случае одновременного доступа к данным
- определяет 4 уровня изоляции транзакций (Read Uncommited (чтение незафиксированных данных), Read Commited (чтение зафиксированных данных), Repeatable Read (повторяемое чтение), Serializable (сериализуемость))
  
**Read Uncommited** - самый слабый (видит все изменения другими транзакцийми, даже, если они были незафиксированы) - практически не поддерживается и будет автоматом повышен до   "Read Commited"

![image](https://github.com/user-attachments/assets/86643065-0b59-4f7b-a75c-e3f4f0954033)
(Аномалия грязного чтения - это когда мы начали транзакция и ее не закончили или сделали ROLLBACK, а второй пользователь уже прочитал данные и видит 500 и 1000, хотя после ROLBACK - 1000, 1000 )

  **Read Commited** - транзакция не прочитает данные, которые еще не были зафиксированы другими транзакциями (исключается грягное чтение). Опасность, что данные, прочитанные текущей транзакцией, могут быть изменены другой транзакцией в процессе работы

![image](https://github.com/user-attachments/assets/e3e56287-d03c-415f-b2b6-4c714e074641)

в этот момент второй пользователь видит первлначальное состояние транзакции, пока транзакция не закончена (COMMIT)
(неповторяемое чтение или фантомное чтение) -читаем одно и то же, а результат разный, так как транзакция не закончена
  
   **Repeatable Read** - данные, прочитанные в первый раз остануться точно такиеми же при повторном чтениии в течении этой транзакции. Но, может быть фантомное чтение - когда что-то изменилось, а мы про это не знаем

![image](https://github.com/user-attachments/assets/798b3be7-edbc-4d56-bbf9-5983d0aa0c79)
![image](https://github.com/user-attachments/assets/c5a09a1a-fbb6-4d4f-969d-df00e5a21599)

(фантомная запись - мы берем данные в транзакцию, а их меняют. И мы про это ничего не знаем)
   
  **Serializable** - самый строгий. На этом уровне нарантируется, что выполнение последовательных транзакций будет эквивалентно последовательному выполнению. Исключает все анамалии, но производительность падает

![image](https://github.com/user-attachments/assets/086ec634-65e7-4097-8b50-f5321a94fe4e)
![image](https://github.com/user-attachments/assets/b24fc9e7-b603-4a2e-b93d-136abd304bf9)

Одна операция не закончится, пока открыто соединение с уровнем "Serializable". 

**Это все для mysql**

**Для posgreSQL** 
- Read Uncommited - отсутствует

### Механизм MVCC
- изменения пользователя не видны другим до тех пор, пока они не будут зафиксированы (изоляция и высокая производетельность)

### Механизм блокировки на уровне таблиц и строк. 
