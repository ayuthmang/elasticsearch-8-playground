## 20. Import a Single Movie via JSON / REST

```bash
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
```


## 21. Insert Many Movies at Once with the Bulk API

```bash
wget http://media.sundog-soft.com/es8/movies.json

curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/_bulk\?pretty --data-binary @movies.json
```

## 22. Updating Data in Elasticsearch 6min

```bash
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
```

## 23. Deleting Data in Elasticsearch

```bash
curl -XGET 127.0.0.1:9200/movies/_search\?pretty\&q=Dark
curl -XDELETE 127.0.0.1:9200/movies/_doc/58559\?pretty # id from above
curl -XGET 127.0.0.1:9200/movies/_search\?pretty\&q=Dark
```

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

## 26. Using Analyzers and Tokenizers

```bash
curl -H "Content-Type: application/json" -XGET 127.0.0.1:9200/movies/_search\?pretty -d '
{
  "query": {
    "match": {
      "title": "Star Trek"
    }
  }
}'

curl -H "Content-Type: application/json" -XGET 127.0.0.1:9200/movies/_search\?pretty -d '
{
  "query": {
    "match_phrase": {
      "genre": "sci"
    }
  }
}'

curl -XDELETE 127.0.0.1:9200/movies

curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/movies -d '
{
  "mappings": {
    "properties": {
      "id": { "type": "integer" },
      "year": { "type": "date" },
      "genre": { "type": "keyword" },
      "title": { "type": "text", "analyzer": "english" }
    }
  }
}'

curl -XGET 127.0.0.1:9200/movies\?pretty
curl -XPUT 127.0.0.1:9200/_bulk\?pretty -H "Content-Type: application/json" --data-binary @movies.json

curl -H "Content-Type: application/json" -XGET 127.0.0.1:9200/movies/_search\?pretty -d '
{
  "query": {
    "match_phrase": {
      "genre": "sci"
    }
  }
}'

curl -H "Content-Type: application/json" -XGET 127.0.0.1:9200/movies/_search\?pretty -d '
{
  "query": {
    "match_phrase": {
      "genre": "sci-fi"
    }
  }
}'

curl -H "Content-Type: application/json" -XGET 127.0.0.1:9200/movies/_search\?pretty -d '
{
  "query": {
    "match_phrase": {
      "genre": "Sci-Fi"
    }
  }
}'

curl -H "Content-Type: application/json" -XGET 127.0.0.1:9200/movies/_search\?pretty -d '
{
  "query": {
    "match": {
      "title": "star wars"
    }
  }
}'
```

## 28. Data Modeling and Parent/Child Relationships, Part 2

```bash
curl -XPUT 127.0.0.1:9200/series -H "Content-Type: application/json" -d '
{
  "mappings": {
    "properties": {
      "film_to_franchise": {
        "type": "join",
        "relations": {
          "franchise": "film"
        }
      }
    }
  }
}
'

curl -O http://media.sundog-soft.com/es8/series.json

curl -XPUT 127.0.0.1:9200/_bulk\?pretty -H "Content-Type: application/json" --data-binary @series.json

curl -XGET 127.0.0.1:9200/series/_search\?pretty -H "Content-Type: application/json" -d '
{
  "query": {
    "has_parent": {
      "parent_type": "franchise",
      "query": {
        "match": {
          "title": "Star Wars"
        }
      }
    }
  }
}
'

curl -XGET 127.0.0.1:9200/series/_search\?pretty -H "Content-Type: application/json" -d '
{
  "query": {
    "has_child": {
      "type": "film",
      "query": {
        "match": {
          "title": "The Force Awakens"
        }
      }
    }
  }
}
'
```

## 29. Flattened Datatype

```bash
curl -O http://media.sundog-soft.com/es/flattened.txt

1.

curl -H "Content-Type: application/json" -XPUT "http://127.0.0.1:9200/demo-default/_doc/1" -d'{
  "message": "[5592:1:0309/123054.737712:ERROR:child_process_sandbox_support_impl_linux.cc(79)] FontService unique font name matching request did not receive a response.",
  "fileset": {
    "name": "syslog"
  },
  "process": {
    "name": "org.gnome.Shell.desktop",
    "pid": 3383
  },
  "@timestamp": "2020-03-09T18:00:54.000+05:30",
  "host": {
    "hostname": "bionic",
    "name": "bionic"
  }
}'


2.

curl -H "Content-Type: application/json" -XGET "http://127.0.0.1:9200/demo-default/_mapping?pretty=true"

3.

curl -H "Content-Type: application/json" -XGET "http://127.0.0.1:9200/_cluster/state?pretty=true" >> es-cluster-state.json

4.

curl -H "Content-Type: application/json" -XPUT "http://127.0.0.1:9200/demo-flattened"

5.

curl -H "Content-Type: application/json" -XPUT "http://127.0.0.1:9200/demo-flattened/_mapping" -d'{
  "properties": {
    "host": {
      "type": "flattened"
    }
  }
}'

6.

curl -H "Content-Type: application/json" -XPUT "http://127.0.0.1:9200/demo-flattened/_doc/1" -d'{
  "message": "[5592:1:0309/123054.737712:ERROR:child_process_sandbox_support_impl_linux.cc(79)] FontService unique font name matching request did not receive a response.",
  "fileset": {
    "name": "syslog"
  },
  "process": {
    "name": "org.gnome.Shell.desktop",
    "pid": 3383
  },
  "@timestamp": "2020-03-09T18:00:54.000+05:30",
  "host": {
    "hostname": "bionic",
    "name": "bionic"
  }
}'

7.

curl -H "Content-Type: application/json" -XGET "http://127.0.0.1:9200/demo-flattened/_mapping?pretty=true"

8.

curl -H "Content-Type: application/json" -XPOST "http://127.0.0.1:9200/demo-flattened/_update/1" -d'{
    "doc" : {
        "host" : {
          "osVersion": "Bionic Beaver",
          "osArchitecture":"x86_64"
        }
    }
}'

9.

curl -H "Content-Type: application/json" -XGET "http://127.0.0.1:9200/demo-flattened/_search?pretty=true" -d'{
  "query": {
    "term": {
      "host": "Bionic Beaver"
    }
  }
}'

10.

curl -H "Content-Type: application/json" -XGET "http://127.0.0.1:9200/demo-flattened/_search?pretty=true" -d'{
  "query": {
    "term": {
      "host.osVersion": "Bionic Beaver"
    }
  }
}'

11.

curl -H "Content-Type: application/json" -XGET "http://127.0.0.1:9200/demo-flattened/_search?pretty=true" -d'{
  "query": {
    "term": {
      "host.osVersion": "Beaver"
    }
  }
}'
```

## 30. Dealing with Mapping Exceptions

```bash
curl -O http://media.sundog-soft.com/es/exceptions.txt

1.
curl -H "Content-Type: application/json" --request PUT 'http://localhost:9200/microservice-logs' \
--data-raw '{
   "mappings": {
       "properties": {
           "timestamp": { "type": "date"  },
           "service": { "type": "keyword" },
           "host_ip": { "type": "ip" },
           "port": { "type": "integer" },
           "message": { "type": "text" }
       }
   }
}'

2.

{"timestamp": "2020-04-11T12:34:56.789Z", "service": "ABC", "host_ip": "10.0.2.15", "port": 12345, "message": "Started!" }


3.

curl -H "Content-Type: application/json" --request POST 'http://localhost:9200/microservice-logs/_doc?pretty' \
--data-raw '{"timestamp": "2020-04-11T12:34:56.789Z", "service": "XYZ", "host_ip": "10.0.2.15", "port": "15000", "message": "Hello!" }'


4.

curl -H "Content-Type: application/json" --request POST 'http://localhost:9200/microservice-logs/_doc?pretty' \
--data-raw '{"timestamp": "2020-04-11T12:34:56.789Z", "service": "XYZ", "host_ip": "10.0.2.15", "port": "NONE", "message": "I am not well!" }'


5.

curl -H "Content-Type: application/json" --request POST 'http://localhost:9200/microservice-logs/_close'

curl -H "Content-Type: application/json" --location --request PUT 'http://localhost:9200/microservice-logs/_settings' \
--data-raw '{
   "index.mapping.ignore_malformed": true
}'

curl -H "Content-Type: application/json" --request POST 'http://localhost:9200/microservice-logs/_open'

6.

curl -H "Content-Type: application/json" --request POST 'http://localhost:9200/microservice-logs/_doc?pretty' \
--data-raw '{"timestamp": "2020-04-11T12:34:56.789Z", "service": "XYZ", "host_ip": "10.0.2.15", "port": "NONE", "message": "I am not well!" }'

curl 'http://localhost:9200/microservice-logs/_doc/lUn8-48B1EL282fGUUia?pretty' # id from above

7.

curl -H "Content-Type: application/json" --request POST 'http://localhost:9200/microservice-logs/_doc?pretty' \
--data-raw '{"timestamp": "2020-04-11T12:34:56.789Z", "service": "ABC", "host_ip": "10.0.2.15", "port": 12345, "message": {"data": {"received":"here"}}}'

8.

curl -H "Content-Type: application/json" --request POST 'http://localhost:9200/microservice-logs/_doc?pretty' \
--data-raw '{"timestamp": "2020-04-11T12:34:56.789Z", "service": "ABC", "host_ip": "10.0.2.15", "port": 12345, "message": "Received...", "payload": {"data": {"received":"here"}}}'

curl 'http://localhost:9200/microservice-logs/_doc/mEkA_I8B1EL282fGiEi2?pretty' # id from above

9.

curl -H "Content-Type: application/json" --request POST 'http://localhost:9200/microservice-logs/_doc?pretty' \
--data-raw '{"timestamp": "2020-04-11T12:34:56.789Z", "service": "ABC", "host_ip": "10.0.2.15", "port": 12345, "message": "Received...", "payload": {"data": {"received": {"even": "more"}}}}'

10.
thousandone_fields_json=$(echo {1..1001..1} | jq -Rn '( input | split(" ") ) as $nums | $nums[] | . as $key | [{key:($key|tostring),value:($key|tonumber)}] | from_entries' | jq -cs 'add')

echo "$thousandone_fields_json"

11
curl --location --request PUT 'http://localhost:9200/big-objects'

curl --request POST 'http://localhost:9200/big-objects/_doc?pretty' \
--header 'Content-Type: application/json' \
--data-raw "$thousandone_fields_json"

12
curl --location --request PUT 'http://localhost:9200/big-objects/_settings?pretty' \
--header 'Content-Type: application/json' \
--data-raw '{
  "index.mapping.total_fields.limit": 1001
}'

curl --request POST 'http://localhost:9200/big-objects/_doc?pretty' \
--header 'Content-Type: application/json' \
--data-raw "$thousandone_fields_json"
```
