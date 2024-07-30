# Null

### Content

1. Introduction
2. Nothing equals null!
3. Is null condition
4. Is not null
5. Nulls in range comparisons
6. Null functions
7. Magic Values

----------------------------------------------------------------------------------------------------------------------
### Prerequisite SQL
```sql
create table toys (
    toy_id               integer not null primary key,
    toy_name             varchar2(100) not null,
    weight               number(10, 2) not null,
    quantity_of_stuffing integer,
    volume_of_wood       integer,
    times_lost           integer
);

insert into toys values (1, 'Mr Penguin', 50, 100, null, 10);
insert into toys values (2, 'Blue Brick', 10, null, 10, null);
insert into toys values (3, 'Red Brick', 20, null, 20, 1);
commit;
```

## 1. Introduction
The table contains rows with several null values. Run the query below to view it:

```sql
select * from toys;
```

## 2. Nothing equals null!
Null is neither equal to nor not equal to anything. The result of:

null = <anything>

is always unknown. So the following query will always return no rows:

```sql
select * from toys
where  volume_of_wood = null;
```

This is also the case when you check if a column is not equal to null:

```sql
select * from toys
where volume_of_wood <> null;
```

## 3. Is null condition
To find rows that have a null-value, use the "is null" condition.

This query finds all the rows storing null in volume_of_wood:

```sql
select * from toys
where volume_of_wood is null;
```

### Try it!

Insert this incomplete query the editor. Complete it to find the row where the times_lost is null ('Blue Brick'):

```sql
select *
from   toys
where  
```

This is a solution:
```sql
select * from toys
where times_lost is null;
```

## 4. Is not null
To do the reverse, and find all the rows with a non-null value, use the "is not null" condition:

```sql
select *
from   toys
where  volume_of_wood is not null;
```

## 5. Nulls in range comparisons

If you want to find the toys with a volume of wood less than 15, you can write:

```sql
select * from toys
where  volume_of_wood < 15;
```

But this will exclude rows where the volume_of_wood is null. If you want to include these, you must also test for this:

```sql
select * from toys
where  volume_of_wood < 15 or
       volume_of_wood is null;
```

## 6. Null functions
Oracle Database includes many functions to help you handle nulls. NVL and coalesce are two that map nulls to non-null values.

### NVL
This takes two arguments. If the first is null, it returns the second:

```sql
select toy_name, volume_of_wood, nvl (volume_of_wood, 0) mapped_volume_of_wood
from toys;
```

### Coalesce
This is like NVL. But it can take any number of arguments. It returns the first non-null value it finds:

```sql
select t.*,
       coalesce ( volume_of_wood, 0 ) coalesce_two,
       coalesce ( times_lost, volume_of_wood, quantity_of_stuffing, 0 ) coalesce_many
from   toys t;
```

You can use these functions in the where clause to map nulls to a real value. So you no longer need a separate "or column is null" test.

For example, these return the same rows as the final query in section 5:

```sql
select *
from   toys
where  nvl ( volume_of_wood , 0 ) < 15;

select *
from   toys
where  coalesce ( volume_of_wood , 0 ) < 15;
```

### Try it
Complete the following query to find all the rows where times_lost is less than 5 or null:

```sql
select *
from   toys
where
```

This is a solution:

```sql
select * from toys
where times_lost < 5 or
      times_lost is null;
```
This query should return the rows for "Red Brick" and "Blue Brick".

Other way to write a query that returns these two rows :

```sql
select * from toys
where coalesce ( times_lost , 0 ) < 5;
```

## 7. Magic Values
Null complicates your where clause. So some people are tempted to use "magic values" instead of null for missing or not applicable information.

For example, Mr. Penguin is made of fluff, not wood! So to show the volume_of_wood doesn't apply, you could set this to the "impossible" value of minus one.

Run this update to do this:

```sql
update toys
set   volume_of_wood = -1
where volume_of_wood is null;

select * from toys;
```

You can now use standard comparison logic to find all the rows with a volume of wood less than 15 or where it doesn't apply:

```sql
select * from toys
where volume_of_wood < 15;
```

__At first glance this seems to make your code simpler. But it brings complications elsewhere. For example, say you're analysing the volume of wood used in your toys. You want to find the mean, standard deviation and minimum values for this.__

Only Blue Brick and Red Brick use wood. So these calculations should return 15, 7.07 (rounded), and 10 respectively.

But if you run the query below, you get different results:

```sql
select avg ( volume_of_wood ),
       stddev ( volume_of_wood ),
       min ( volume_of_wood )
from   toys;
```

This is because it includes the "impossible" value of -1 for Mr. Penguin.

To avoid this, you need to check that the volume_of_wood is greater than or equal to zero:

```sql
select avg ( volume_of_wood ),
       stddev ( volume_of_wood ),
       min ( volume_of_wood )
from   toys
where  volume_of_wood >= 0; 
```

So while magic values appear to make life easier, they often bring bigger problems elsewhere.
