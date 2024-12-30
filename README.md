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

### Получение данных

```sh
  // ВЫБОРКА - ТАБЛИЦА - УСЛОВИЕ

  - SELECT * FROM FILM; //все поля
  - SELECT title, country FROM film;

 - SELECT title, country, rating/2 AS fire_start_rating FROM film;
 - SELECT * FROM film where release_date > "2000-01-01" AND age_restriction < 18;
 - SELECT * FROM film ORDER BY reting DESC; //ASC

- SELECT DISTINCT country FROM film; //DISTINCT - только уникальные значения
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
