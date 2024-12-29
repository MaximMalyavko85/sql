# sql

```sh
  CREATE TABLE films(
    title VARCHAR(100),   // символы до 100
    release_date: DATE,
    age_restriction INT,
    rating: REAL, // тип для хранения значений с плавающей точкой одинарной точности
    country VARCHAR(60),
);
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

**LEFT JOIN ** - если надо вывести все фильмы со странами, даже, если стран нет (все данные из исходной, даже, если в смежной данных нет)

**RIGHT JOIN ** - в выборку попадут все значения из соединяемой таблицы

```sh
// надо указать условие - по чему обьединять
// если поменять местами SELECT * FROM counntry Join film - результат такой же (порядок только иной)

  SELECT * FROM film JOIN country ON film.country = country.name;

  SELECT title, country, currency FROM film JOIN country ON film.country = country.name;


// left JOIN - порядок FROM country LEFT JOIN film важент - так как country будет вставлена вся целиком
  SELECT title, name, currency FROM country LEFT JOIN film ON film.country = country.name;

// right JOIN 
 SELECT title, name, currency FROM country RIGHT JOIN film ON film.country = country.name;
```

LEFT JOIN

<img width="314" alt="image" src="https://github.com/user-attachments/assets/4e5031b8-5da9-4788-b77b-a5881ef43a28" />


### Псевдонимы (очень укарачивают записи)

```sh
// 
 SELECT title, name, currency FROM country c RIGHT JOIN film f ON f.country = c.name;
```
