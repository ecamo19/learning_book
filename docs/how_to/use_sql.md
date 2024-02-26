
# Basic Queries


Shows the database schema
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

## WHERE

Restrict query results 

```sql
SELECT *
FROM movies
WHERE imdb_rating  > 8;
```

### WHERE LIKE

Operator when you want to compare similar values. In the example above a string of 


```sql
SELECT *
FROM movies
WHERE name LIKE 'Se_en'
```
