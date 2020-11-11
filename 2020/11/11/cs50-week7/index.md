# CS50 2020 Week7: SQL Problem Answers

<!--more-->
Click title to detail page

## [Movies](https://cs50.harvard.edu/x/2020/psets/7/movies)

```sql
--schema
CREATE TABLE movies (
                    id INTEGER,
                    title TEXT NOT NULL,
                    year NUMERIC,
                    PRIMARY KEY(id)
                );
CREATE TABLE stars (
                movie_id INTEGER NOT NULL,
                person_id INTEGER NOT NULL,
                FOREIGN KEY(movie_id) REFERENCES movies(id),
                FOREIGN KEY(person_id) REFERENCES people(id)
            );
CREATE TABLE directors (
                movie_id INTEGER NOT NULL,
                person_id INTEGER NOT NULL,
                FOREIGN KEY(movie_id) REFERENCES movies(id),
                FOREIGN KEY(person_id) REFERENCES people(id)
            );
CREATE TABLE ratings (
                movie_id INTEGER NOT NULL,
                rating REAL NOT NULL,
                votes INTEGER NOT NULL,
                FOREIGN KEY(movie_id) REFERENCES movies(id)
            );
CREATE TABLE people (
                id INTEGER,
                name TEXT NOT NULL,
                birth NUMERIC,
                PRIMARY KEY(id)
            );
```
### 1.sql
```sql
select title from movies where year=2008;
```
### 2.sql
```sql
select birth from people where name="Emma Stone";
```
### 3.sql
```sql
select title from movies where year >= 2018 order by title asc;
```
### 4.sql
```sql
select count(movies.id) from movies 
join ratings on movies.id=ratings.movie_id 
and ratings.rating=10;
```
### 5.sql
```sql
select title, year from movies 
where title like 'Harry Potter%' 
order by year;
```
### 6.sql
```sql
select avg(rating) from ratings 
join movies on movies.id=ratings.movie_id 
where movies.year=2012;
```
### 7.sql
```sql
select ratings.rating, movies.title
from movies
join ratings on movies.id=ratings.movie_id
where movies.year=2010
order by ratings.rating desc,
movies.title asc;
```
### 8.sql
```sql
select name from people 
join stars on stars.person_id=people.id 
join movies on stars.movie_id=movies.id 
where movies.title="Toy Story";
```
### 9.sql
```sql
select distinct name from people
join stars on stars.person_id=people.id
join movies on stars.movie_id=movies.id
join ratings on ratings.movie_id=movies.id
where movies.year=2004
order by people.birth;
```
### 10.sql
```sql
select name from people
join directors on directors.person_id=people.id
join movies on directors.movie_id=movies.id
join ratings on ratings.movie_id=movies.id
where ratings.rating >= 9;
```
### 11.sql
```sql
select movies.title from movies
join stars on stars.movie_id=movies.id
join people on stars.person_id=people.id
join ratings on ratings.movie_id=movies.id
where people.name="Chadwick Boseman"
order by ratings.rating desc
limit 5;
```
### 12.sql
```sql
select title from movies
join stars on stars.movie_id=movies.id
join people on stars.person_id=people.id
where people.name="Johnny Depp"
and title in (
select title from movies
join stars on stars.movie_id=movies.id
join people on stars.person_id=people.id
where people.name="Helena Bonham Carter"
);
```
### 13.sql
```sql
select name from people
join stars on stars.person_id=people.id
join movies on stars.movie_id=movies.id
where movies.id in
(select movies.id from movies
join people on stars.person_id=people.id
join stars on stars.movie_id=movies.id
where people.name="Kevin Bacon"
and people.birth=1958)
and people.name != "Kevin Bacon";
```
