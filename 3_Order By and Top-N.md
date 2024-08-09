# Order By and Top-N : Sorting and Limiting Rows

### Content

1. Unordered Data
2. The Order By Clause
3. Reversing the Order
4. Resolving Ties
5. Sorting Null
6. Custom Sorting
7. Positional Notation vs. Aliases
8. Top-N Queries
9. Top-N with Ties
----------------------------------------------------------------------------------------------------------------------
### Prerequisite SQL

```sql
create table toys (
    toy_name       varchar2(30),
    weight         integer,
    price          number(5,2),
    purchased_date date,
    last_lost_date date
);

insert into toys values ('Miss Snuggles', 4,  9.99,  date'2018-02-01', date'2018-06-01');
insert into toys values ('Baby Turtle',   1,  5.00,  date'2016-09-01', date'2017-03-03');
insert into toys values ('Kangaroo',       10, 29.99, date'2017-03-01', date'2018-06-01');
insert into toys values ('Blue Dinosaur', 8,  9.99,  date'2013-07-01', date'2016-11-01');
insert into toys values ('Purple Ninja',  8,  29.99, date'2018-02-01', null);

commit;
```

## 1. Unordered Data
Queries without an order by can return data in any sequence. When accessing a single table, this is often the order you inserted the rows into the table.

For example, the rows in the toys table are out-of-sequence for all its columns:
```sql
select * from toys;
```

## 2. The Order By Clause
To guarantee the rows appear in a given sequence, you must use an order by. This sorts numbers from smallest to largest. So to sort the toys from cheapest to most expensive, order by price:
```sql
select * from toys
order  by price;
```

Dates sort from oldest to newest. So the following sorts the toys with the most recently purchased last:
```sql
select * from toys
order  by purchased_date;
```

And character data sorts alphabetically. So this sorts by the toy's names:
```sql
select * from toys
order  by toy_name;
```

## 3. Reversing the Order
You can invert the order by adding the desc keyword. This stands for descending.

So the following query sorts the rows from the most expensive toy to the cheapest:
```sql
select * from toys
order  by price desc;
```

## 4. Resolving Ties
There are two toys priced at 9.99 (Miss Snuggles & Blue Dinosaur) and two at 29.99 (Kangaroo & Purple Ninja). If you sort by only price, the database can return each of these either way round.

To avoid this and guarantee a particular sequence, add more columns to your order by. Do this until each set of values appear in only one row.

You can do this here by adding toy_name:

```sql
select toy_name, price from toys
order  by price, toy_name;
```

### Try it
Complete the following query, so it sorts the rows by:

1. Weight, lightest to heaviest
2. Toys with the same weight by purchased date from the last bought to the first:

```sql
select toy_name, weight, purchased_date 
from   toys
```

This is a solution:
```sql
select toy_name, weight, purchased_date 
from   toys
order  by weight, purchased_date desc;
```
![image](https://github.com/user-attachments/assets/51c3ce6b-8bae-4ef8-ab8c-95d008c1eda7)

## 5. Sorting Null
By default, Oracle Database sorts null last in the data:
```sql
select * from toys
order  by last_lost_date;
```

You can change this with the nulls clause. Set this to nulls first to get null-valued rows at the top:
```sql
select * from toys
order  by last_lost_date nulls first;
```

### Try it

Complete the query to sort the rows by:

1. Price, cheapest to most expensive
2. Toys with same price by date last lost, from newest to oldest. Rows with a null last_lost_date should appear above any of the same price
```sql
select toy_name, price, last_lost_date 
from   toys
```

This is a solution:
```sql
select toy_name, price, last_lost_date from toys
order  by price, last_lost_date desc nulls first;
```
![image](https://github.com/user-attachments/assets/c5a99362-586d-44e9-8886-803389323fed)

## 6. Custom Sorting
Sometimes you want to order data in way that doesn't follow the standard sorting rules. For example, say you want to sort the toys by name. But Miss Snuggles is your favourite, so you want this to always appear at the top. All the toys after should appear alphabetically.

To do this, you need to map Miss Snuggles to a value lower than for all the other toys. For example, 1 for Miss Snuggles and 2 for everything else.

You can do this with a case expression in your order by. For example:

```sql
select * from toys
order  by case
  when toy_name = 'Miss Snuggles' then 1
  else 2
end, toy_name;
```

## 7. Positional Notation vs. Aliases
It can be tricky to debug custom sorting functions. To help with this, include the expression in your select:

```sql
select t.*,
       case
         when toy_name = 'Miss Snuggles' then 1
         else 2
       end
from   toys t
order  by case
         when toy_name = 'Miss Sbuggles' then 1
         else 2
       end, toy_name;

```

But this means you have the function in two places! This makes code maintenance harder.

You can avoid this by using positional notation or aliases.

### Positional notation
This is where you put the number of the column in the select you want to order by (working from left to right in the output). So the following sorts by the sixth column then the first. i.e. the case expression then toy_name:
```sql
select t.*,
       case
         when toy_name = 'Miss Snuggles' then 1
         else 2
       end
from   toys t
order  by 6, 1;
```
But this makes it hard to spot which column you're sorting by. Particularly if the query uses "select *". And if you need to change the columns you're selecting, it's easy to overlook the positional order by. Leading to wrong results!

### Aliases
It's better to give the function an alias. Then refer to this alias in the order by clause:

```sql
select t.*,
       case
         when toy_name = 'Miss Snuggles' then 1
         else 2
       end custom_sort
from   toys t
order  by custom_sort, toy_name;
```

### Try it
Complete the query, so:

- Kangaroo is top
- Blue Dinosaur is second
- The remaining toys are ordered by price, cheapest to most expensive

```sql
select t.toy_name, t.price,
       case
         when toy_name = 'Kangaroo'
         when toy_name = 'Blue Dinosaur'
         else 
       end custom_sort
from   toys t
order  by
```

This is a solution:
```sql
select t.toy_name, t.price,
       case
         when toy_name = 'Kangaroo' then 1
         when toy_name = 'Blue Dinosaur' then 2
         else 3
        end custom_sort
from    toys t
order   by custom_sort, toy_name;
```

## 8. Top-N Queries
A top-N query returns the first N rows in a sorted data set. For example, to find the three cheapest toys.

There are several way to do this in Oracle Database

### Rownum
Rownum is an Oracle-specific function. It assigns an increasing number to each row you fetch.

But if you use it in a where clause before the order by, you'll get unexpected results. For example, the following tries to get the three most expensive toys:

```sql
select * from toys
where  rownum <= 3
order  by price desc;
```

But it includes the cheapest, Baby Turtle! This is because the database processes order by after where. So this gets any three rows. Then sorts them.

To fix this, sort the data in a subquery. Then filter the results of this:
```sql
select * from (
    select *
    from   toys t
    order  by price desc
)
where rownum <= 3;
```

### Row_number
Row_number is an analytic function. Like rownum, it assigns an incrementing counter. This is determined by the sort defined in the order by in the over clause.

To use this in a top-N query, you must also use a subquery:

```sql
select * from (
    select t.*, row_number() over (order by price desc ) rn
    from toys t
)
where rn <= 3
order by rn;
```

### Fetch first

Oracle Database 12c introduced the ANSI compliant fetch first clause. This goes after the order by and removes the need to use a subquery:

```sql
select * from toys
order  by price desc
fetch  first 3 rows only;
```

## 9. Top-N with Ties
When doing a top-N query sorting by non-unique values, you have an important question to answer:

Do you want exactly N rows, all the rows for the first N values, or N rows along with any that have the same value as the Nth?

For example, both Miss Snuggles and Blue Dinosaur have a price of 9.99. So if you fetch the three most expensive toys, you could get either of them!

You can guarantee you get both using fetch first. Swap only for "with ties". This will return you N rows, plus any that have the same value for the order by columns as the last. So you get both the 9.99 priced toys:
```sql
select toy_name, price from toys
order  by price desc
fetch  first 3 row with ties;
```

You can get the same effect using a subquery by swapping row_number for rank:
```sql
select * from (
    select t.*,
           rank() over ( order by price desc ) rn
    from   toys t
)
where rn <= 3
order by rn;
```

If you want all the rows for the first three values, use dense_rank in the subquery instead:

```sql
select * from (
    select t.*,
           dense_rank() over ( order by price desc ) rn
    from   toys t
)
where rn <= 3
order by rn;
```

In all these examples you'll get at least three rows in your output. But it could be many more!

Usually you want to get exactly N rows. To do this and get the same rows every time, ensure there are no duplicated values in your order by columns.

If you want to learn more about the difference between rank, dense_rank and row_number, [watch this video](https://www.youtube.com/watch?v=Tuxxbuywb1w).

### Try it
Write a query to return the first three toys, ordered by toy_name
```sql
select toy_name 
from   toys
```

This is a solution:
```sql
select toy_name 
from toys
order by toy_name
fetch first 3 row with ties;

select toy_name from (
    select t.*,
           rank() over ( order by toy_name ) rn
    from toys t
)
where rn <= 3
order by rn;
```
![image](https://github.com/user-attachments/assets/92883c39-f16a-45b3-bd19-e9db3cd42624)

