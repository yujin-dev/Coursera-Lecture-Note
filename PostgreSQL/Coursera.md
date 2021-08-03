## text in database
`select 'https://sql4e.com/neon/'`  
`trunc(random()*100000) || repeat('Lemon', 5) || generate_series(1,5)`

### text functions
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

## regular expressions
`select email from em where email ~ 'umich';` : like : contains umich  
`select email from em where email ~ '^c';` like : starts with c  
`select email from em where email ~ 'edu$';` like : ends with edu  
`select email from em where email ~ '^[gnt]';` like : start with g OR n OR t
`select email from em where email ~ '[0-9]';` like : contains 0 ~ 9 ( any )    
`select email from em where email ~ '[0-9][0-9]';` like : contains 0~9 and 0~9
 

 ## Blocks  
read an entire block into memory  
compute the start of a block within a file for random access
- size : 8K( default )
- where a row is in a block? : 
block에 대한 정보로 row를 탐색

## Indexes
key to block  
- B-tree( default ) : keep the keys in sorted order by reorganizing the tree as keys are inserted
- Hash : quick lookup of long key strings
-  BRIN : Block Range Index 
- GIN : Generalized Inverted Indexes
- GiST : Generalized Search Tree
- SP-GiST : Space Partitioned Generalized Search Tree

#### B-Tree
http://upload.wikimedia.org/wikipedia/commons/thumb/6/65/B-tree.svg/500px-B-tree.svg.png

ex. insert 4 : less than 7   
첫번째 block에 삽입되는데 size에 따라 첫번째 block의 뒤에 숫자(6)은 다음 block으로 넘어감  
free space가 있기에 insert가 수월함  

### Forward and Invereted Index
- Forward : index( logical key ) -> key를 포함한 row를 탐색  
B-Tree, BRIN, Hash
- Inverse : index( string; query ) -> query를 매치되는 모든 rows 리스트를 반환함. 텍스트 검색에 주로 적용  
GIN, GiST 


### Stemmping
```sql
SELECT DISTINCT id, doc FROM docs AS D
JOIN docs_gin AS G ON D.id = G.doc_id
WHERE G.keyword = ANY(string_to_array('Search for Lemons and Neons', ' '));

 id |                         doc
----+-----------------------------------------------------
  1 | This is SQL and Python and other fun teaching stuff
  3 | UMSI also teaches Python and also SQL

```
The word "and" contributed no real meaning to our query. And it took up valuable space in our GIN index. So we put it on the stop word list

### Inverted Indexes
보통은 GIN, insert, update가 많으면 Gist가 효율적
```sql 
CREATE TABLE docs (id SERIAL, doc TEXT, PRIMARY KEY(id));

CREATE INDEX gin1 ON docs USING gin(string_to_array(doc, ' ')  array_ops);

INSERT INTO docs (doc) VALUES
('This is SQL and Python and other fun teaching stuff'),
('More people should learn SQL from UMSI'),
('UMSI also teaches Python and also SQL');
```
```sql
SELECT id, doc FROM docs WHERE '{learn}' <@ string_to_array(doc, ' ');

 id |                  doc
----+----------------------------------------
  2 | More people should learn SQL from UMSI

EXPLAIN SELECT id, doc FROM docs WHERE '{learn}' <@ string_to_array(doc, ' '); -- {learn} : text array

                                 QUERY PLAN
----------------------------------------------------------------------------
 Bitmap Heap Scan on docs  (cost=12.05..21.53 rows=6 width=32)
   Recheck Cond: ('{learn}'::text[] <@ string_to_array(doc, ' '::text))
   ->  Bitmap Index Scan on gin1  (cost=0.00..12.05 rows=6 width=0)
         Index Cond: ('{learn}'::text[] <@ string_to_array(doc, ' '::text))
-- Not Seq Scan ! Using Index
```

## Text Search Functions
- ts_vector() : order에 따라 distance betweend words
```sql
SELECT to_tsvector('english', 'UMSI also teaches Python and also SQL'); -- indexing

                   to_tsvector
--------------------------------------------------
 'also':2,6 'python':4 'sql':7 'teach':3 'umsi':1
```
- ts_query() : stemming( using conflation )
```sql
SELECT to_tsquery('english', 'teaching');

 to_tsquery
------------
 'teach'
```
```sql
CREATE TABLE docs (id SERIAL, doc TEXT, PRIMARY KEY(id));

CREATE INDEX gin1 ON docs USING gin(to_tsvector('english', doc));

INSERT INTO docs (doc) VALUES
('This is SQL and Python and other fun teaching stuff'),
('More people should learn SQL from UMSI'),
('UMSI also teaches Python and also SQL');

SELECT id, doc FROM docs WHERE
    to_tsquery('english', 'learn') @@ to_tsvector('english', doc);

 id |                  doc
----+----------------------------------------
  2 | More people should learn SQL from UMSI

EXPLAIN SELECT id, doc FROM docs WHERE
    to_tsquery('english', 'learn') @@ to_tsvector('english', doc);

                                      QUERY PLAN
--------------------------------------------------------------------------------------
 Bitmap Heap Scan on docs  (cost=12.05..23.02 rows=6 width=36)
   Recheck Cond: ('''learn'''::tsquery @@ to_tsvector('english'::regconfig, doc))
   ->  Bitmap Index Scan on gin1  (cost=0.00..12.05 rows=6 width=0)
         Index Cond: ('''learn'''::tsquery @@ to_tsvector('english'::regconfig, doc))
```

## Making NL Inverted Index
- GIN & GiST Index
```sql
CREATE INDEX gin1 ON docs USING gin(to_tsvector('english', doc)); -- GIN
INSERT INTO docs (doc) VALUES
 ('This is SQL and Python and other fun teaching stuff'),
 ('More people should learn SQL from UMSI'),
 ('UMSI also teaches Python and also SQL');

INSERT INTO docs (doc) SELECT 'Neon ' || generate_series(10000,20000);

SELECT id, doc FROM docs WHERE ('english', 'learn') @@ to_tsvector('english', doc);
 id |                  doc                   
----+----------------------------------------
  2 | More people should learn SQL from UMSI
(1 row)

EXPLAIN SELECT id, doc FROM docs WHERE to_tsquery('english', 'learn') @@ to_tsvector('english', doc);
                                      QUERY PLAN                                      
--------------------------------------------------------------------------------------
 Bitmap Heap Scan on docs  (cost=208.27..268.71 rows=35 width=36)
   Recheck Cond: ('''learn'''::tsquery @@ to_tsvector('english'::regconfig, doc))
   ->  Bitmap Index Scan on gin1  (cost=0.00..208.26 rows=35 width=0)
         Index Cond: ('''learn'''::tsquery @@ to_tsvector('english'::regconfig, doc))
(4 rows)

drop index gin1;
CREATE INDEX gist1 ON docs USING gist(to_tsvector('english', doc)); -- GiST                 
EXPLAIN SELECT id, doc FROM docs WHERE to_tsquery('english', 'learn') @@ to_tsvector('english', doc);
                                      QUERY PLAN                                      
--------------------------------------------------------------------------------------
 Bitmap Heap Scan on docs  (cost=4.54..73.90 rows=50 width=15)
   Recheck Cond: ('''learn'''::tsquery @@ to_tsvector('english'::regconfig, doc))
   ->  Bitmap Index Scan on gist1  (cost=0.00..4.53 rows=50 width=0)
         Index Cond: ('''learn'''::tsquery @@ to_tsvector('english'::regconfig, doc))
(4 rows)
```
GiST index is faster than GIN index( lower cost )