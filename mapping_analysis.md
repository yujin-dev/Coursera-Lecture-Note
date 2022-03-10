## Using Analyze API
### `standard` analyzer로 string 분석
```sh
POST /_analyze
{
  "text": "2 guys walk into   a bar, but the third... DUCKS! :-)",
  "analyzer": "standard"
}
```
위와 동일한 로직으로 적용된다.
```sh
POST /_analyze
{
  "text": "2 guys walk into   a bar, but the third... DUCKS! :-)",
  "char_filter": [],
  "tokenizer": "standard",
  "filter":["lowercase"]
}
```

## Understanding type coercion
### floating point
```sh
PUT /coercion_test/_doc/1
{
  "price": 7.4
}
```
### floating point withing a string
PUT /coercion_test/_doc/2
{
  "price": "7.4"
}
Apache Lucene에서 data type을 정의하는데 coercion이 string을 float로 변환하여 데이터를 저장한다. 하지만 데이터를 `GET`하면 string으로 불러오는데 이는 index type이 반영된 것이다.

## invalid value
PUT /coercion_test/_doc/3
{
  "price": "7.4m"
}

## Understanding arrays
array라는 개념은 없다고 한다.

### Arrays of strings are concatenated 
```sj
POST /_analyze
{
  "text": ["Strings are simply", "merged together."],
  "analyzer": "standard"
}
```
multiple 값이 아닌 single 값으로 인식한다. []안에 데이터는 모두 동일한 data type이어야 한다. coercion이 적용되기에 [34, 56, "9"] 같은 경우도 가능하긴 하나 정확한 type을 정의하는게 좋다.

```sh
{
  "tokens" : [
    {
      "token" : "strings",
      "start_offset" : 0,
      "end_offset" : 7,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "are",
      "start_offset" : 8,
      "end_offset" : 11,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "simply",
      "start_offset" : 12,
      "end_offset" : 18,
      "type" : "<ALPHANUM>",
      "position" : 2
    },
    {
      "token" : "merged",
      "start_offset" : 19,
      "end_offset" : 25,
      "type" : "<ALPHANUM>",
      "position" : 3
    },
    {
      "token" : "together",
      "start_offset" : 26,
      "end_offset" : 34,
      "type" : "<ALPHANUM>",
      "position" : 4
    }
  ]
}

```

## Adding explicit mappings
### add field mappings for `reviews` index
```sh
PUT /reviews
{
  "mappings": {
    "properties": {
      "rating": { "type": "float" },
      "content": { "type": "text" },
      "product_id": { "type": "integer" },
      "author":{
          "properties": {
            "first_name": { "type": "text" },
            "last_name": { "type": "text" },
            "email": { "type": "keyword" }
      
          }  
      }
    }
  }
}
```
아래를 예시로 추가하면
```sh
PUT /reviews/_doc/1
{
  "rating": 5.0,
  "content": "Outstanding course! Bo really taught me a lot about Elasticsearch!",
  "product_id": 123,
  "author": {
    "first_name": "John",
    "last_name": "Doe",
    "email": {}
  }
}
``` 
email data type이 안 맞기에 error을 발생한다.
```sh
PUT /reviews/_doc/1
{
  "rating": 5.0,
  "content": "Outstanding course! Bo really taught me a lot about Elasticsearch!",
  "product_id": 123,
  "author": {
    "first_name": "John",
    "last_name": "Doe",
    "email": "johndoe123@example.com"
  }
}
```

## Using dot notation in field names
nested json 형식에 보기 쉽게 dot properties를 이용한다.
### Using dot notation for `author`
```sh
PUT /reviews_dot_notation
{
  "mappings": {
    "properties": {
      "rating": { "type": "float" },
      "content": { "type": "text" },
      "product_id": { "type": "integer" },
      "author.first_name": { "type": "text" },
      "author.last_name": { "type": "text" },
      "author.email": { "type": "keyword" }
    }
  }
}
```
결과를 확인하면,
```sh
GET /reviews_dot_notation/_mapping
```
```sh
{
  "reviews_dot_notation" : {
    "mappings" : {
      "properties" : {
        "author" : {
          "properties" : {
            "email" : {
              "type" : "keyword"
            },
            "first_name" : {
              "type" : "text"
            },
            "last_name" : {
              "type" : "text"
            }
          }
        },
        "content" : {
          "type" : "text"
        },
        "product_id" : {
          "type" : "integer"
        },
        "rating" : {
          "type" : "float"
        }
      }
    }
  }
}
```
## Adding mappings to existing indices
### Add new field mapping to existing index
`mapping` API를 이용하여 기존의 index에 mapping을 추가한다.
```sh
PUT /reviews/_mapping
{
  "properties":  {
    "created_at": {
      "type": "date"
    }
  }
}
```
## How dates work
### only a date
```sh
PUT /reviews/_doc/2
{
  "rating": 4.5,
  "content": "Not bad. Not bad at all!",
  "product_id": 123,
  "created_at": "2015-03-27",
  "author": {
    "first_name": "Average",
    "last_name": "Joe",
    "email": "avgjoe@example.com"
  }
}
```
### both a date and time
```sh
PUT /reviews/_doc/3
{
  "rating": 3.5,
  "content": "Could be better",
  "product_id": 123,
  "created_at": "2015-04-15T13:07:41Z",
  "author": {
    "first_name": "Spencer",
    "last_name": "Pearson",
    "email": "spearson@example.com"
  }
}
```
### UTC offset
```sh
PUT /reviews/_doc/4
{
  "rating": 5.0,
  "content": "Incredible!",
  "product_id": 123,
  "created_at": "2015-01-28T09:21:51+01:00",
  "author": {
    "first_name": "Adam",
    "last_name": "Jones",
    "email": "adam.jones@example.com"
  }
}
```
### timestamp
```sh
PUT /reviews/_doc/5
{
  "rating": 4.5,
  "content": "Very useful",
  "product_id": 123,
  "created_at": 1436011284000,
  "author": {
    "first_name": "Taylor",
    "last_name": "West",
    "email": "twest@example.com"
  }
}
```

## Updating existing mappings
보통 field mappings는 업데이트될 수 없다.
```sh
PUT /reviews/_mapping
{
  "properties": {
    "product_id": {
      "type": "keyword"
    }
  }
}
```
따라서 실행하면 오류가 발생한다.

하지만 `ignore_above`는 업데이트될 수 있다.
```sh
PUT /reviews/_mapping
{
  "properties": {
    "author": {
      "properties": {
          "email": {
            "type": "keyword",
            "ignore_above": 256
        }
      }
    }
  }
}
```

## Reindexing documents with the Reindex API
### Add new index with new mapping
```sh
PUT /reviews_new
{
  "mappings" : {
    "properties" : {
      "author" : {
        "properties" : {
          "email" : {
            "type" : "keyword",
            "ignore_above" : 256
          },
          "first_name" : {
            "type" : "text"
          },
          "last_name" : {
            "type" : "text"
          }
        }
      },
      "content" : {
        "type" : "text"
      },
      "created_at" : {
        "type" : "date"
      },
      "product_id" : {
        "type" : "keyword"
      },
      "rating" : {
        "type" : "float"
      }
    }
  }
}
```
### Reindex documents into `reviews_new`
```sh
POST /_reindex
{
  "source": {
    "index": "reviews"
  },
  "dest": {
    "index": "reviews_new"
  }
}
```
```sh
{
  "took" : 14,
  "timed_out" : false,
  "total" : 5,
  "updated" : 0,
  "created" : 5,
  "deleted" : 0,
  "batches" : 1,
  "version_conflicts" : 0,
  "noops" : 0,
  "retries" : {
    "bulk" : 0,
    "search" : 0
  },
  "throttled_millis" : 0,
  "requests_per_second" : -1.0,
  "throttled_until_millis" : 0,
  "failures" : [ ]
}

```
5 document가 새로 index 되었다. 전체 결과를 확인하면
```sh
  "hits" : {
    "total" : {
      "value" : 5,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "reviews_new",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "rating" : 5.0,
          "content" : "Outstanding course! Bo really taught me a lot about Elasticsearch!",
          "product_id" : 123,
          "author" : {
            "first_name" : "John",
            "last_name" : "Doe",
            "email" : "johndoe123@example.com"
          }
        }
      },
      {
        "_index" : "reviews_new",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "rating" : 4.5,
          "content" : "Not bad. Not bad at all!",
          "product_id" : 123,
...
```
`product_id`가 integer로 적용되었다.

reindex를 다시 정의하기 위해서는 기존의 것을 삭제해야 한다. Query API로 삭제 가능하다.
```sh
POST /reviews_new/_delete_by_query
{
  "query": {
    "match_all": {}
  }
}
```
### Convert `product_id` values to strings
```sh
POST /_reindex
{
  "source": {
    "index": "reviews"
  },
  "dest": {
    "index": "reviews_new"
  },
  "script": {
    "source": """
      if (ctx._source.product_id != null) {
        ctx._source.product_id = ctx._source.product_id.toString();
      }
    """
  }
}
```
`product_id`를 index를 새로 정의하여 검색 결과를 확인하다.

```sh
...
  "hits" : {
    "total" : {
      "value" : 5,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "reviews_new",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "author" : {
            "last_name" : "Doe",
            "first_name" : "John",
            "email" : "johndoe123@example.com"
          },
          "product_id" : "123",
          "rating" : 5.0,
          "content" : "Outstanding course! Bo really taught me a lot about Elasticsearch!"
        }
      },
      {
  ...
```
### Reindex specific documents
```sh
POST /_reindex
{
  "source": {
    "index": "reviews",
    "query": {
      "match_all": { }
    }
  },
  "dest": {
    "index": "reviews_new"
  }
}
```
### Reindex only positive reviews
```sh
POST /_reindex
{
  "source": {
    "index": "reviews",
    "query": {
      "range": {
        "rating": {
          "gte": 4.0
        }
      }
    }
  },
  "dest": {
    "index": "reviews_new"
  }
}
```

### Removing fields
`_source` 파라미터를 통해 정해진 field만 reindex 가능하다.
```sh
POST /_reindex
{
  "source": {
    "index": "reviews",
    "_source":  ["content", "created_at", "rating"]
  },
  "dest": {
    "index": "reviews_new"
  }
}
```

### Changing field's name : "content" -> "comment"
```sh
POST /_reindex
{
  "source": {
    "index": "reviews"
  },
  "dest": {
    "index": "reviews_new"
  },
  "script": {
    "source": """
    ctx._source.comment = ctx._source.remove("content");
    """
  }
}
```

### Ignore review with ratings below 4.0
`noop`를 통해 document를 index에 포함시키지 않을 수 있다. `ctx.op` 값을 변경하여 적용 가능하다.
```sh
POST /_reindex
{
  "source": {
    "index": "reviews"
  },
  "dest": {
    "index": "reviews_new"
  },
  "script": {
    "source": """
      if (ctx._source.rating < 4.0) {
        ctx.op = "noop"; # "delete"
      }
    """
  }
}
```
## Define field aliases
### Add `comment` alias pointing to the `content` field
```sh
PUT /reviews/_mapping
{
  "properties": {
    "comment": {
      "type": "alias",
      "path": "content"
    }
  }
}
```
field alias로 `match` 검색 결과를 불러온다.
```sh
GET /reviews/_search
{
  "query": {
    "match": {
      "comment": "outstanding"
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
    "max_score" : 0.9116392,
    "hits" : [
      {
        "_index" : "reviews",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.9116392,
        "_source" : {
          "rating" : 5.0,
          "content" : "Outstanding course! Bo really taught me a lot about Elasticsearch!",
          "product_id" : 123,
          "author" : {
            "first_name" : "John",
            "last_name" : "Doe",
            "email" : "johndoe123@example.com"
          }
        }
      }
    ]
  }
}
```

## Multi-field mappings
```sh
PUT /multi_field_test
{
  "mappings": {
    "properties": {
      "description": {
        "type": "text"
      },
      "ingredients": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

`text` type은 analyzer를 거쳐 변환되어 저장되지만 `ingredients.keyword`는 `keyword` type으로 변환되지 않는다.

예시를 추가한다.
```sh
POST /multi_field_test/_doc
{
  "description": "To make this spaghetti carbonara, you first need to...",
  "ingredients": ["Spaghetti", "Bacon", "Eggs"]
}
```
`text` mapping을 통해 쿼리 요청하여 결과를 확인하면  
```sh
GET /multi_field_test/_search
{
  "query": {
    "match": {
      "ingredients": "Spaghetti"
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
        "_index" : "multi_field_test",
        "_type" : "_doc",
        "_id" : "y6H2SHsBbsvJPDrGdDoV",
        "_score" : 1.0,
        "_source" : {
          "description" : "To make this spaghetti carbonara, you first need to...",
          "ingredients" : [
            "Spaghetti",
            "Bacon",
            "Eggs"
          ]
        }
      }
    ]
  }
}
```
`description`은 변환된 데이터를 불러오지만 `ingredients`는 raw 값의 keyword를 포함한다. 

### Querying the `kerword` mapping
`keyword`의 정확한 값을 찾아 `term` 쿼리를 요청하여 결과를 확인하면
```sh
GET /multi_field_test/_search
{
  "query": {
    "term": {
      "ingredients.keyword": "Spaghetti"
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
    "max_score" : 0.39556286,
    "hits" : [
      {
        "_index" : "multi_field_test",
        "_type" : "_doc",
        "_id" : "y6H2SHsBbsvJPDrGdDoV",
        "_score" : 0.39556286,
        "_source" : {
          "description" : "To make this spaghetti carbonara, you first need to...",
          "ingredients" : [
            "Spaghetti",
            "Bacon",
            "Eggs"
          ]
        }
      }
    ]
  }
}
```
`match` 쿼리와 `term` 쿼리의 `_score`값이 다른 것을 알 수 있다.

## Index templates
주로 wildcards를 포함하는 template으로 구성하게 된다. wildcard 형식에 따라 index를 자동으로 생성하여 monthly, daily basis로 index를 만들 수 있다.

## Adding an index template names `access-logs`
```sh
PUT /_template/access-logs
{
  "index_patterns": ["access-logs-*"], # 
  "settings": {
    "number_of_shards": 2,
    "index.mapping.coerce": false
  }, 
  "mappings": {
    "properties": {
      "@timestamp": {
        "type": "date"
      },
      "url.original": {
        "type": "keyword"
      },
      "http.request.referrer": {
        "type": "keyword"
      },
      "http.response.status_code": {
        "type": "long"
      }
    }
  }
}
```
### Adding an index matching the index template's pattern
```sh
PUT /access-logs-2020-01-01
```
### Verify : the mapping is applied
```sh
GET /access-logs-2020-01-01
```
`access-logs-2020-01-01` 정보를 불러온다.

## Elastic Common Schema
ECS는 `@timestamp` 같은 field는 event source(PostgresSQL, Kafka등에 따라 상관없이)가 무엇이든간에 동일한 schema임을 의미한다.

## dynamic mapping
`mapping`이 정의되지 않은 index에 document를 넣으면 자동으로 `mapping`이 형성되어 data type 같은 정보가 생성된다. 기본적으로 number가 포함된 데이터는 넣을 수 있는 수치를 최대한으로 잡기 위해 `long`이 입력되고 `text` type은 `field: {'type': 'keyword'}`가 포함된다.

## Combining explicit and dynamic mapping
### Create index with 1 field mapping
```sh
PUT /people
{
  "mappings": {
    "properties": {
      "first_name": {
        "type": "text"
      }
    }
  }
}
```
## Index a test document with an unmapped field
```sh
POST /people/_doc
{
  "first_name": "Bo",
  "last_name": "Andersen"
}
```
`GET /people/_mapping` 를 실행하여 `mapping` 정보를 확인하면
```sh
{
  "people" : {
    "mappings" : {
      "properties" : {
        "first_name" : {
          "type" : "text"
        },
        "last_name" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256 # 256자 이상인 text는 무시됨
            } 
          }
        }
      }
    }
  }
}
```
`last_name`에 대해서는 정의하지 않아 자동으로 생성되었음을 알 수 있다.

## Configuring dynamic mapping
### Disable dynamic mapping
```sh
PUT /people
{
  "mappings": {
    "dynamic": false,
    "properties": {
      "first_name": {
        "type":"text"
      }
    }
  }
}
```
```sh
POST /people/_doc
{
  "first_name": "Bo",
  "last_name": "Andersen"
}
```
를 실행해도 `last_name`에 대해 `mapping`이 생성되지 않음을 확인할 수 있다. `last_name`에 대해서는 `_source`에 존재하지만 inverted index가 적용되지 않는다. 

## Set dynamic mapping to `strict`
```sh
PUT /people
{
  "mappings": {
    "dynamic": "strict",
    "properties": {
      "first_name": {
        "type": "text"
      }
    }
  }
}
```
`mapping`된 field만 데이터를 넣을 수 있어 오류가 발생한다.

## Dynamic templates
### Map numbers as `integer` 
```sh
PUT /dynamic_template_test
{
  "mappings": {
    "dynamic_templates": [
        {
        "integers": {
          "match_mapping_type": "long",
          "mapping": {
            "type": "integer"
          }
        }
      }
    ]
  }
}
```
예시로 확인해본다.
```sh
POST /dynamic_template_test/_doc
{
  "in_stock": 123
}
```
`GET /dynamic_template_test/_mapping` 를 실행하면 아래와 같은 결과가 출력된다.
```sh
{
  "dynamic_template_test" : {
    "mappings" : {
      "dynamic_templates" : [
        {
          "integers" : {
            "match_mapping_type" : "long",
            "mapping" : {
              "type" : "integer"
            }
          }
        }
      ],
      "properties" : {
        "in_stock" : {
          "type" : "integer"
        }
      }
    }
  }
}
```
`in_stock` type이 `integer`로 설정되었음을 알 수 있다.

### Modify default mapping for strings
```sh
PUT /test_index
{
  "mappings": {
    "dynamic_templates": [
      {
        "strings": {
          "match_mapping_type": "string",
          "mapping": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 512
              }
            }
          }
        }
      }
    ]
  }
}
```
### Using `match` , `unmatch`
```sh
PUT /test_index
{
  "mappings": {
    "dynamic_templates": [
      {
        "strings_only_text": {
          "match_mapping_type": "string",
          "match": "text_*",
          "unmatch": "*_keyword",
          "mapping": {
            "type": "text"
          }
        }
      },
      {
        "strings_only_keyword": {
          "match_mapping_type": "string",
          "match": "*_keyword",
          "mapping": {
            "type": "keyword"
          }
        }
      }
    ]
  }
}
```
```sh
POST /test_index/_doc
{
  "text_product_description": "A description.",
  "text_product_id_keyword": "ABC-123"
}
```
mapping 정보를 확인하면
```sh
{
  "test_index" : {
    "mappings" : {
      "dynamic_templates" : [
        {
          "strings_only_text" : {
            "match" : "text_*",
            "unmatch" : "*_keyword",
            "match_mapping_type" : "string",
            "mapping" : {
              "type" : "text"
            }
          }
        },
        {
          "strings_only_keyword" : {
            "match" : "*_keyword",
            "match_mapping_type" : "string",
            "mapping" : {
              "type" : "keyword"
            }
          }
        }
      ],
      "properties" : {
        "text_product_description" : {
          "type" : "text"
        },
        "text_product_id_keyword" : {
          "type" : "keyword"
        }
      }
    }
  }
}
```

### Setting `match_pattern` -> `regrex`
```sh
PUT /test_index
{
  "mappings": {
    "dynamic_templates": [
        {
        "names": {
          "match_mapping_type": "string",
          "match": "^[a-zA-Z]+_name$",
          "match_pattern": "regex",
          "mapping": {
            "type": "text"
          }
        }
      }
    ]
  }
}
```
### Using `path_match`
```sh
PUT /test_index
{
  "mappings": {
    "dynamic_templates": [
      {
        "copy_to_full_name": {
          "match_mapping_type": "string",
          "path_match": "employer.name.*",
          "mapping": {
            "type": "text",
            "copy_to": "full_name"
          }
        }
      }
    ]
  }
}
```

### Using placeholders
```sh
PUT /test_index
{
  "mappings": {
    "dynamic_templates": [
      {
        "no_doc_values": {
          "match_mapping_type": "*",
          "mapping": {
            "type": "{dynamic_type}",
            "index": false
          }
        }
      }
    ]
  }
}
```

## Built-in Analyzers
documentation 참고하면 많은 analyzer, filters가 내장되어있음을 알 수 있다.

## Creating custom analyzers
```sh
POST /_analyze
{
  "analyzer": "standard",
  "text": "I&apos;m in a <em>good</em> mood&nbsp;-&nbsp;and I <strong>love</strong> açaí!"
}
```
원하는 방향이 아니지만 html 태그까지 text로 저장된다.

### Remove HTML tags , convert HTML entries
```sh
POST /_analyze
{
  "char_filter": ["html_strip"],
  "text": "I&apos;m in a <em>good</em> mood&nbsp;-&nbsp;and I <strong>love</strong> açaí!"
}
```
```sh
{
  "tokens" : [
    {
      "token" : "I'm in a good mood - and I love açaí!",
      "start_offset" : 0,
      "end_offset" : 78,
      "type" : "word",
      "position" : 0
    }
  ]
}

```
특수문자나 html 태그가 제거되어 text로 저장된다.

### Create a custom analyzer( `my_custom_analyzer`)
```sh
PUT /analyzer_test
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "char_filter": ["html_strip"],
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "stop",
            "asciifolding"
          ]
        }
      }
    }
  }
}
```
예시를 통해 `custom_analyzer`를 적용해본다.
```sh
POST /analyzer_test/_analyze
{
  "analyzer": "my_custom_analyzer", 
  "text": "I&apos;m in a <em>good</em> mood&nbsp;-&nbsp;and I <strong>love</strong> açaí!"
}
```
```sh
{
  "tokens" : [
    {
      "token" : "i'm",
      "start_offset" : 0,
      "end_offset" : 8,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "good",
      "start_offset" : 18,
      "end_offset" : 27,
      "type" : "<ALPHANUM>",
      "position" : 3
    },
    {
      "token" : "mood",
      "start_offset" : 28,
      "end_offset" : 32,
      "type" : "<ALPHANUM>",
      "position" : 4
    },
    {
      "token" : "i",
      "start_offset" : 49,
      "end_offset" : 50,
      "type" : "<ALPHANUM>",
      "position" : 6
    },
    {
      "token" : "love",
      "start_offset" : 59,
      "end_offset" : 72,
      "type" : "<ALPHANUM>",
      "position" : 7
    },
    {
      "token" : "acai",
      "start_offset" : 73,
      "end_offset" : 77,
      "type" : "<ALPHANUM>",
      "position" : 8
    }
  ]
}
```
소문자로 변형되었고, stopwords가 제거되었으면 `acai`로 ascii code가 수정되었음을 알 수 있다. 

### Configure analyzer to danish stop words
```sh
PUT /analyzer_test
{
  "settings": {
    "analysis": {
      "filter": {
        "danish_stop": {
          "type": "stop",
          "stopwords": "_danish_"
        }
      }, 
      # "char_filter" : {}, 추가 가능
      # "tokenizer": {}, 추가 가능
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "char_filter": ["html_strip"],
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "stop",
            "asciifolding"
          ]
        }
      }
    }
  }
}
```
영어로 된 stopwords 대신 danish stopwords로 override된다.

## Adding analyzers to existing indices

### Add new analyzer
```sh
PUT /analyzer_test/_settings
{
  "analysis": {
    "analyzer": {
      "my_second_analyzer": {
        "type": "custom",
        "tokenizer": "standard",
        "char_filter": ["html_strip"],
        "filter": [
          "lowercase",
          "stop",
          "asciifolding"
          ]
      }
    }
  }
}
```
실행하면 아래와 같은 오류가 발생한다.

```sh
  "error" : {
    "root_cause" : [
      {
        "type" : "illegal_argument_exception",
        "reason" : "Can't update non dynamic settings [[index.analysis.analyzer.my_second_analyzer.tokenizer, index.analysis.analyzer.my_second_analyzer.type, index.analysis.analyzer.my_second_analyzer.char_filter, index.analysis.analyzer.my_second_analyzer.filter]] for open indices [[analyzer_test/AbGbwdinRvSbEK4aHmyMGw]]"
      }
```
### Close `analyzer_test` index
```sh
POST /analyzer_test/_close
```
analyzer_test를 close하고 `my_second_analyzer`를 추가하면 오류없이 실행된다. 

### Open `analyzer_test` index
```sh
POST /analyzer_test/_open
```
`analyze`를 실행하려면 다시 open을 실행한다.

`GET /analyzer_test/_settings` 를 실행하면 아래와 같이 2개의 custom analyzer가 포함되었음을 알 수 있다.
```sh
        "analysis" : {
          "analyzer" : {
            "my_second_analyzer" : {
              "filter" : [
                "lowercase",
                "stop",
                "asciifolding"
              ],
              "char_filter" : [
                "html_strip"
              ],
              "type" : "custom",
              "tokenizer" : "standard"
            },
            "my_custom_analyzer" : {
              "filter" : [
                "lowercase",
                "stop",
                "asciifolding"
              ],
              "char_filter" : [
                "html_strip"
              ],
              "type" : "custom",
              "tokenizer" : "standard"
            }
          }
        },
...
```

## Updating analyzers
### Add `description` mapping using `my_custom_analyzer`
```sh
PUT /analyzer_test/_mapping
{
  "properties": {
    "description": {
      "type": "text",
      "analyzer": "my_custom_analyzer"
    }
  }
}
```
예시로 통해 확인한다.
```sh
POST /analyzer_test/_doc
{
  "description": "Is that Peter's cute-looking dog?"
}
```

### Search a query using `keyword` analyzer
```sh
GET /analyzer_test/_search
{
  "query": {
    "match": {
      "description": {
        "query": "that",
        "analyzer": "keyword"
      }
    }
  }
}
```
검색 결과에서 아무것도 나오지 않는데 stopwords를 제거하였기에 `that`이 데이터로 포함되지 않는다.

### Update  `my_custom_analyzer` 
```sh
POST /analyzer_test/_close
```
update하기 위해서는 close를 먼저 실행한다.

```sh
PUT analyzter_test/_settings
{
  "analysis": {
    "analyzer": {
      "my_custom_analyzer": {
        "type": "custom",
        "tokenizer": "standard",
        "char_filter": ["html_strip"],
        "filter": [
          "lowercase",
          "asciifolding"
        ]
      }
    }
  }
}
```
```sh
POST /analyzer_test/_open
```
이 후 똑같은 검색을 실행하면 `that`에 대한 result가 반영된다.

`_update_by_query`로도 수정 가능하다.
```sh
POST /analyzer_test/_update_by_query?conflicts=proceed
```
