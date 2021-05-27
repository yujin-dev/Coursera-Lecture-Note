
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
## Character Sets
`select ascii('H')` : 초기 memory save를 위해 주로 사용  
`select chr(72)` : much larger range of characters( ex. english, chinese .. )  
` show server_encoding`  
[ encoding ]
- ascii
- utf-8 : 32bit per character

[ python ]  
all string in memory are Unicode( good in memory)
- network, connections files, database data..

## Hash
- hashes : checksum( if a message was altered in transit)
- fixed-size output

[hash function]  
character, lower/upper case, length 등에 따라 hash 달라짐  
초기에 MD5, 현재는 sha-256( longer length ) 
 
## Index 
`create unique index cr2_md5 on cr2 (md5(url));`  
```sql
explain select * from cr2 where url='lemons'
-- Seq Scan
```
```sql
explain select * from cr2 where md5(url)=md5('lemons')
-- Index Scan using cr2_md5
```  
[ case 1 ]
```sql
create table cr3 (
    id SERIAL,
    url TEXT,
    url_md5 uuid unique,
    content TEXT
) ;
```
[ case 2 ]
```sql
create table cr3 (
    id SERIAL,
    url TEXT,
    content TEXT
) ;
```
```sql
insert into cr3 (url)
select repeat('Neon', 1000) || generate_series(1, 5000); 
-- 5000 rows( each row 1000*4 characters)
update cr3 set url_md5 = md5(url)::uuid 
-- casting( convert to uuid )
select pg_relation_size('cr3'), pg_indexes_size('cr3')
-- pg_relation_size > pg_indexes_size
-- B-tree index ( not Hash index )
explain analyze select * from cr3 where url_md5=md5('lemons')::uuid;
-- Index Scan using cr3_url_md5_key on crd
```
[ case 1 ] is much faster( 0.03 ) than [ case 2 ] ( 0.142)

[ Index Type ]  
- B-tree(default)
- Hash : smaller pg_relation_size, pg_indexes_size

