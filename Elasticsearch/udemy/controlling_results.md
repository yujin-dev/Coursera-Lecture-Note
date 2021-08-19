# Controlling Query Results
## Specifying the result format
### Return results as YAML
```sh
GET /recipe/_search?format=yaml
{
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}
```

### Return pretty JSON
terminal에서 결과를 확인할 때 유용하다.
```
GET /recipe/_search?pretty
{
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}
```
## Source filtering
### Exclude the `_source` field
```sh
GET /recipe/_search
{
  "_source": false,
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}
```
### Only return the `created` field
`_source`의 `created`만 반환한다.
```
GET /recipe/_search
{
  "_source": "created",
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}
```
```sh
    "_source" : {
          "created" : "2002/01/04"
        }
      },
      ...
```
### Only return `ingredients` object's key
```sh
GET /recipe/_search
{
  "_source": "ingredients.name",
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}
```
```sh
        "_source" : {
          "ingredients" : [
            {
              "name" : "Silken tofu"
            },
            {
              "name" : "Sauerkraut brine"
            },
        ...
```
### Return all of `ingredients` object's keys
```sh
GET /recipe/_search
{
  "_source": "ingredients.*",
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}
```
```sh
        "_source" : {
          "ingredients" : [
            {
              "quantity" : "200g",
              "name" : "Silken tofu"
            },
            {
              "quantity" : "120ml",
              "name" : "Sauerkraut brine"
            },
        ...
```
### Return the `ingredients` object with all keys, and the `servings` field
```sh
GET /recipe/_search
{
  "_source": ["ingredients.*", "servings"],
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}
```
```sh
    "_source" : {
        "servings" : {
        "min" : 4,
        "max" : 4
        },
        "ingredients" : [
        {
            "quantity" : "200g",
            "name" : "Silken tofu"
        },
    ...
```

### Include all of the `ingredients` object's keys, except the `name` key
```sh
GET /recipe/_search
{
  "_source": {
    "includes": "ingredients.*",
    "excludes": "ingredients.name"
  },
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}
```
```sh
    "_source" : {
        "ingredients" : [
        {
            "quantity" : "200g"
        },
        {
            "quantity" : "120ml"
        },
    ...
```

## Specifying the result size
### Using a query parameter
GET /recipe/_search?size=2
{
  "_source": false,
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}
documents 2개까지만 반환한다.
```sh
  "hits" : {
    "total" : {
      "value" : 9,
      "relation" : "eq"
    },
    "max_score" : 1.0835493,
    "hits" : [
      {
        "_index" : "recipe",
        "_type" : "_doc",
        "_id" : "8",
        "_score" : 1.0835493,
        "_ignored" : [
          "description.keyword",
          "steps.keyword"
        ]
      },
      {
        "_index" : "recipe",
        "_type" : "_doc",
        "_id" : "18",
        "_score" : 0.8755694,
        "_ignored" : [
          "description.keyword",
          "steps.keyword"
        ]
      }
    ]
  }
```

### Using a parameter within the requets body
위와 동일한 결과를 반환한다.
```sh
GET /recipe/_search
{
  "_source": false,
  "size": 2,
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}
```
## Specifying an offset
pagination으로 다음 documents를 가져올 수 있다.
### Specifying an offset with the `from` parameter
```sh
GET /recipe/_search
{
  "_source": false,
  "size": 2,
  "from": 2,
  "query": {
    "match": {
      "title": "pasta"
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
    "max_score" : 1.0835493,
    "hits" : [
      {
        "_index" : "recipe",
        "_type" : "_doc",
        "_id" : "13",
        "_score" : 0.8229182,
        "_ignored" : [
          "steps.keyword"
        ]
      },
      {
        "_index" : "recipe",
        "_type" : "_doc",
        "_id" : "19",
        "_score" : 0.8229182,
        "_ignored" : [
          "steps.keyword"
        ]
      }
    ]
  }
```
## Sorting results
### Sorting by ascending order
```sh
GET /recipe/_search
{
  "_source": false,
  "query": {
    "match_all": {}
  },
  "sort": [
    "preparation_time_minutes"
    ]
}
```
```sh
    "hits" : [
      {
        "_index" : "recipe",
        "_type" : "_doc",
        "_id" : "5",
        "_score" : null,
        "_ignored" : [
          "description.keyword",
          "steps.keyword"
        ],
        "sort" : [
          8
        ]
      },
      {
        "_index" : "recipe",
        "_type" : "_doc",
        "_id" : "6",
        "_score" : null,
        "_ignored" : [
          "steps.keyword"
        ],
        "sort" : [
          10
        ]
      },
      ...
```
### Sorting by descending order
```sh
GET /recipe/_search
{
  "_source": "created",
  "query": {
    "match_all": {}
  },
  "sort": [
    { "created": "desc" }
  ]
}
```
```sh
   "hits" : [
      {
        "_index" : "recipe",
        "_type" : "_doc",
        "_id" : "12",
        "_score" : null,
        "_ignored" : [
          "description.keyword",
          "steps.keyword"
        ],
        "_source" : {
          "created" : "2017/05/20"
        },
        "sort" : [
          1495238400000
        ]
      },
      {
        "_index" : "recipe",
        "_type" : "_doc",
        "_id" : "10",
        "_score" : null,
        "_ignored" : [
          "steps.keyword"
        ],
        "_source" : {
          "created" : "2017/04/27"
        },
        "sort" : [
          1493251200000
        ]
      },
      ...
```

## Sorting by multi-value fields
### Sorting by the average rating(descending)
```sh
GET /recipe/_search
{
  "_source": "ratings",
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "ratings": {
        "order": "desc",
        "mode": "avg"
      }
    }
    ]
}
```
```sh
    "hits" : [
      {
        "_index" : "recipe",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : null,
        "_ignored" : [
          "description.keyword",
          "steps.keyword"
        ],
        "_source" : {
          "ratings" : [
            5.0
          ]
        },
        "sort" : [
          5.0
        ]
      },
      {
        "_index" : "recipe",
        "_type" : "_doc",
        "_id" : "5",
        "_score" : null,
        "_ignored" : [
          "description.keyword",
          "steps.keyword"
        ],
        "_source" : {
          "ratings" : [
            5.0,
            5.0,
            4.5,
            5.0,
            4.0,
            5.0
          ]
        },
        "sort" : [
          4.75
        ]
      },
    ...
```

## Filters
### Adding a `filter` clause to the `bool` query
`filter` 쿼리는 Yes or No의 문제로 score와 연관이 없지만 documents를 거르는 방식으로 적용된다. 
```sh
GET /recipe/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": "pasta"
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
```sh
        "_source" : {
          "title" : "Pesto Pasta With Potatoes and Green Beans",
          "description" : "This classic Genovese method of preparing pasta with pesto includes cubes of potato and pieces of green bean, all cooked together in the pasta pot until tender.",
          "preparation_time_minutes" : 15,
          "servings" : {
            "min" : 4,
            "max" : 4
          },
        ...
```

+ES에서는 자주 쓰이는 퀴리를 캐싱하여 적용한다.