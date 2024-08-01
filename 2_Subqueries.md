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
## 6. Scalar Subqueries
## 7. Common Table Expressions
## 8. CTEs: Reusabe Subqueries
## 9. Literate SQL
## 10. Testing Subqueries
