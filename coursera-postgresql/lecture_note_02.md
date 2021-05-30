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


## Inverted Index
```sql
SELECT string_to_array('Hello world', ' ');
/*
 string_to_array
-----------------
 {Hello,world}
*/
```
```sql
SELECT unnest(string_to_array('Hello world', ' '));-- array(horizontal) to rows( vertical )
/*
 unnest
--------
 Hello
 world
 */
 ```
 
