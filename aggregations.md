# Aggregations
```sh
GET /order/_mapping
```
```sh
{
  "order" : {
    "mappings" : {
      "properties" : {
        "lines" : {
          "type" : "nested",
          "properties" : {
            "amount" : {
              "type" : "double"
            },
            "product_id" : {
              "type" : "integer"
            },
            "quantity" : {
              "type" : "short"
            }
          }
        },
        "purchased_at" : {
          "type" : "date"
        },
        "sales_channel" : {
          "type" : "keyword"
        },
        "salesman" : {
          "properties" : {
            "id" : {
              "type" : "integer"
            },
            "name" : {
              "type" : "text"
            }
          }
        },
        "status" : {
          "type" : "keyword"
        },
        "total_amount" : {
          "type" : "double"
        }
      }
    }
  }
}
```
## Metrix aggregations
- single-value numeric metrix aggregations 
- multi-value numeric metrix aggregations 

### Calculating statistics with `sum`, `avg`, `min`, `max` aggregations
```sh
GET /order/_search
{
  "size": 0,
  "aggs": {
    "total_sales": {
      "sum": {
        "field": "total_amount"
      }
    },
    "avg_sale": {
      "avg": {
        "field": "total_amount"
      }
    },
    "min_sale": {
      "min": {
        "field": "total_amount"
      }
    },
    "max_sale": {
      "max": {
        "field": "total_amount"
      }
    }
  }
}
```
```sh
  "hits" : {
    "total" : {
      "value" : 1000,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "max_sale" : {
      "value" : 281.77
    },
    "avg_sale" : {
      "value" : 109.20961
    },
    "min_sale" : {
      "value" : 10.27
    },
    "total_sales" : {
      "value" : 109209.61
    }
  }
```

### Retrieving the number of distinct values
`cardinality`로 unique한 대략적인 값을 추릴 수 있다. 정확한 값을 계산하려면 다른 방법을 사용해야 할 것.
```sh
GET /order/_search
{
  "size": 0,
  "aggs": {
    "total_salesmen": {
      "cardinality": {
        "field": "salesman.id"
      }
    }
  }
}
```
```sh
  "aggregations" : {
    "total_salesmen" : {
      "value" : 100
    }
  }
```

### Retrieving the number of values
```sh
GET /order/_search
{
  "size": 0,
  "aggs": {
    "values_count": {
      "value_count": {
        "field": "total_amount"
      }
    }
  }
}
```
```sh
  "aggregations" : {
    "values_count" : {
      "value" : 1000
    }
```

### Using `stats` aggregation for common statistics
```sh
GET /order/_search
{
  "size": 0,
  "aggs": {
    "amount_stats": {
      "stats": {
        "field": "total_amount"
      }
    }
  }
}
```
```sh
  "aggregations" : {
    "amount_stats" : {
      "count" : 1000,
      "min" : 10.27,
      "max" : 281.77,
      "avg" : 109.20961,
      "sum" : 109209.61
    }
  }
```

## Bucket aggregations
### Creating a bucket for each `status` value
```sh
GET /order/_search
{
  "size": 0,
  "aggs": {
    "status_term": {
      "terms": {
        "field": "status"
      }
    }
  }
}
```

### Including `20` terms instead of the default `10`
```sh
GET /order/_search
{
  "size": 0, 
  "aggs": {
    "status_terms": {
      "terms": {
        "field": "status",
        "size": 20
      }
    }
  }
}
```
### Aggregating documents with missing field
```sh
GET /order/_search
{
  "size": 0,
  "aggs": {
    "status_term": {
      "terms": {
        "field": "status",
        "size": 20,
        "missing": "N/A"
      }
    }
  }
}
```
### Changing the mininum document count for a bucket to be created
```sh
GET /order/_search
{
  "size": 0,
  "aggs": {
    "status_term": {
      "terms": {
          "field": "status",
          "size": 20,
          "missing": "N/A",
          "min_doc_count": 0
      }
    }
  }
}
```
### Ordering the buckets
```sh
GET /order/_search
{
  "size": 0,
  "aggs": {
    "status_term": {
      "terms": {
        "field": "status",
        "size": 20,
        "missing": "N/A",
        "min_doc_count": 0,
        "order": {
          "_key": "asc"
        }
      }
    }
  }
}
```