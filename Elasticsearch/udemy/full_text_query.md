## Flexible matching with the match query
### Standard `match` 
```sh
GET /recipe/_serach
{
  "query": {
    "match": {
      "title": "Recipes with pasta or spaghetti"
    }
  }
}
```
해당 쿼리에서는 `title` 조건에 나오는 모든 단어가 포함되지 않아도 된다.

### Specifying a boolean operator
`boolean` operator를 포함하면 검색하려는 단어가 전부 포함되야 한다. 
```sh
GET /recipe/_search
{
  "query": {
    "match": {
      "title": {
        "query": "Recipes with pasta or spaghetti",
        "operator": "and"
      }
    }
  }
}
```
```sh
GET /recipe/_search
{
  "query": {
    "match": {
        "title": {
          "query": "pasta spaghetti",
          "operator": "and"
      }
    }
  }
}
```
## Matching phrases
`term` query와 같은 동작으로 검색하려는 단어와 일치되는 경우가 포함된 documents를 불러온다. 
따라서 아래 2개의 쿼리는 서로 다른 결과를 반환한다.
```sh 
GET /recipe/_search
{
  "query": {
    "match_phrase": {
      "title": "spaghetti puttanesca"
    }
  }
}
```
```sh
GET /recipe/_search
{
  "query": {
    "match_phrase": {
      "title": "puttanesca spaghetti"
    }
  }
}
```
## Searching multiple fields
여러 field에 매치되는 결과가 있는 검색한다.
```sh
GET /recipe/_search
{
  "query": {
    "multi_match": {
      "query": "pasta",
      "fields": ["title", "description"]
    }
  }
}
```
 