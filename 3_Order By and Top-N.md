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
insert into toys values ('Kangaro',       10, 29.99, date'2017-03-01', date'2018-06-01');
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

```

## 8. Top-N Queries
## 9. Top-N with Ties
