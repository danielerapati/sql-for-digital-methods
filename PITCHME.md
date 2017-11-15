Big Data and Digital Methods
----------------------------

Daniele Rapati

Data Engineer

---

[tinyurl.com/yajq8vgp](https://tinyurl.com/yajq8vgp)

[gitpitch.com/danielerapati/sql-for-digital-methods](https://gitpitch.com/danielerapati/sql-for-digital-methods)


---

Big Data in businesses
----------------------

- Volume, Velocity, Variety
- Data storage at __scale__
- Data as an __asset__ rather than a cost

Note:

This is the "buzzwords" slide. Bear with me.

A __buzzword__ is a word that people used too much, eroding its meaning.

If there is time at the end I have a slide on going beyond "Big Data" and where the industry is moving.
"Big Data" definitely buzzword.

+++

Relational Databases
--------------------

- invented in the 70s by IBM / university collaboration ([Codd](https://en.wikipedia.org/wiki/Relational_database))
- enabled: electronic banking, e-commerce, web 2.0 |
- paid: Oracle, Microsoft SQL Server |
- open source: MySQL, PostgreSQL, SQLite |
- cloud |

Still by far the most used data storage.

Note:

among other notable usages: RPG videogames with lots of weapons and items and monsters

Also watch out for embedded databases in mobile apps. Look for SQLite in the about page

I can talk for hours about NoSQL systems and "movement", why it came about and why it died quickly, more on this at the end.

+++

SQL
----  

The language of relational databases.

A __table__ of data has __columns__ and __rows__.

You ask questions to the data using a __query__.

---

Practical SQL on Google Sheets
------------------------------

You need:

- a laptop
- a Google account |

+++

Practical SQL on Google Sheets
------------------------------

[tinyurl.com/yavh3dch](https://tinyurl.com/yavh3dch)

go here and clone this Google Sheet:

File -> Make a copy

+++

A Query starts with `SELECT`

```sql
SELECT <column>, <other column>, ...
```

In Google Sheets __columns identified by letters__

Note:

I want number of likes and the link
```sql
select E, C
```

+++
```sql
SELECT <column>, <other column>, ...
WHERE <condition>
```

you can filter using `WHERE`

Note:

I want the text only if they have few comments

```sql
SELECT B
WHERE D < 200
```

or less than 1000.


```sql
SELECT B, D
WHERE D < 1000
```

Let's add a condition: in September
```sql
SELECT B, A, D
WHERE D < 1000
  AND A < date'2017-10-01'
```

+++

Exercise
--------

Find links of all the tweets with more than 1 million likes

Note:
```sql
SELECT C
WHERE E > 1000000
```

+++

Search for specific content:

```sql
SELECT ...
WHERE <text column> CONTAINS <text>
```

Note:

```sql
SELECT A, B, D
WHERE B CONTAINS '@jimmy_chin'
```

```sql
SELECT A, B, D
WHERE B CONTAINS '#africa'
```

```sql
SELECT A, B, D
WHERE B CONTAINS 'africa'
```

```sql
SELECT A, B, D
WHERE B CONTAINS 'africa'
AND NOT B CONTAINS '#africa'
```

+++
```sql
SELECT ...
WHERE ...
ORDER BY <column> <ASC|DESC>, <other column> ...
```

Note:
Text ordered by likes
```sql
SELECT B, E
ORDER BY E
```

Most likes at the top
```sql
SELECT B, E
ORDER BY E DESC
```

Can also order by date.
```sql
SELECT B, E, A
ORDER BY A
```

+++

Avoid overwhelming data:

```sql
SELECT ...
...
LIMIT <how many>
```

Note:

The most used query in the world
```sql
SELECT *
LIMIT 10
```
quick look at data.

Limit gets more interesting if used with order by:
```sql
SELECT C, B, E
ORDER BY E DESC
LIMIT 5
```

+++

Exercise
--------

Find everything about the 10 oldest tweets

Note:
```sql
SELECT *
ORDER BY A
LIMIT 10
```

+++

Exercise
--------

Find link and date for the 20 top tweets by number of likes

Note:
```sql
SELECT C, A
ORDER BY E DESC
LIMIT 20
```

Bonus: not containing the word Photo

+++
Aggregation Functions
---------------------

```sql
SELECT <function>(<column>) AS <alias>, ...
```
- `SUM`
- `AVG`
- `COUNT` , `COUNT(DISTINCT <column>)``
- and all other functions supported

Note:
```sql
SELECT COUNT(*)
WHERE B CONTAINS 'photo'
```

```sql
SELECT
'jimmy_chin',
COUNT(A),
SUM(D),
SUM(E),
MIN(A),
MAX(A)
WHERE B CONTAINS '@jimmy_chin'
```

+++

```sql
SELECT
  <group column> ,
  <aggregation function>
...
GROUP BY <group column>
```

Note:
```sql
SELECT
  F,
  COUNT(A),
  SUM(E)
GROUP BY F
```

```sql
select
  month(A),
  count(A),
  SUM(E),
  MIN(A),
  MAX(A)
group by month(A)
```

```sql
SELECT
  month(A) +1,
  day(A),
  count(A),
  SUM(E),
  hour(MIN(A)), minute(MIN(A)),
  hour(MAX(A)), minute(MAX(A))
group by month(A) +1, day(A)
order by month(A) +1, day(A)
```

+++

Exercise
--------

Find the total number of tweets and average likes by day.

Note:
```sql
SELECT
  month(A) +1,
  day(A),
  count(A),
  AVG(E)
group by month(A) +1, day(A)
order by month(A) +1, day(A)
```

but we can also cheat and add another column

can we do a Graph?

+++

"Under the Hood"
----------------

```
=query(<data>, <query>, <first row is columns>)
```

---

Break: 13 minutes
-----------------

More at: [codingisforlosers.com/google-sheets-query-function](https://codingisforlosers.com/google-sheets-query-function/)

Feel free to ask me anything

---

A real-world example: GDELT
---------------------------

- [Google BigQuery](https://cloud.google.com/bigquery/) as a database
- [redash](https://github.com/getredash/redash/) as SQL interface

+++

[demo.redash.io](https://demo.redash.io)

Login with your Google Account

+++

BigQuery
--------

Cloud database by Google

Massive scale

Large number of free datasets included

+++

GDELT dataset
-------------

[gdeltproject.org/data.html](https://www.gdeltproject.org/data.html)

GDELT 2.0 Global Knowledge Graph

+++

Use `FROM` to specify the dataset.

```sql
SELECT *
FROM [gdelt-bq:gdeltv2.gkg@-86400000-]
LIMIT 10
```

Note:
Not the easiest dataset to work with, we will use a few tricks to get what we want.

+++

```sql
SELECT
    date(
      FORMAT_UTC_USEC(string(integer(date/1000000)))
    ) as date,
    count(*) as articles
FROM [gdelt-bq:gdeltv2.gkg]
group by 1
order by 1 desc
```

Note:
(if there is time and people show interest) we are going to build this query up

+++

```sql
SELECT
    date(
      FORMAT_UTC_USEC(string(integer(date/1000000)))
    ) as date,
    dayofweek(date(
      FORMAT_UTC_USEC(string(integer(date/1000000)))
    )) as dayofweek,
    count(*) as articles
FROM [gdelt-bq:gdeltv2.gkg]
group by 1,2
order by 1 desc,2
```

+++

```sql
SELECT
    theme,
    COUNT(*) as count
FROM (
    select UNIQUE(REGEXP_REPLACE(SPLIT(V2Themes,';'), r',.*', '')) theme
    from [gdelt-bq:gdeltv2.gkg@-86400000-]
    where V2Persons like '%Trump%'
)
group by theme
ORDER BY 2 DESC
LIMIT 300
```

+++

```sql
SELECT
    date(
      FORMAT_UTC_USEC(string(integer(date/1000000)))
    ) as date,
    theme,
    COUNT(*) as count
FROM (
    select date, UNIQUE(REGEXP_REPLACE(SPLIT(V2Themes,';'), r',.*', '')) theme
    from [gdelt-bq:gdeltv2.gkg@-2764800000-]
    where V2Persons like '%Trump%'
)
group by 1, theme
having count(*) > 8000
ORDER BY 1 desc, 3 DESC
```

+++

```sql
SELECT lat, lon, sum(cnt) as total,
case when sum(cnt) > 50000 then '>50000'
     when sum(cnt) > 10000 then '>10000'
     else '>5000'
end as series
from (
  SELECT lat, lon, COUNT(*) as cnt
  FROM (
    SELECT
           REGEXP_REPLACE(
              REGEXP_EXTRACT(
                SPLIT(V2Locations, ';'),
                r'^[2-5]#.*?#.*?#.*?#.*?#(.*?#.*?)#'),
              '^(.*?)#(.*?)$', '\\1') AS lat,
           REGEXP_REPLACE(
              REGEXP_EXTRACT(
                SPLIT(V2Locations, ';'),
                r'^[2-5]#.*?#.*?#.*?#.*?#(.*?#.*?)#'),
                '^(.*?)#(.*?)$', '\\2') AS lon,
    FROM [gdelt-bq:gdeltv2.gkg@-2764800000-]
    where V2Persons like '%Trump%'
  )
  WHERE lat is not null
  GROUP BY lat, lon
  ORDER BY 3 DESC
)
GROUP BY 1,2
HAVING sum(cnt) > 5000
```

+++


For a lot more see:

- [GDELT Demos](https://blog.gdeltproject.org/a-compilation-of-gdelt-bigquery-demos/)
- [alternative BigQuery interface](https://bigquery.cloud.google.com/table/gdelt-bq:gdeltv2.gkg?pli=1)

---

Thank you :D

Feel free to ask me anything.

[github/danielerapati](https://github.com/danielerapati)

Note:
Last Slide

---

Bonus: Big Data is a buzzword
-----------------------------

- how much is __Big__ ?
- value is not necessarily linked to size |
- more variables, more problems: [correlation is not causation](https://www.xkcd.com/552/) |
- personal privacy |

---

Bonus: NoSQL is dead
--------------------

- scale requires multiple computers
- difficult to make SQL work on multiple computers |
- 2005-2015 alternative products (2010-2013 NoSQL) |
- 2013+ full SQL in the Cloud (Google Spanner) |
