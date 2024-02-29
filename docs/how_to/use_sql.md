
## Basic Queries


### Shows the database schema
```sql
.shema
```


```sql
SELECT *
FROM movies;
```


```sql
SELECT column_1, column_2
FROM movies;
```

 Rename column
```sql
SELECT name AS 'titles'
FROM movies;
```

### DISTINCT

Return the unique values in a column

```sql
SELECT DISTINCT genre
FROM movies;
```

### WHERE

Restrict query results 

```sql
SELECT *
FROM movies
WHERE imdb_rating  > 8;
```

#### WHERE LIKE

##### WHERE LIKE with _

Operator when you want to compare similar values. In the example below a string of 5 
characters that start with Se and end with en and that has one unknown character will be searched i.e(Se7en or Seven)


```sql
SELECT *
FROM movies
WHERE name LIKE 'Se_en';
```

##### WHERE LIKE with %

Read % as: Filter values that start with

```sql
SELECT *
FROM movies
WHERE name LIKE 'A%';
```

... or end with


```sql
SELECT *
FROM movies
WHERE name LIKE '%A';
```

... or contain

```sql
SELECT *
FROM movies
WHERE name LIKE '%man%';
```

##### IS NULL

```sql
SELECT name
FROM movies
WHERE imdb_rating IS NOT NULL;

```


##### BETWEEN
```sql
SELECT *
FROM movies
WHERE name BETWEEN 'A' AND 'J';
```

##### AND
```sql
SELECT * 
FROM movies 
WHERE year > 1985
	AND genre == 'horror';
```

##### OR
```sql
SELECT *
FROM movies
WHERE year > 2014
	OR genre = 'action';
```
### ORDER BY

```sql
SELECT name, year, imdb_rating
FROM movies
ORDER BY imdb_rating DESC;
```

### LIMIT
```sql
SELECT *
FROM movies
ORDER BY imdb_rating DESC
limit 3;
```

### CASE

 It is SQL’s way of handling if-then logic.

```sql
SELECT name,
CASE
	WHEN genre = 'romance' THEN 'Chill'
	WHEN genre = 'comedy' THEN  'Chill'
	ELSE 'Intense'
END AS 'Mood'
FROM movies;

```

## AGGREGATE FUNCTIONS

### COUNT 

Calculate how many rows are in a table
```sql
SELECT COUNT(*)
FROM fake_apps
WHERE price = 0;
```

### SUM

Function that takes the name of a column as an argument and returns the sum of all the values in that column