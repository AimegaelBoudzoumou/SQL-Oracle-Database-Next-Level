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
Because you access CTEs in the same way as a regular table, you can use it many times in your query. This can help if you want to use this subquery many times.

For example if you want to find:
- Which colours you have more bricks of than the minimum needed
- The average number of bricks you have of each colour

You need to group the bricks by colour. Then filter the colours table where this count is greater than the minimum_bricks_needed for that colour. And compute the mean of the counts.

You can do the filtering with a nested subquery. And show the average in a scalar subquery. This looks like:

```sql
select c.colour_name,
       c.minimum_bricks_needed, (
         select avg ( count(*) )
         from   bricks b
         group  by b.colour
       ) mean_bricks_per_colour
from    colours c
where   c.minimum_bricks_needed < (
    select count(*) c
    from   bricks b
    where  b.colour = c.colour_name
    group  by b.colour
);
```
![image](https://github.com/user-attachments/assets/1f036b72-f547-4267-ad7f-d05c74e63cf2)

Note that "group by colour" appears twice in the statement. This creates maintenance problems. If you need to change this, say to join bricks to another table, you have to do this in two places.

Using CTEs, you can do the group by once. Then refer to it in your select:

```sql
with brick_counts as (
    select b.colour, count(*) c
    from   bricks b
    group  by b.colour
)
    select c.colour_name,
           c.minimum_bricks_needed, (
             select avg ( bc.c )
             from   brick_counts bc
           ) mean_bricks_per_colour
    from   colours c
    where  c.minimum_bricks_needed < (
      select bc.c
      from   brick_counts bc
      where  bc.colour = c.colour_name
    );
```
![image](https://github.com/user-attachments/assets/95a0cff6-d8a4-45f1-a5dc-c2158e729ff0)

So now if you need to join bricks to a new table to get the count you only have to edit the subquery brick_counts.

Oracle Database can also optimize queries that access the same CTE many times. This can make it more efficient than regular subqueries.

## 9. Literate SQL
Literate programming is a concept introduced by Donald Knuth. The idea is to write code that makes sense if you read it like a book: from start to finish.

Simple SQL queries follow this principle. For example the query "Get me the brick_ids of all the bricks that are red or blue" is the following statement:

```sqm
select brick_id
from   bricks
where  colour in ('red', 'blue');
```

But subqueries usually break this human readable flow. For example, if you want to find which bricks you have less of than the average number of each colour, you need to:

1. Count the bricks by colour
2. Take the average of these counts
3. Return those rows where the value in step 1 is less than in step 2

Without the with clause, you need to write something like the following:

```sql
select colour
from   bricks
group  by colour
having count (*) < (
    select avg ( colour_count )
    from   (
      select colour, count (*) colour_count
      from   bricks
      group  by colour
    )
);
```

Step one is at the bottom of the query! Using CTEs, you can order the subqueries to match the logic above:

```sql
with brick_counts as (
    -- 1. Count the bricks by colour
    select b.colour, count (*) c
    from   bricks b
    group  by b.colour
),   average_bricks_per_colour as (
    -- 2. Take the average of these counts
    select avg ( c ) average_count
    from   brick_counts
)
    select * from brick_counts bc
    join   average_bricks_per_colour ac
	-- 3. Return those rows where the value in step 1 is less than in step 2
	on     bc.c < ac.average_count;
```
![image](https://github.com/user-attachments/assets/481f154b-14cd-4397-a3fb-3ce21fa47e7d)

This makes it easier to figure out what a SQL statement does when you return to it later.

## 10. Testing Subqueries
Another big advantage of the with clause is it makes your SQL easier to test and debug.

If you want to check that the brick counts from the previous query are correct, update the final from clause:
```sql
with brick_counts as (
    select b.colour, count (*) c
    from   bricks b
    group  by b.colour
),   average_bricks_per_colour as (
    select avg ( c ) average_count
    from   brick_counts
)
    select * from brick_counts bc;
```

### Try it
Complete the following with clause to count how many rows there are in colours:
```sql
with colour_count as (

)
  select * from colour_count;
```

This is a solution:
```sql
with colour_count as (
    select count (*)
    from   colours
)
 select * from colour_count;
```
![image](https://github.com/user-attachments/assets/157de535-5357-41fe-be66-5341f19400d2)

