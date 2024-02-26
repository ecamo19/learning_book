
# Basic Queries


```sql
SELECT *
FROM movies;
```


```sql
SELECT column_1, column_2
FROM movies;
```

```sql
SELECT name AS 'titles'
FROM movies
```


## DISTINCT

Return the unique values in a column

```sql
SELECT DISTINCT genre
FROM movies;
```