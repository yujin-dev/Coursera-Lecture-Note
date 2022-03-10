# compound queries
leaf query로 boolean query로 묶어서 표현한 쿼리

## Querying with boolean logic 
### Adding query clauses to `must` 
`must`의 조건을 충족하는 documents를 가져오도록 한다.
```sh
GET /recipe/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        },
        {
          "range": {
            "preparation_time_minutes": {
              "lte": 15
            }
          }
        }
        ]
    }
  }
}
```

### Moving `range` query to `filter`
`filter`에 해당하는 조건은 relevance score와 관련없다.
```sh
GET /recipe/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "preparation_time_minutes": {
              "lte": 15
            }
          }
        }
      ]
    }
  }
}
```

### Adding query clauses to `must_not` 
`tuna`는 match되지 않도록 한다.
```sh
GET /recipe/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "ingredients.name": "tuna"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "preparation_time_minutes": {
              "lte": 15
            }
          }
        }
      ]
    }
  }
}
```
### Adding query clause to `should` 
반드시 있어야하진 않지만 `should`에 해당되는 조건이 있으면 score가 더 높다. 아래 2가지 쿼리에 대해 반환되는 documents 수는 동일하지만 `max_score`가 다르게 측정된다.

```sh
GET /recipe/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "ingredients.name": "tuna"
          }
        }
      ],
      "should": [
        {
          "match":{
            "ingredients.name": "parsley"
          }
        }
        ],
      "filter": [
        {
          "range": {
            "preparation_time_minutes": {
              "lte": 15
            }
          }
        }
      ]
    }
  }
}
```
### `should` query clauses
```sh
GET /recipe/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "pasta"
          }
        }
      ],
      "should": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        }
      ]
    }
  }
}
```
```sh
  "hits" : {
    "total" : {
      "value" : 6,
      "relation" : "eq"
    },
    "max_score" : 2.759146,
...
```

```sh
GET /recipe/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        }
      ]
    }
  }
}
```
```sh
  "hits" : {
    "total" : {
      "value" : 6,
      "relation" : "eq"
    },
    "max_score" : 1.379573,
...
```
## Debugging `bool` queries with named queries
 각 쿼리에 이름을 설정하여 쿼리 매칭이 되었는지 확인할 수 있다.
```sh
GET /recipe/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": {
              "query": "parmesan",
              "_name": "parmesan_must"
            }
          }
        }
        ],
        "must_not": [
          {
            "match": {
              "ingredients.name": {
                "query": "tuna",
                "_name": "tuna_must_not"
              }
            }
          }
          ],
          "should": [
            {
              "match": {
                "ingredients.name": {
                  "query": "parsley",
                  "_name": "parsley_should"
                }
              }
            }
            ],
          "filter": [
            {
              "range": {
                "preparation_time_minutes": {
                  "lte": 15,
                  "_name": "prep_time_filter"
                }
              }
            }
            ]
    }
  }
}
```
아래와 같이 2개의 결과가 반환되었는데 각각 어떤 쿼리에 매칭되는지 보여준다.
```sh
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
  ...
```
score가 더 높은 documents는 `parsley_should`가 포함되었음을 알 수 있다. 
```sh
    "matched_queries" : [
      "prep_time_filter",
      "parmesan_must",
      "parsley_should"
    ]
    ...
    "matched_queries" : [
    "prep_time_filter",
    "parmesan_must"
  ]
```


## How `match` query works
아래 쿼리는 2가지 `term` 쿼리를 `should`로 엮은 compound 쿼리와 동일하다.  
```sh
GET /recipe/_search
{
  "query": {
    "match": {
      "title": "pasta carbonara"
    }
  }
}
```
```sh
  "hits" : {
    "total" : {
      "value" : 9,
      "relation" : "eq"
    },
    "max_score" : 4.548811,
```
따라서 아래의 쿼리와 동일한데 같은 결과를 반환함을 알 수 있다.
```sh
GET /recipe/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "title": "pasta"
          }
        },
        {
          "term": {
            "title": "carbonara"
          }
        }
        ]
    }
  }
}
```
```sh
  "hits" : {
    "total" : {
      "value" : 9,
      "relation" : "eq"
    },
    "max_score" : 4.548811,
```
다음 쿼리는 정확한 단어를 포함하는지에 대한 여부로 `term` 쿼리를 `must` 로 엮은 compound 쿼리로 나타낼 수 있다. 
```sh
GET /recipe/_search
{
  "query": {
    "match": {
      "title": {
        "query": "pasta carbonara",
        "operator": "and"
      }
    }
  }
}
```
```sh
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 4.548811,
```
즉, 아래와 같이
```sh
GET /recipe/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "title": "pasta"
          }
        },
        {
          "term": {
            "title": "carbonara"
          }
        }
        ]
    }
  }
}
```
```sh
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 4.548811,
```

여기서 `match`쿼리르 사용하는 경우 analyze가 적용되지만 `term`쿼리는 raw 값 자체로 비교하기에 대문자가 포함된 단어를 대상으로 하면 서로 다른 결과를 반환한다.