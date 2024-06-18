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
