# 增删改查

## SELECT

```
`SELECT column1, column2, ...
FROM table_name;
```

\```

## INSERT

```
`insert into table_name clown1,clown2,......
```

`values(value1,value2,.......)``

## UPDATE

```
UPDATE table_name
SET column1 = value1, column2 = value2, ...
WHERE condition_column = condition_value;
```

## DELETE

```
DELETE FROM table_name
WHERE column = condition_value;
```

# DISTINCT 

例如，一个教师可以教很多门课程，那么我们如何从这些课程中筛选出教师呢？

sql如下

```sql
SELECT DISTINCT c.teacher_code
FROM b_curriculum c
LEFT JOIN b_teacher t
    ON c.teacher_code = t.teacher_code
WHERE c.school_year = '2024-2025'
  AND t.teacher_code IS NULL;
```

解释：在需要筛选的数据前使用DISTINCT即可

# Union

从多张表中查询相同列名的值之后在一起展示，UNION会自动将两表中查询出的列值进行去重，若不需要去重，则使用UNION ALL

```
SELECT  clown1 FROM table1
UNION
SELECT  clown2 FROM table2
```