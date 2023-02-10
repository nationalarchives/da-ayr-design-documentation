# OpenSearch Example

Guide to run a local OpenSearch instance and load and query some sample data:

* [1. Run a local OpenSearch instance](#1-run-a-local-opensearch-instance)
* [2. Set up an example bag index](#2-set-up-an-example-bag-index)
* [3. Load sample bag data](#3-load-sample-bag-data)
* [4. Viewing Data](#4-viewing-data)
    * [All root data](#all-root-data)
    * [All bag data](#all-bag-data)
    * [Queries](#queries)
    * [List Indexes](#list-indexes)

## 1. Run a local OpenSearch instance

Run the following from this directory (containing [docker-compose.yml](docker-compose.yml)):

```bash
# To start OpenSearch in Docker compose:
docker-compose up --detach

# To tail logs:
docker-compose logs --follow

# To shutdown containers and remove data (omit --volumes to keep data)
docker-compose down --volumes
```

## 2. Set up an example bag index

Either:

* Add example index with a pre-defined index mapping
* Add example index only (first loaded record will create an index mapping)

To add index with [pre-defined](bag_mappings.json) index mapping: 

```bash
curl \
  -H "Content-Type: application/x-ndjson" \
  -X PUT "https://localhost:9200/bag" \
  -ku "${USER}:${PASSWORD}" \
  --data-binary "@bag_mappings.json"
```

To add index with no pre-defined mapping:

```bash
# PUT bag index
curl \
    -H "Content-Type: application/x-ndjson" \
    -X PUT "https://localhost:9200/bag" \
    -ku "${USER}:${PASSWORD}"
```

To view an index mapping configuration:

```bash
curl \
    -H 'Content-Type: application/json' \
    -X GET "https://localhost:9200/bag?pretty=true" \
    -ku "${USER}:${PASSWORD}"
```

To delete an index and mapping:

```bash
# Delete bag index
curl \
    -X DELETE "https://localhost:9200/bag" \
    -ku "${USER}:${PASSWORD}"
```

> Will show an auto-created definition if a data record is imported prior to
> an index mapping being added manually.

## 3. Load sample bag data

```bash
curl \
    -H "Content-Type: application/x-ndjson" \
    -X PUT "https://localhost:9200/bag/_bulk" \
    -ku "${USER}:${PASSWORD}" \
    --data-binary "@bag.json.bulk"
```

## 4. Viewing Data

### All root data

```bash
curl \
    -X GET "https://localhost:9200/_search?pretty=true" \
    -ku "${USER}:${PASSWORD}"
```

### All bag data

```bash
curl \
    -X GET "https://localhost:9200/bag/_search?pretty=true" \
    -ku "${USER}:${PASSWORD}"
```

### Queries

* `match_all`:

    ```bash
    curl \
        -H 'Content-Type: application/json' \
        -X GET "https://localhost:9200/bag/_search?pretty=true" \
        -ku "${USER}:${PASSWORD}" \
        -d' {"query":{"match_all":{}}}'
    ```

* `match` (will match on "Testing" or "A" in text fields; not keyword fields):

    ```bash
    curl \
        -H 'Content-Type: application/json' \
        -X GET "https://localhost:9200/bag/_search?pretty=true" \
        -ku "${USER}:${PASSWORD}" \
        -d' {"query":{"match":{"bag_info.source_organization":"Testing A"}}}'
    ```

* `match_phrase` (avoids match on "Testing" or "A" in text fields):

```bash
curl \
    -H 'Content-Type: application/json' \
    -X GET "https://localhost:9200/bag/_search?pretty=true" \
    -ku "${USER}:${PASSWORD}" \
    -d' {"query":{"match_phrase":{"bag_info.source_organization":"Testing A"}}}'
```

### List Indexes

```bash
curl \
    -X GET "https://localhost:9200/_cat/indices?pretty=true" \
    -ku "${USER}:${PASSWORD}"
```
