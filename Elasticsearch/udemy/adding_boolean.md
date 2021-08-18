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
## How `match` query works
### 2 queries below are equivalent
GET /recipe/_search
{
  "query": {
    "match": {
      "title": "pasta carbonara"
    }
  }
}
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

### equivalent
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
