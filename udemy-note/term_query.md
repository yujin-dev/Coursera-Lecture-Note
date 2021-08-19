## Searching for a term
### Matching documents for {`is_active`:`true`}
```sh
GET /products/_search
{
  "query": {
    "term": {
      "is_active": true
    }
  }
}
```
```sh
GET /products/_search
{
  "query": {
    "term": {
      "is_active": {
        "value": true
      }
    }
  }
}
```
## Searching for multiple terms
```sh
GET /products/_search
{
  "query": {
    "terms": {
      "tags.keyword": [
        "Soup",
        "Cake"
      ]
    }
  }
}
```
## Retrieving documents based on IDs
ID가 1,2,3 인 document만 가져온다.
```sh
GET /products/_search
{
  "query": {
    "ids":{
      "values": [1,2,3]
    }
  }
}
```

## Matching documents with range values
### Matching documents for `is_stock` between 1 and 5
`in_stock`이 1이상 5이하인 documents만 가져온다.
```sh
GET /products/_search
{
  "query": {
    "range": {
      "in_stock": {
        "gte": 1,
        "lte": 5
      }
    }
  }
}
```

### Matching documents with a date range
```sh
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "2010/01/01",
        "lte": "2010/12/31"
      }
    }
  }
}
```
### Matching documents with a date range , custom date format 
```sh
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "01-01-2010",
        "lte": "31-12-2010",
        "format": "dd-MM-yyyy"
      }
    }
  }
}
```
## Working with relative dates
### Subtracting 1 year from `2010/01/01`
`||`는 앞의 날짜가 anchor dates를 의미한다. 
```sh
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "2010/01/01||-1y"
      }
    }
  }
}
```

### Subtracting 1year, 1day from `2010/01/01`
```sh
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "2010/01/01||-1y-1d"
      }
    }
  }
}
```
### Subtracting 1year from `2010/01/01` , rounding by month
`2010/01/01` 기준으로 월초나 월말로 맞춘다.   
`2010-01-20` 경우 해당 날짜가 포함되냐에 따라 아래와 같이 맞춰진다.
- `gt`: `2010-01-31` 기준으로 보다 큰
- `gte`: `2010-01-01` 기준으로 이상
- `lt`: `2010-01-01` 기준으로 보다 작은
- `lte` : `2010-01-31` 기준으로 이하
```sh
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "2010/01/01||-1y/M"
      }
    }
  }
}
```
### Rounding by month before subtracting 1 year from `2010/01/01`
```sh
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "2010/01/01||/M-1y"
      }
    }
  }
}
```
### Rounding monthly before subtracting 1 year from the current date
```sh
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "now/M-1y"
      }
    }
  }
}
```
### Matching documents with `created` field containing the current date or later
```sh
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "now"
      }
    }
  }
}
```

## Matching documents with non-null values
적어도 하나 이상의 non null 값이 있어햐 하는 경우의 쿼리이다.
```sh
GET /products/_search
{
  "query": {
    "exists": {
      "field": "tags"
    }
  }
}
```
## Matching based on prefixes
### Matching documents containing a tag beginning with `Vege`
`tag` field의 keyword mapping에 따라 시작 글자가 `Vege`인 documents만 불러온다.
```sh
GET /products/_search
{
  "query": {
    "prefix": {
      "tags.keyword": "Vege"
    }
  }
}
```
## Searching with wildcards
### Adding an asterisk for any characters
`*`는 empty character까지 포함하여 아무 문자나 들어있으면 해당된다.
```sh
GET /products/_search
{
  "query": {
    "wildcard": {
      "tags.keyword": "Veg*ble"
    }
  }
}
```
### Adding a question mark for any single character
```sh
GET /products/_search
{
  "query": {
    "wildcard": {
      "tags.keyword": "Veget?ble"
    }
  }
}
```
## Searching with regular expressions
알파벳 소대문자 포함하여 아무 문자가 하나는 있어야 하는 쿼리이다.
```sh
GET /products/_search
{
  "query": {
    "regexp": {
      "tags.keyword": "Veg[a-zA-Z]+ble"
    }
  }
}
```