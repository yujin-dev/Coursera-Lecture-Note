## Using Analyze API
### `standard` analyzer로 string 분석
```sh
POST /_analyze
{
  "text": "2 guys walk into   a bar, but the third... DUCKS! :-)",
  "analyzer": "standard"
}
```

### 위와 동일한 로직으로 적용됨
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

# retrieve document
GET /coercion_test/_doc/2

DELETE /coercion_test

# Understanding arrays
array라는 개념은 없다.
## Arrays of strings are concatenated 
POST /_analyze
{
  "text": ["Strings are simply", "merged together."],
  "analyzer": "standard"
}
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

# Adding explicit mappings
## add field mappings for `reviews` index
`reviews` index에 mapping 추가
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

## index test document
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
email data type이 안 맞기에 error을 발생함
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
### Retrieve index mapping field 
```sh
GET /reviews/_mapping/field/content
```

# Using dot notation in field names
nested json 형식에 보기 쉽게 dot properties를 이용한다.
## Using dot notation for `author`
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
## Retrieve mapping
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
# Adding mappings to existing indices
## Add new field mapping to existing index
`mapping` API를 이용하여 기존의 index에 mapping을 추가한다.

PUT /reviews/_mapping
{
  "properties":  {
    "created_at": {
      "type": "date"
    }
  }
}

GET /reviews/_mapping

## How dates work
## only a date
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

## both a date and time
UTC time을 반영함
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

## UTC offset
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

## timestamp
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

GET /reviews/_search
{
  "query": {
    "match_all": {}
  }
}

# Updating existing mappings
## 보통 field mappings는 업데이트될 수 없음
PUT /reviews/_mapping
{
  "properties": {
    "product_id": {
      "type": "keyword"
    }
  }
}

## `ignore_above`는 업데이트될 수 있음
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

# Reindexing documents with the Reindex API
## Add new index with new mapping
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

## Retrieve mapping
GET /reviews/_mappings

## Reindex documents into `reviews_new`
POST /_reindex
{
  "source": {
    "index": "reviews"
  },
  "dest": {
    "index": "reviews_new"
  }
}

## Delete all documents
POST /reviews_new/_delete_by_query
{
  "query": {
    "match_all": {}
  }
}

## convert `product_id` values to strings
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

## Retrieve documents
GET /reviews_new/_search
{
  "query": {
    "match_all": {}
  }
}

## Reindex specific documents
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

## Reindex only positive reviews
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

## Removing fields
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

## Changing field's name : "content" -> "comment"
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


## Ignore review with ratings below 4.0
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

# Define field aliases
## Add `comment` alias pointing to the `content` field
PUT /reviews/_mapping

{
  "properties": {
    "comment": {
      "type": "alias",
      "path": "content"
    }
  }
}
## Using the field alias
GET /reviews/_search
{
  "query": {
    "match": {
      "comment": "outstanding"
    }
  }
}

## Multi-field mappings
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


## Index a test document
POST /multi_field_test/_doc
{
  "description": "To make this spaghetti carbonara, you first need to...",
  "ingredients": ["Spaghetti", "Bacon", "Eggs"]
}


## Retrieve documents
GET /multi_field_test/_search
{
  "query": {
    "match_all": {}
  }
}


## Querying the `text` mapping
GET /multi_field_test/_search
{
  "query": {
    "match": {
      "ingredients": "Spaghetti"
    }
  }
}


## Querying the `kerword` mapping
GET /multi_field_test/_search
{
  "query": {
    "term": {
      "ingredients.keyword": "Spaghetti"
    }
  }
}

# Index templates
## Adding an index template names `access-logs`
PUT /_template/access-logs
{
  "index_patterns": ["access-logs-*"],
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

## Adding an index matching the index template's pattern
PUT /access-logs-2020-01-01

## Verify : the mapping is applied
GET /access-logs-2020-01-01


# Combining explicit and dynamic mapping
## Create index with 1 field mapping
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

## Index a test document with an unmapped field
POST /people/_doc
{
  "first_name": "Bo",
  "last_name": "Andersen"
}

## Retrieve mapping
GET /people/_mapping

# Configuring dynamic mapping
## Disable dynamic mapping
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

## Set dynamic mapping to `strict`
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


## Index a test document
POST /people/_doc
{
  "first_name": "Bo",
  "last_name": "Andersen"
}

## Search `first_name` field
GET /people/_search
{
  "query": {
    "match": {
      "first_name": "Bo"
    }
  }
}

## serach `last_name` field
GET /people/_search
{
  "query": {
    "match": {
      "last_name": "Andersen"
    }
  }
}

# Dynamic templates
## Map numbers as `integer` 
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

## Test the dynamic template
POST /dynamic_template_test/_doc
{
  "in_stock": 123
}

## Modify default mapping for strings
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


## Using `match` , `unmatch`
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

POST /test_index/_doc
{
  "text_product_description": "A description.",
  "text_product_id_keyword": "ABC-123"
}


## Setting `match_pattern` -> `regrex`
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

POST /test_index/_doc
{
  "first_name": "John",
  "middle_name": "Edward",
  "last_name": "Doe"
}

## Using `path_match`
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

POST /test_index/_doc
{
  "employer": {
    "name": {
      "first_name": "John",
      "middle_name": "Edward",
      "last_name": "Doe"
    }
  }
}

## Using placeholders
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

POST /test_index/_doc
{
  
  "name": "John Doe",
  "age": 26
}

# Creating custom analyzers
## Remove HTML tags , convert HTML entries
POST /_analyze
{
  "char_filter": ["html_strip"],
  "text": "I&apos;m in a <em>good</em> mood&nbsp;-&nbsp;and I <strong>love</strong> açaí!"
}


## Add `standard` tokenizer
POST /_analyze
{
  "char_filter": ["html_strip"],
  "tokenizer": "standard",
  "text": "I&apos;m in a <em>good</em> mood&nbsp;-&nbsp;and I <strong>love</strong> açaí!"
 
}

## Add `lowercase` token filter
POST /_analyze
{
  "char_filter": ["html_strip"],
  "tokenizer": "standard",
  "filter": [
    "lowercase"],
    "text": "I&apos;m in a <em>good</em> mood&nbsp;-&nbsp;and I <strong>love</strong> açaí!"

}


## Add the `stop` token filter
POST /_analyze
{
  "char_filter": ["html_strip"],
  "tokenizer": "standard",
  "filter": [
    "lowercase",
    "stop"
    ],
    "text": "I&apos;m in a <em>good</em> mood&nbsp;-&nbsp;and I <strong>love</strong> açaí!"
}


## Add `asciifolding` token filter
POST /_analyze
{
  "char_filter": ["html_strip"],
  "tokenizer": "standard",
  "filter": [
    "lowercase",
    "stop",
    "asciifolding"
    ],
    "text": "I&apos;m in a <em>good</em> mood&nbsp;-&nbsp;and I <strong>love</strong> açaí!"
}

## Create a custom analyzer( `my_custom_analyzer`)
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

## Configure analyzer to danish stop words


## Test the custom analyzer
POST /analyzer_test/_analyze
{
  "analyzer": "my_custom_analyzer", 
  "text": "I&apos;m in a <em>good</em> mood&nbsp;-&nbsp;and I <strong>love</strong> açaí!"
}

# Adding analyzers to existing indices
## Close `analyzer_test` index
POST /analyzer_test/_close

## Add new analyzer
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

## Open `analyzer_test` index
POST /analyzer_test/_open

## Retrieve index settings
GET /analyzer_test/_settings


# Updating analyzers
## Add `description` mapping using `my_custom_analyzer`
PUT /analyzer_test/_mapping
{
  "properties": {
    "description": {
      "type": "text",
      "analyzer": "my_custom_analyzer"
    }
  }
}


## Index a test document
POST /analyzer_test/_doc
{
  "description": "Is that Peter's cute-looking dog?"
}


## Search a query using `keyword` analyzer
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

## close `analyzer_test` index
POST /analyzer_test/_close
## Update  `my_custom_analyzer` 
PUT /analyzter_test/_settings
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

## Open `anazlyer_test` index
POST /analyzer_test/_open

## Retrieve index settings
GET /analyzer_test/_settings

## Reindex documents
POST /analyzer_test/_update_by_query?conflicts=proceed
