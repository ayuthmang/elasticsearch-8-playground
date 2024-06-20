## Importing Data with a Script

```bash
curl -O http://media.sundog-soft.com/es8/MoviesToJson.py
python3 MoviesToJson.py >> moremovies.json

curl -H 'Content-Type: application/json' -X DELETE '127.0.0.1:9200/movies?pretty'
curl -H 'Content-Type: application/json' -X PUT '127.0.0.1:9200/_bulk?pretty' --data-binary @moremovies.json

curl -H 'Content-Type: application/json' -X GET '127.0.0.1:9200/movies/_search?q=mary%20poppins&pretty'
```

## Importing with Client Libraries

```bash
# using python client
curl -O http://media.sundog-soft.com/es8/IndexRatings.py
python IndexRatings.py
curl -H 'Content-Type: application/json' -X GET '127.0.0.1:9200/ratings/_search?pretty'
```

## [Exercise] Importing with a Script

```bash
python IndexTags.py
curl -H 'Content-Type: application/json' -X GET '127.0.0.1:9200/tags/_search?pretty'
```

## Logstash

52. Introducing Logstash
53. Installing Logstash
54. Running Logstash

```bash

# testing, restart docker compose logstash ...
docker compose down logstash && docker compose up logstash -d

# cat all indices
curl -X GET 'http://localhost:9200/_cat/indices?v'

# then grab the index name and
curl -X GET 'http://localhost:9200/.ds-logs-generic-default-2024.06.18-000001/_search?pretty'
```

## 55. Logstash and MySQL, Part 1 & 2

```bash
curl -O http://files.grouplens.org/datasets/movielens/ml-100k.zip
unzip ml-100k.zip
```

```sql
CREATE DATABASE movielens;

CREATE TABLE movielens.movies(
	movieId INT PRIMARY KEY NOT NULL,
	title TEXT,
	releaseDate Date
);

-- docker exec to mysql shell and run the following
-- $ docker compose exec -it mysql bash
-- $ head /home/ml-100k/u.item -- make sure file exists
-- $ mysql -uroot -ppassword --local_infile=1

SET GLOBAL local_infile = TRUE;

LOAD DATA LOCAL INFILE '/home/ml-100k/u.item' INTO TABLE movielens.movies CHARACTER SET latin1 FIELDS TERMINATED BY '|' ( movieId, title, @releaseDate ) SET releaseDate = STR_TO_DATE( @releaseDate, '%d-%b-%Y' );

USE movielens;

SELECT * FROM movies WHERE title LIKE 'Star%';

-- create user
-- CREATE USER 'student'@'localhost' IDENTIFIED BY 'password';
-- GRANT ALL PRIVILEGES ON movielens.* TO 'student'@'localhost';
-- FLUSH PRIVILEGES;
```


```bash
# for testing logstash conf
docker compose run logstash bash
# docker compose down logstash && docker compose build && docker compose up logstash -d
logstash -f /usr/share/logstash/pipeline/mysql.conf



curl -H 'Content-Type: application/json' -X GET 'http://localhost:9200/movielens-sql/_search?q=star&pretty'

# if get error "high disk watermark [90%] exceeded on" then run the following and restart the logstash
curl -XPUT "http://localhost:9200/_cluster/settings" \
 -H 'Content-Type: application/json' -d'
{
  "persistent": {
    "cluster": {
      "routing": {
        "allocation.disk.threshold_enabled": false
      }
    }
  }
}'
```

## 58. Importing JSON Data with Logstash

```bash
# https://github.com/coralogix-resources/elk-course-samples

curl -H 'Content-Type: application/json' -X GET '127.0.0.1:9200/demo-csv/_search?pretty'
# curl -H 'Content-Type: application/json' -X DELETE '127.0.0.1:9200/demo-csv?pretty'  # for delete

# restart logstash and enable mount file csv-read-drop.conf
# after run with csv-read-drop.conf

curl -H 'Content-Type: application/json' -X GET '127.0.0.1:9200/demo-csv-drop/_search?pretty' -d '{"size": 1}'
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
      "value" : 4,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "demo-csv-drop",
        "_id" : "aABeNpAB1C7yIqy7tO85",
        "_score" : 1.0,
        "_source" : {
          "log" : {
            "file" : {
              "path" : "/home/csv-schema-short-numerical.csv"
            }
          },
          "age" : 32,
          "timestamp" : "2019-11-16T14:55:13Z",
          "paymentType" : "Mastercard",
          "country" : "China",
          "name" : "Rod Edelmann",
          "gender" : "Male",
          "ip_address" : "131.61.251.254",
          "purpose" : "Clothing",
          "id" : "2",
          "event" : {
            "original" : "2,2019-11-16T14:55:13Z,Mastercard,Rod Edelmann,Male,131.61.251.254,Clothing,China,32\r"
          }
        }
      }
    ]
  }
}
```
