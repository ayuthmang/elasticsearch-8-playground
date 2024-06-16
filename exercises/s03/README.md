# Section 3: Searching with Elasticsearch

## JSON Search In-Depth

```bash
curl -XGET '127.0.0.1:9200/movies/_search?pretty' -H 'Content-Type: application/json' -d '{
  "query": {
    "match": {
      "title": "star"
    }
  }
}'
```

Response

```json
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 0.919734,
    "hits": [
      {
        "_index": "movies",
        "_id": "135569",
        "_score": 0.919734,
        "_source": {
          "id": "135569",
          "title": "Star Trek Beyond",
          "year": 2016,
          "genre": ["Action", "Adventure", "Sci-Fi"]
        }
      },
      {
        "_index": "movies",
        "_id": "122886",
        "_score": 0.666854,
        "_source": {
          "id": "122886",
          "title": "Star Wars: Episode VII - The Force Awakens",
          "year": 2015,
          "genre": ["Action", "Adventure", "Fantasy", "Sci-Fi", "IMAX"]
        }
      }
    ]
  }
}
```

```bash
curl -XGET '127.0.0.1:9200/movies/_search?pretty' -H 'Content-Type: application/json' -d '{
  "query": {
    "bool": {
      "must": {
        "term": {
          "title": "trek"
        }
      },
      "filter": {
        "range": {
          "year": {
            "gte": 2010
          }
        }
      }
    }
  }
}'
```

Response

```json
{
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1.456388,
    "hits": [
      {
        "_index": "movies",
        "_id": "135569",
        "_score": 1.456388,
        "_source": {
          "id": "135569",
          "title": "Star Trek Beyond",
          "year": 2016,
          "genre": ["Action", "Adventure", "Sci-Fi"]
        }
      }
    ]
  }
}
```

## Phrase Matching

```bash
curl -XGET '127.0.0.1:9200/movies/_search?pretty' -H 'Content-Type: application/json' -d '{
  "query": {
    "match": {
      "title": "star wars"
    }
  }
}'
```

response

```json
{
  "took": 6,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 1.7228094,
    "hits": [
      {
        "_index": "movies",
        "_id": "122886",
        "_score": 1.7228094,
        "_source": {
          "id": "122886",
          "title": "Star Wars: Episode VII - The Force Awakens",
          "year": 2015,
          "genre": ["Action", "Adventure", "Fantasy", "Sci-Fi", "IMAX"]
        }
      },
      {
        "_index": "movies",
        "_id": "135569",
        "_score": 0.919734,
        "_source": {
          "id": "135569",
          "title": "Star Trek Beyond",
          "year": 2016,
          "genre": ["Action", "Adventure", "Sci-Fi"]
        }
      }
    ]
  }
}
```

`match_phrase`

```bash
curl -XGET '127.0.0.1:9200/movies/_search?pretty' -H 'Content-Type: application/json' -d '{
  "query": {
    "match_phrase": {
      "title": "star wars"
    }
  }
}'

# response
{
  "took" : 13,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.7228093,
    "hits" : [
      {
        "_index" : "movies",
        "_id" : "122886",
        "_score" : 1.7228093,
        "_source" : {
          "id" : "122886",
          "title" : "Star Wars: Episode VII - The Force Awakens",
          "year" : 2015,
          "genre" : [
            "Action",
            "Adventure",
            "Fantasy",
            "Sci-Fi",
            "IMAX"
          ]
        }
      }
    ]
  }
}
```

playing with `slop` value 1 or 100

```bash
curl -XGET '127.0.0.1:9200/movies/_search?pretty' -H 'Content-Type: application/json' -d '{
  "query": {
    "match_phrase": {
      "title": {
        "query": "star beyond",
        "slop": 1 #
      }
    }
  }
}'

# response
{
  "took" : 8,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.5607002,
    "hits" : [
      {
        "_index" : "movies",
        "_id" : "135569",
        "_score" : 1.5607002,
        "_source" : {
          "id" : "135569",
          "title" : "Star Trek Beyond",
          "year" : 2016,
          "genre" : [
            "Action",
            "Adventure",
            "Sci-Fi"
          ]
        }
      }
    ]
  }
}
```

## [Exercise] Querying in Different Ways

Search for "Star Wars" movies related after 1980 using both `URI search` and a `request body search`.

solution

```bash
# with uri (actual string is :q=+year:>1980 +title:"star wars")
curl -XGET '127.0.0.1:9200/movies/_search?q=%2Byear%3A%3E1980%20%2Btitle%3A%22star%20wars%22&pretty' -H 'Content-Type: application/json'

# response
{
  "took" : 15,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 2.7228093,
    "hits" : [
      {
        "_index" : "movies",
        "_id" : "122886",
        "_score" : 2.7228093,
        "_source" : {
          "id" : "122886",
          "title" : "Star Wars: Episode VII - The Force Awakens",
          "year" : 2015,
          "genre" : [
            "Action",
            "Adventure",
            "Fantasy",
            "Sci-Fi",
            "IMAX"
          ]
        }
      }
    ]
  }
}
```

with request body

```bash
curl -XGET '127.0.0.1:9200/movies/_search?pretty' -H 'Content-Type: application/json' -d '{
  "query": {
    "bool": {
      "must": {
        "match": {
          "title": "star wars"
        }
      },
      "filter": {
        "range": {
          "year": {
            "gte": 1980
          }
        }
      }
    }
  }
}'

# response
{
  "took" : 8,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 1.7228094,
    "hits" : [
      {
        "_index" : "movies",
        "_id" : "122886",
        "_score" : 1.7228094,
        "_source" : {
          "id" : "122886",
          "title" : "Star Wars: Episode VII - The Force Awakens",
          "year" : 2015,
          "genre" : [
            "Action",
            "Adventure",
            "Fantasy",
            "Sci-Fi",
            "IMAX"
          ]
        }
      },
      {
        "_index" : "movies",
        "_id" : "135569",
        "_score" : 0.919734,
        "_source" : {
          "id" : "135569",
          "title" : "Star Trek Beyond",
          "year" : 2016,
          "genre" : [
            "Action",
            "Adventure",
            "Sci-Fi"
          ]
        }
      }
    ]
  }
}
```

## Pagination

pagination syntax

```bash
curl -XGET '127.0.0.1:9200/movies/_search?size=2&pretty'
curl -XGET '127.0.0.1:9200/movies/_search?size=2&from=2&pretty'
curl -XGET '127.0.0.1:9200/movies/_search?pretty' -H 'Content-Type: application/json' -d '{
  "from": 2,
  "size": 2,
  "query": {"match": {"genre": "Sci-Fi"}}
}'
```

## Sorting

```bash
curl -XGET '127.0.0.1:9200/movies/_search?sort=year&pretty'

# result
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 5,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [
      {
        "_index" : "movies",
        "_id" : "1924",
        "_score" : null,
        "_source" : {
          "id" : "1924",
          "title" : "Plan 9 from Outer Space",
          "year" : 1959,
          "genre" : [
            "Horror",
            "Sci-Fi"
          ]
        },
        "sort" : [
          -347155200000
        ]
      },
      {
        "_index" : "movies",
        "_id" : "58559",
        "_score" : null,
        "_source" : {
          "id" : "58559",
          "title" : "Dark Knight, The",
          "year" : 2008,
          "genre" : [
            "Action",
            "Crime",
            "Drama",
            "IMAX"
          ]
        },
        "sort" : [
          1199145600000
        ]
      },
      {
        "_index" : "movies",
        "_id" : "109487",
        "_score" : null,
        "_source" : {
          "id" : "109487",
          "title" : "Interstellar",
          "year" : 2014,
          "genre" : [
            "Sci-Fi",
            "IMAX"
          ]
        },
        "sort" : [
          1388534400000
        ]
      },
      {
        "_index" : "movies",
        "_id" : "122886",
        "_score" : null,
        "_source" : {
          "id" : "122886",
          "title" : "Star Wars: Episode VII - The Force Awakens",
          "year" : 2015,
          "genre" : [
            "Action",
            "Adventure",
            "Fantasy",
            "Sci-Fi",
            "IMAX"
          ]
        },
        "sort" : [
          1420070400000
        ]
      },
      {
        "_index" : "movies",
        "_id" : "135569",
        "_score" : null,
        "_source" : {
          "id" : "135569",
          "title" : "Star Trek Beyond",
          "year" : 2016,
          "genre" : [
            "Action",
            "Adventure",
            "Sci-Fi"
          ]
        },
        "sort" : [
          1451606400000
        ]
      }
    ]
  }
}

# sort title
curl -XGET '127.0.0.1:9200/movies/_search?sort=title&pretty'

# result
{
  "error" : {
    "root_cause" : [
      {
        "type" : "illegal_argument_exception",
        "reason" : "Fielddata is disabled on [title] in [movies]. Text fields are not optimised for operations that require per-document field data like aggregations and sorting, so these operations are disabled by default. Please use a keyword field instead. Alternatively, set fielddata=true on [title] in order to load field data by uninverting the inverted index. Note that this can use significant memory."
      }
    ],
    "type" : "search_phase_execution_exception",
    "reason" : "all shards failed",
    "phase" : "query",
    "grouped" : true,
    "failed_shards" : [
      {
        "shard" : 0,
        "index" : "movies",
        "node" : "0pF_eyTgTNiJoma4qrUE8w",
        "reason" : {
          "type" : "illegal_argument_exception",
          "reason" : "Fielddata is disabled on [title] in [movies]. Text fields are not optimised for operations that require per-document field data like aggregations and sorting, so these operations are disabled by default. Please use a keyword field instead. Alternatively, set fielddata=true on [title] in order to load field data by uninverting the inverted index. Note that this can use significant memory."
        }
      }
    ],
    "caused_by" : {
      "type" : "illegal_argument_exception",
      "reason" : "Fielddata is disabled on [title] in [movies]. Text fields are not optimised for operations that require per-document field data like aggregations and sorting, so these operations are disabled by default. Please use a keyword field instead. Alternatively, set fielddata=true on [title] in order to load field data by uninverting the inverted index. Note that this can use significant memory.",
      "caused_by" : {
        "type" : "illegal_argument_exception",
        "reason" : "Fielddata is disabled on [title] in [movies]. Text fields are not optimised for operations that require per-document field data like aggregations and sorting, so these operations are disabled by default. Please use a keyword field instead. Alternatively, set fielddata=true on [title] in order to load field data by uninverting the inverted index. Note that this can use significant memory."
      }
    }
  },
  "status" : 400
}

# delete, create new index and import data
curl -X DELETE '127.0.0.1:9200/movies'
curl -X PUT '127.0.0.1:9200/movies' -H 'Content-Type: application/json' -d '{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "fields": {
          "raw": {
            "type": "keyword"
          }
        }
      }
    }
  }
}'
curl -XPUT '127.0.0.1:9200/_bulk?pretty' -H 'Content-Type: application/json' --data-binary @movies.json

curl -XGET '127.0.0.1:9200/movies/_search?sort=title&pretty'

# result
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 5,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [
      {
        "_index" : "movies",
        "_id" : "58559",
        "_score" : null,
        "_source" : {
          "id" : "58559",
          "title" : "Dark Knight, The",
          "year" : 2008,
          "genre" : [
            "Action",
            "Crime",
            "Drama",
            "IMAX"
          ]
        },
        "sort" : [
          "Dark Knight, The"
        ]
      },
      {
        "_index" : "movies",
        "_id" : "109487",
        "_score" : null,
        "_source" : {
          "id" : "109487",
          "title" : "Interstellar",
          "year" : 2014,
          "genre" : [
            "Sci-Fi",
            "IMAX"
          ]
        },
        "sort" : [
          "Interstellar"
        ]
      },
      {
        "_index" : "movies",
        "_id" : "1924",
        "_score" : null,
        "_source" : {
          "id" : "1924",
          "title" : "Plan 9 from Outer Space",
          "year" : 1959,
          "genre" : [
            "Horror",
            "Sci-Fi"
          ]
        },
        "sort" : [
          "Plan 9 from Outer Space"
        ]
      },
      {
        "_index" : "movies",
        "_id" : "135569",
        "_score" : null,
        "_source" : {
          "id" : "135569",
          "title" : "Star Trek Beyond",
          "year" : 2016,
          "genre" : [
            "Action",
            "Adventure",
            "Sci-Fi"
          ]
        },
        "sort" : [
          "Star Trek Beyond"
        ]
      },
      {
        "_index" : "movies",
        "_id" : "122886",
        "_score" : null,
        "_source" : {
          "id" : "122886",
          "title" : "Star Wars: Episode VII - The Force Awakens",
          "year" : 2015,
          "genre" : [
            "Action",
            "Adventure",
            "Fantasy",
            "Sci-Fi",
            "IMAX"
          ]
        },
        "sort" : [
          "Star Wars: Episode VII - The Force Awakens"
        ]
      }
    ]
  }
}
```

## More with Filters

```bash
curl -XGET '127.0.0.1:9200/movies/_search?pretty' -H 'Content-Type: application/json' -d '{
  "query": {
    "bool": {
      "must": { "match": { "genre": "Sci-Fi" } },
      "must_not": { "match": { "title": "trek" } },
      "filter": { "range": { "year": { "gte": 2010, "lt": 2015 } } }
    }
  }
}'

# result
{
  "took" : 5,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.640912,
    "hits" : [
      {
        "_index" : "movies",
        "_id" : "109487",
        "_score" : 0.640912,
        "_source" : {
          "id" : "109487",
          "title" : "Interstellar",
          "year" : 2014,
          "genre" : [
            "Sci-Fi",
            "IMAX"
          ]
        }
      }
    ]
  }
}
```

## Using Filters

```bash
curl -XGET '127.0.0.1:9200/movies/_search?sort=title.raw&pretty' -H 'Content-Type: application/json' -d '{
  "query": {
    "bool": {
      "must": { "match": { "genre": "Sci-Fi" } },
      "filter": { "range": { "year": { "lt": 1960 } } }
    }
  }
}'

# response
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [
      {
        "_index" : "movies",
        "_id" : "1924",
        "_score" : null,
        "_source" : {
          "id" : "1924",
          "title" : "Plan 9 from Outer Space",
          "year" : 1959,
          "genre" : [
            "Horror",
            "Sci-Fi"
          ]
        },
        "sort" : [
          "Plan 9 from Outer Space"
        ]
      }
    ]
  }
}
```

## Fuzzy Queries

```bash
curl -XGET '127.0.0.1:9200/movies/_search?pretty' -H 'Content-Type: application/json' -d '{
  "query": {
    "match": {
      "title": "interstellar"
    }
  }
}'

# response
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.9844898,
    "hits" : [
      {
        "_index" : "movies",
        "_id" : "109487",
        "_score" : 1.9844898,
        "_source" : {
          "id" : "109487",
          "title" : "Interstellar",
          "year" : 2014,
          "genre" : [
            "Sci-Fi",
            "IMAX"
          ]
        }
      }
    ]
  }
}

# fuzzy
curl -XGET '127.0.0.1:9200/movies/_search?pretty' -H 'Content-Type: application/json' -d '{
  "query": {
    "fuzzy": {
      "title": {
        "value": "intersteller",
        "fuzziness": 1
      }
    }
  }
}'

# response
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.8191156,
    "hits" : [
      {
        "_index" : "movies",
        "_id" : "109487",
        "_score" : 1.8191156,
        "_source" : {
          "id" : "109487",
          "title" : "Interstellar",
          "year" : 2014,
          "genre" : [
            "Sci-Fi",
            "IMAX"
          ]
        }
      }
    ]
  }
}

curl -XGET '127.0.0.1:9200/movies/_search?pretty' -H 'Content-Type: application/json' -d '{
  "query": {
    "fuzzy": {
      "title": {
        "value": "interustell",
        "fuzziness": 2
      }
    }
  }
}'

curl -XGET '127.0.0.1:9200/movies/_search?pretty' -H 'Content-Type: application/json' -d '{
  "query": {
    "fuzzy": {
      "title": {
        "value": "warz",
        "fuzziness": 1
      }
    }
  }
}'

# response
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.77331555,
    "hits" : [
      {
        "_index" : "movies",
        "_id" : "122886",
        "_score" : 0.77331555,
        "_source" : {
          "id" : "122886",
          "title" : "Star Wars: Episode VII - The Force Awakens",
          "year" : 2015,
          "genre" : [
            "Action",
            "Adventure",
            "Fantasy",
            "Sci-Fi",
            "IMAX"
          ]
        }
      }
    ]
  }
}
```

## Partial Matching

```bash
curl -X DELETE '127.0.0.1:9200/movies'
curl -X PUT '127.0.0.1:9200/movies' -H 'Content-Type: application/json' -d '{
  "mappings": {
    "properties": {
      "year": { "type": "text" }
    }
  }
}'
curl -XPUT '127.0.0.1:9200/_bulk?pretty' -H 'Content-Type: application/json' --data-binary @movies.json

curl -XGET '127.0.0.1:9200/movies/_search?pretty' -H 'Content-Type: application/json' -d '{
  "query": {
    "prefix": {
      "year": "201"
    }
  }
}'

# response
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "movies",
        "_id" : "135569",
        "_score" : 1.0,
        "_source" : {
          "id" : "135569",
          "title" : "Star Trek Beyond",
          "year" : 2016,
          "genre" : [
            "Action",
            "Adventure",
            "Sci-Fi"
          ]
        }
      },
      {
        "_index" : "movies",
        "_id" : "122886",
        "_score" : 1.0,
        "_source" : {
          "id" : "122886",
          "title" : "Star Wars: Episode VII - The Force Awakens",
          "year" : 2015,
          "genre" : [
            "Action",
            "Adventure",
            "Fantasy",
            "Sci-Fi",
            "IMAX"
          ]
        }
      },
      {
        "_index" : "movies",
        "_id" : "109487",
        "_score" : 1.0,
        "_source" : {
          "id" : "109487",
          "title" : "Interstellar",
          "year" : 2014,
          "genre" : [
            "Sci-Fi",
            "IMAX"
          ]
        }
      }
    ]
  }
}

# wildcard
curl -XGET '127.0.0.1:9200/movies/_search?pretty' -H 'Content-Type: application/json' -d '{
  "query": {
    "wildcard": {
      "year": "1*"
    }
  }
}'

# response
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "movies",
        "_id" : "1924",
        "_score" : 1.0,
        "_source" : {
          "id" : "1924",
          "title" : "Plan 9 from Outer Space",
          "year" : 1959,
          "genre" : [
            "Horror",
            "Sci-Fi"
          ]
        }
      }
    ]
  }
}
```
