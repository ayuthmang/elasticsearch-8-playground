## 20. Import a Single Movie via JSON / REST
curl -XPUT 127.0.0.1:9200/movies -H "Content-Type: application/json" -d '
{
  "mappings": {
    "properties": {
      "year": {
        "type": "date"
      }
    }
  }
}
'

curl -XGET 127.0.0.1:9200/movies/_mapping

curl -XPUT 127.0.0.1:9200/movies/_doc/109487\?pretty -H "Content-Type: application/json" -d '
{
  "genre": ["IMAX", "Sci-Fi"],
  "title": "Interstellar",
  "year": 2014
}
'

curl -XGET 127.0.0.1:9200/movies/_search\?pretty


## 21. Insert Many Movies at Once with the Bulk API

wget http://media.sundog-soft.com/es8/movies.json

curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/_bulk\?pretty --data-binary @movies.json

## 22. Updating Data in Elasticsearch 6min

curl -XPUT 127.0.0.1:9200/movies/_doc/109487\?pretty -H "Content-Type: application/json" -d '
{
  "genre": ["IMAX", "Sci-Fi"],
  "title": "Interstellar foo",
  "year": 2014
}
'

curl -XGET 127.0.0.1:9200/movies/_search\?pretty

curl -XPOST 127.0.0.1:9200/movies/_update/109487\?pretty -H "Content-Type: application/json" -d '
{
  "doc": {
    "title": "Interstellar"
  }
}
'

## 23. Deleting Data in Elasticsearch

curl -XGET 127.0.0.1:9200/movies/_search\?pretty\&q=Dark
curl -XDELETE 127.0.0.1:9200/movies/_doc/58559\?pretty # id from above
curl -XGET 127.0.0.1:9200/movies/_search\?pretty\&q=Dark

## [Exercise] Insert, Update and Delete a Movie

```bash
# create
curl -XPUT 127.0.0.1:9200/movies/_doc/200000\?pretty -H "Content-Type: application/json" -d '
{
  "title": "Babys Adventures in Elasticsearch",
  "genre": ["Documentary"],
  "year": 2024
}
'

# get
curl -XGET 127.0.0.1:9200/movies/_doc/200000\?pretty

# update
curl -XPOST 127.0.0.1:9200/movies/_update/200000\?pretty -H "Content-Type: application/json" -d '
{
  "doc": {
    "genre": ["Documentary", "Comedy"]
  }
}
'

# delete
curl -XDELETE 127.0.0.1:9200/movies/_doc/200000\?pretty
```

## 25. Dealing with Concurrency

```bash
curl -XGET 127.0.0.1:9200/movies/_doc/109487\?pretty

curl -XPUT 127.0.0.1:9200/movies/_doc/109487\?pretty\&if_seq_no=17\&if_primary_term=1 -H "Content-Type: application/json" -d '
{
  "genre": ["IMAX", "Sci-Fi"],
  "title": "Interstellar foo",
  "year": 2014
}'

# let es does its jobs
curl -XPOST 127.0.0.1:9200/movies/_update/109487\?pretty\&retry_on_conflict=5 -H "Content-Type: application/json" -d '
{
  "doc": {
    "title": "Interstellar typo"
  }
}'

curl -XGET 127.0.0.1:9200/movies/_doc/109487\?pretty
```
