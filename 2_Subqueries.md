# Subqueries

### Content

1. Introduction
2. Inline Views
3. Nested Subqueries
4. Correlated vs. Uncorrelated
5. NOT IN vs NOT EXISTS
6. Scalar Subqueries
7. Common Table Expressions
8. CTEs: Reusabe Subqueries
9. Literate SQL
10. Testing Subqueries
----------------------------------------------------------------------------------------------------------------------
### Prerequisite SQL

```sql
create table bricks (
    brick_id integer,
    colour   varchar2(10)
);

create table colour (
    colour_name           varchar2(10),
    minimum_bricks_needed integer
);

insert into colour values ('blue', 2);
insert into colour values ('green', 3);
insert into colour values ('red', 2);
insert into colour values ('orange', 1);
insert into colour values ('yellow', 1);
insert into colour values ('purple', 1);

insert into bricks values (1, 'blue');
insert into bricks values (2, 'blue');
insert into bricks values (3, 'blue');
insert into bricks values (4, 'green');
insert into bricks values (5, 'green');
insert into bricks values (6, 'red');
insert into bricks values (7, 'red');
insert into bricks values (8, 'red');
insert into bricks values (8, null);

commit;
```

## 1. Introduction
This tutorial shows you how to write subqueries. It uses the bricks and colours table. Run the queries below to see their contents:

```sql
select * from bricks;

select * from colours;
```

## 2. Inline Views
An inline view replaces a table in the from clause of your query. In this query, "select * from bricks" is an inline view:

```sql
select * from (
    select * from bricks
);
```
You use inline views to calculate an intermediate result set. For example, you can count the number of bricks you have of each colour using:

```sql
select * from (
    select colour, count(*) c
    from bricks
    group by colour
)   brick_counts
```

You can then join the result of this to the colours table. This allows you to find out which colours you have fewer bricks of than the minimum needed defined in colours:

```sql
select * from (
    select colour, count(*) c
    from bricks
    group by colour
)   brick_counts
right join colours
on    brick_counts.colour = colours.colour_name
where nvl ( brick_counts.c, 0 ) < colours.minimum_bricks_needed
```

### Try it
Complete the query below, using an inline view to find the min and max brick_id for each colour of brick:

```sql

```

The solution is:
```sql

```

## 3. Nested Subqueries
## 4. Correlated vs. Uncorrelated
## 5. NOT IN vs NOT EXISTS
## 6. Scalar Subqueries
## 7. Common Table Expressions
## 8. CTEs: Reusabe Subqueries
## 9. Literate SQL
## 10. Testing Subqueries
