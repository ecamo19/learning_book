
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
