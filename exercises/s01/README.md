## Installing Elasticsearch [Step by Step]

```bash
curl -X GET '127.0.0.1:9200/?pretty'
curl -H 'Content-Type: application/json' -X PUT '127.0.0.1:9200/shakespeare?pretty' --data-binary @shakes-mapping.json

curl -H 'Content-Type: application/json' -X PUT '127.0.0.1:9200/shakespeare/_bulk?pretty' --data-binary @shakespeare_8.0.json
curl -H 'Content-Type: application/json' -X GET '127.0.0.1:9200/shakespeare/_search?pretty' -d '
{
  "query" : {
    "match_phrase" : {
        "text_entry" : "to be or not to be"
    }
  }
}'
```
