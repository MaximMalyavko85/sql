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

