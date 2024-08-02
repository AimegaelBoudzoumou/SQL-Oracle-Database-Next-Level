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

create table colours (
    colour_name           varchar2(10),
    minimum_bricks_needed integer
);

insert into colours values ('blue', 2);
insert into colours values ('green', 3);
insert into colours values ('red', 2);
insert into colours values ('orange', 1);
insert into colours values ('yellow', 1);
insert into colours values ('purple', 1);

insert into bricks values (1, 'blue');
insert into bricks values (2, 'blue');
insert into bricks values (3, 'blue');
insert into bricks values (4, 'green');
insert into bricks values (5, 'green');
insert into bricks values (6, 'red');
insert into bricks values (7, 'red');
insert into bricks values (8, 'red');
insert into bricks values (9, null);

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
select * from (

)
```
Hint: you need group by colour. Use min(brick_id) to find the minimum. The query should return the following rows (the order may be different):

This is a solution:
```sql
select * from (
    select colour, min(brick_id), max(brick_id)
    from bricks
    group by colour
)
```
![image](https://github.com/user-attachments/assets/102bd72d-622e-49e7-be46-9c0283307bc0)

## 3. Nested Subqueries
Nested subqueries go in your where clause. The query filters rows in the parent tables.

For example, to find all the rows in colours where you have a matching brick, you could write:

```sql
select * from colours c
where c.colour_name in (
    select b.colour from bricks b
);
```

You can also use exists in a nested subquery. The following is the same as the previous query:
```sql
select * from colours c
where EXISTS (
    select null from bricks b
    where b.colour = c.colour_name
);
```

You can filter rows in a subquery too. To find all the colours that have at least one brick with a brick_id less than 5, write:

```sql
select * from colours c
where  c.colour_name in (
    select b.colour from bricks b
    where  b.brick_id < 5
);
```

Or with exists:

```sql
select * from colours c
where exists (
    select null from bricks b
    where c.colour_name = b.colour
    and   b.brick_id < 5
);
```

## 4. Correlated vs. Uncorrelated
A subquery is correlated when it joins to a table from the parent query. If you don't, then it's uncorrelated.

This leads to a difference between IN and EXISTS. EXISTS returns rows from the parent query, as long as the subquery finds at least one row. So the following uncorrelated EXISTS returns all the rows in colours:

```sql
select * from colours
where exists (
    select null from bricks
);
```
Note: When using EXISTS, what you select in the subquery does not matter because it is only checking the existence of a row that matches the where clause (if there is one). You can "select null" or "select 1" or select an actual column.

This is because the query means:

Return all the in colours if there is at least one row in bricks

To find all the colour rows which have at least one row in bricks of the same colour, you must join in the subquery. Normally you need to correlate EXISTS subqueries with a table in the parent.

## 5. NOT IN vs NOT EXISTS
You can do the reverse of IN & EXISTS by placing NOT in front of them. This returns you all the rows from the parent which don't have a match in the subquery.

For example to find all the rows in colours without a matching colour in bricks, you can use NOT EXISTS:

```sql
select * from colours c
where not exists (
    select null from bricks b
    where c.colour_name = b.colour
);
```
But do the same with NOT IN:
```sql
select * from colours c
where  c.colour_name not in (
    select b.colour from bricks b
);
```
And your query returns nothing!
This is because there is a row in bricks with a null colour. So the previous query is the same as:

```sql
select * from colours c
where c.colour_name not in (
    'red', 'green', 'blue',
    'orange', 'yellow', 'purple',
    null
);
```
For the NOT IN condition to be true, comparing all its elements to the parent table must return false.

But remember that comparing anything to null gives unknown! So the whole expression is unknown and you get no data.

To resolve this, either use NOT EXISTS or add a where clause to stop the subquery returning null values:
```sql
select * from colours c
where c.colour_name not in (
    select b.colour from bricks b
    where  b.colour is not null
);
```
### Try it
Complete the subquery to find all the rows in bricks with a colour where colours.minimum_bricks_needed = 2:

```sql
```

This is a solution:
```sql
select * from bricks b
where b.colour in (
    select c.colour_name from colours c
    where c.minimum_bricks_needed = 2
);

select * from bricks b
where exists (
    select c.colour_name from colours c
    where c.minimum_bricks_needed = 2
    and   c.colour_name = b.colour
);
```
![image](https://github.com/user-attachments/assets/bbf85a4e-8c59-49e4-8fa6-2242662e7751)

## 6. Scalar Subqueries
Scalar subqueries return one column and at most one row. You can replace a column with a scalar subquery in most cases.

For example, to return a count of the number of bricks matching each colour, you could do the following:
```sql
select colour_name, (
    select count(*)
    from bricks b
    where b.colour = c.colour_name
    group by b.colour
)   bricks_counts
from colours c;
```
Note the colours with no matching bricks return null. To show zero instead, you can use NVL or coalesce. This needs to go around the whole subquery:
```sql
select colour_name, nvl ( (
    	select count(*)
    	from   bricks b
    	where  b.colour = c.colour_name
    	group by b.colour
    ), 0 ) brick_counts
from colours c;
```
Usually you will correlate a scalar subquery with a parent table to give the correct answer.

You can also use scalar subqueries in your having clause. So instead of a join, you could write the query in part 1 to find those bricks you have less than the minimum needed like so:

```sql
select colour, count(*) c
from bricks b
group by colour
having count(*) < (
    select c.minimum_bricks_needed
    from   colours c
    where  c.colour_name = b.colour
);
```
### Try it
Complete the scalar subquery below to find the minimum brick_id for each colour:

```sql
select c.colour_name, (

       ) min_brick_id
from   colours c
where  c.colour_name is not null;
```

This is a solution:

```sql
select c.colour_name, (
    select min(brick_id)
    from bricks b
    where c.colour_name = b.colour
    group by colour
      ) min_brick_id
from  colours c
where c.colour_name is not null;
```

![image](https://github.com/user-attachments/assets/d4eb0f2c-1efd-46dc-bf58-9fe6c5d45741)


## 7. Common Table Expressions
Common table expressions (CTEs) enable you to name subqueries. You then refer to these like normal tables elsewhere in your query. This can make your SQL easier to write and understand later.

CTEs go in the with clause above the select statement. The following defines a CTE that counts how many rows of each colour there are in the bricks table:

```sql
with brick_colour_counts as (
    select colour, count(*)
    from   bricks b
    group  by colour
)

select * from brick_colour_counts ;
```

## 8. CTEs: Reusabe Subqueries


## 9. Literate SQL
## 10. Testing Subqueries
