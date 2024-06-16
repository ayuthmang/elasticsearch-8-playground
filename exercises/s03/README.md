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

# with request body

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
