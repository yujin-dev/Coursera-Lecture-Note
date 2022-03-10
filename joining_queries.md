# Joining Queries
RDBMS 에서는 정규화를 하여 join을 이용하는데 ES에서는 성능을 위해 denormalize를 하는 편이다. 하지만 ES에서도 join과 같은 nested query가 존재한다.

## Querying nested objects  
### Create the index with mapping
```sh
PUT /department
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text"
      },
      "employees": {
        "type": "nested"
      }
    }
  }
}
```
### Querying nested fields
`nested` data type에 대한 쿼리는 `nested` 를 적용하여 검색한다. 
```sh
GET /department/_search
{
  "_source": false, 
  "query": {
    "nested": {
      "path": "employees",
      "query": {
        "bool": {
          "must": [
            {
              "match": {
                "employees.position": "intern"
              }
            },
            {
              "term": {
                "employees.gender.keyword": {
                  "value": "F"
                }
              }
            }
            ]
        }
      }
    }
  }
}
```
```sh
    "hits" : [
      {
        "_index" : "department",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 2.3905568
      },
      {
        "_index" : "department",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 2.3905568
      }
    ]
```
## Nested inner hits
매칭된 각 documents의 `employees` 내부적으로 각각에 대해 score를 구할 수 있다.
```sh
GET /department/_search
{
  "_source": false,
  "query": {
    "nested": {
      "path": "employees",
      "inner_hits": {}, 
      "query": {
        "bool": {
          "must": [
            {
              "match": {
                "employees.position": "intern"
              }
            },
            {
            "term": {
              "employees.gender.keyword": {
                "value": "F"
              }
            }
            }
          ]
        }
      }
    }
  }
}
```
```sh
   "hits" : [
      {
        "_index" : "department",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 2.3905568,
        "inner_hits" : {
          "employees" : {
            "hits" : {
              "total" : {
                "value" : 1,
                "relation" : "eq"
              },
              "max_score" : 2.3905568,
              "hits" : [
                {
                  "_index" : "department",
                  "_type" : "_doc",
                  "_id" : "1",
                  "_nested" : {
                    "field" : "employees",
                    "offset" : 3
                  },
                  "_score" : 2.3905568,
                  "_source" : {
                    "gender" : "F",
                    "name" : "Julie Powell",
                    "position" : "Intern",
                    "age" : 26
                  }
                }
              ]
            }
          }
        }
      },
      {
        "_index" : "department",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 2.3905568,
        "inner_hits" : {
          "employees" : {
            "hits" : {
              "total" : {
                "value" : 2,
                "relation" : "eq"
              },
              "max_score" : 2.3905568,
              "hits" : [
                {
                  "_index" : "department",
                  "_type" : "_doc",
                  "_id" : "2",
                  "_nested" : {
                    "field" : "employees",
                    "offset" : 2
                  },
                  "_score" : 2.3905568,
                  "_source" : {
                    "gender" : "F",
                    "name" : "Margaret Harris",
                    "position" : "Intern",
                    "age" : 19
                  }
                },
                {
                  "_index" : "department",
                  "_type" : "_doc",
                  "_id" : "2",
                  "_nested" : {
                    "field" : "employees",
                    "offset" : 7
                  },
                  "_score" : 2.3905568,
                  "_source" : {
                    "gender" : "F",
                    "name" : "Evelyn Henderson",
                    "position" : "Intern",
                    "age" : 24
                  }
                }
              ]
            }
```
## Mapping document relationships
document 관계에 대해 `join`으로 data type을 설정하여 `parent: child` 형식으로 정의할 수 있다.
```sh
PUT /department/_mapping
{
  "properties": {
    "join_field": {
      "type": "join",
      "relations": {
        "department": "employee"
      }
    }
  }
}
```

### Adding documents
위에서 설정한 field명에 따라 `department` documents를 추가한다.
```sh
PUT /department/_doc/1
{
  "name": "Development",
  "join_field": "department"
}

PUT /department/_doc/2
{
  "name": "Marketing",
  "join_field": "department"
}
```
`employee`를 추가할 때는 parent ID를 routing 파라미터를 설정한다.
```sh
PUT /department/_doc/3?routing=1
{
  "name": "Bo Anderson",
  "age": 28,
  "gender": "M",
  "join_field": {
    "name": "employee",
    "parent":1
  }
}
PUT /department/_doc/4?routing=2
{
  "name": "John Doe",
  "age": 44,
  "gender": "M",
  "join_field": {
    "name": "employee",
    "parent": 2
  }
}
```

## Querying by parent
```sh
GET /department/_search
{
  "query": {
    "parent_id": {
      "type": "employee",
      "id":1
    }
  }
}
```
```sh
  "hits" : {
    "total" : {
      "value" : 4,
      "relation" : "eq"
    },
    "max_score" : 0.4248832,
```
`id = 2`인 경우 위와 다른 결과를 반환한다.
```sh
GET /department/_search
{
  "query": {
    "parent_id": {
      "type": "employee",
      "id":2
    }
  }
}
```
```sh
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 1.060872,
    "hits" : [
```

## Querying child documents by parent
### Matching child documents by parent criteria
`has_parent`를 통해 특정 parent ID를 지정하지 않아도 검색할 수 있다.
```sh
GET /department/_search
{
  "query": {
    "has_parent": {
      "parent_type": "department",
      "query": {
        "term": {
          "name.keyword": "Development"
        }
      }
    }
  }
}
```
## Querying parent by child documents
### Finding parents with child documents matching a bool query
마찬가지로 `has_child`로 조건에 맞는 child를 가진 parents documents를 검색할 수 있다.
```sh
GET /department/_search
{
  "query": {
    "has_child": {
      "type": "employee",
      "query": {
        "bool": {
          "must": [
            {
              "range": {
                "age": {
                  "gte": 50
                }
              }
            }
          ],
          "should": [
            {
              "term": {
                "gender.keyword": "M"
              }
            }
          ]
        }
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
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "department",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "name" : "Development",
          "join_field" : "department"
        }
      }
    ]
  }
```

### Taking relevance scores into account with score_mode
child documents의 score 합계를 산정한다.
```sh
GET /department/_search
{
  "query": {
    "has_child": {
      "type": "employee",
      "score_mode": "sum",
      "query": {
        "bool": {
          "must": [
            {
              "range": {
                "age": {
                  "gte": 50
                }
              }
            }
          ],
          "should": [
            {
              "term": {
                "gender.keyword": "M"
              }
            }
          ]
        }
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
    "max_score" : 1.4418328,
    "hits" : [
      {
        "_index" : "department",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.4418328,
        "_source" : {
          "name" : "Development",
          "join_field" : "department"
        }
      }
    ]
  }
```
### Specifying the minimum and maximum number of children
매치되는 child documents 갯수의 범위를 산정한다.
```sh
GET /department/_search
{
  "query": {
    "has_child": {
      "type": "employee",
      "score_mode": "sum",
      "min_children": 2,
      "max_children": 5,
      "query": {
        "bool": {
          "must": [
            {
              "range": {
                "age": {
                  "gte": 50
                }
              }
            }
          ],
          "should": [
            {
              "term": {
                "gender.keyword": "M"
              }
            }
          ]
        }
      }
    }
  }
}

```
매치되는 결과가 1개 뿐이였기에 반환값이 없다.
```sh
  "hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  }
```
## Multi-level relations
여러 단계의 relations를 나타내는 mappings를 생성한다.
### Creating the index with mapping
- `employee` -> `department` -> `company`
- `supplier` -> `company`
```sh
PUT /company
{
  "mappings": {
    "properties": {
      "join_field": {
        "type": "join",
        "relations": {
          "company": ["department", "supplier"],
          "department": "employee"
        }
      }
    }
  }
}
```

### Querying multi-level relations
`department`의 child 중 `employee` child에서 `name = John Doe` 인 document를 검색한다.
```sh
GET /company/_search
{
  "query": {
    "has_child": {
      "type": "department",
      "query": {
        "has_child": {
          "type": "employee",
          "query": {
            "term": {
              "name.keyword": "John Doe"
            }
          }
        }
      }
    }
  }
}
```
`index` 가 `company`인 documents를 반환한다.
```sh
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "company",
        "_type" : "_doc",
        "_id" : "4",
        "_score" : 1.0,
        "_source" : {
          "name" : "Another Company, Inc.",
          "join_field" : "company"
        }
      }
    ]
  }
```
## Parent/child inner hits
### Including inner hits for `has_child` query
조건에 맞는 child hits score 및 documents를 반환한다.
```sh
GET /department/_search
{
  "query": {
    "has_child": {
      "type": "employee",
      "inner_hits": {}, 
      "query": {
        "bool": {
          "must": [
            {
              "range": {
                "age": {
                  "gte": 50
                }
              }
            }
            ],
            "should": [
              {
                "term": {
                  "gender.keyword": "M"
                }
              }
            ]
        }
      }
    }
  }
}
```
```sh
        "inner_hits" : {
          "employee" : {
            "hits" : {
              "total" : {
                "value" : 1,
                "relation" : "eq"
              },
              "max_score" : 1.4418328,
              "hits" : [
                {
                  "_index" : "department",
                  "_type" : "_doc",
                  "_id" : "6",
                  "_score" : 1.4418328,
                  "_routing" : "1",
                  "_source" : {
                    "name" : "Daniel Harris",
                    "age" : 52,
                    "gender" : "M",
                    "join_field" : {
                      "name" : "employee",
                      "parent" : 1
                    }
                  }
                }
              ]
            }
          }
        }
```

### Including inner hits for `has_parent` query
마찬가지로 `has_parent`도 적용 가능하다.
```sh
GET /department/_search
{
  "query": {
    "has_parent": {
      "inner_hits": {},
      "parent_type": "department", 
      "query": {
        "term": {
          "name.keyword": "Development"
        }
      }
    }
  }
}
```

## Terms lookup mechanism 
### Querying stories from a user's followers
```sh
GET /stories/_search
{
  "query": {
    "terms": {
      "user": {
        "index": "users",
        "id": "1",
        "path": "following"
      }
    }
  }
}
```
sample data를 추가하면 
```sh
PUT /users/_doc/1
{
  "name": "John Roberts",
  "following" : [2, 3]
}
...
PUT /stories/_doc/1
{
  "user": 3,
  "content": "Wow look, a penguin!"
}
...
PUT /stories/_doc/6
{
  "user": 2,
  "content": "Chilling at the beach @ Greece #vacation #goodtimes"
```
`stories`의 `term`쿼리를 위해 `user` index를 내부적으로 타고 `GET`을 실행한다.   

```sh
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "stories",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "user" : 3,
          "content" : "Wow look, a penguin!"
        }
      },
      {
        "_index" : "stories",
        "_type" : "_doc",
        "_id" : "6",
        "_score" : 1.0,
        "_source" : {
          "user" : 2,
          "content" : "Chilling at the beach @ Greece #vacation #goodtimes"
        }
      }
    ]
  }
```