
## text in database
`select 'https://sql4e.com/neon/'`

`trunc(random()*100000) || repeat('Lemon', 5) || generate_series(1,5)`

## text functions
- postgresql.org/docs/functions-string

`select pg_relation_size('textfun), pg_indexes_size('textfun');`

`select content from textfun where content like '%150000%';`

`select strpos(contecnt, 'ttps://') from textfun `

`select substr(content, 2, 4) from `

`select split_part(content, '/', 4)`

`select translate(content, 'th.p/', 'TH!P')` 

### B-Tree Index 
```sql
explain analyze select content from textfun where content like '%racing%'
-- Seq Scan : 언제 끝날지 모르기에 bad query
```
```sql 
explain analyze select content from textfun where content like 'racing%'
-- Index Only Scan : 속도 훨씬 빠름
```
```sql 
explain analyze select content from textfun where content in (
    select content from textfun where content like '%racing%'
)
-- subselect: not good
-- Nested Loop : 훨씬 오래 걸림
```