<p align="center">
  <img src="img.png" height="300" >
</p>

<p align="center">
  <h1 align="center">Elasticsearch Cheat Sheet "will be updated continuously"</h1>
</p>


you can test these queries on kibana dev tools 
if you will test them with curl or postman don't forget to add correct headers

## General

- cluster/nodes/shards details

  ```sh
  GET /_cat/shards/movies?v
  GET /_cat/nodes?v
  GET /_cat/indices?v
  GET /_cluster/health
  ```

## Index Operations

- add new index named movies

  ```sh
  PUT movies
  ```

- delete index named movies

  ```sh
  DELETE movies
  ```

## Documents CRUD

- add document with specific id

  ```sh
  POST movies/_doc/1
  {
    "name" : "dune",
    "rate": 8.3
  }

  ```

- add document without id (elastic will generate id)

  ```sh
  POST movies/_doc
  {
    "name" : "dune",
    "rate": 8.3
  }
  ```

- add bulk documents

  ```sh
  POST _bulk
  { "index" : { "_index" : "movies", "_id" : "1" } }
  { "name" : "dune", "rate" : 8.2 }
  { "index" : { "_index" : "movies", "_id" : "2" } }
  { "name" : "the matrix", "rate" : 8.5 }
  { "index" : { "_index" : "movies", "_id" : "3" } }
  { "name" : "the matrix 2", "rate" : 7.5 }
  { "index" : { "_index" : "movies", "_id" : "4" } }
  { "name" : "the matrix 3", "rate" : 6.5 }
  ```

- retrieve index document by id

  ```sh
  GET movies/_doc/1
  ```

- update index document by id

  ```sh
  POST movies/_update/1
  {
    "doc" : {
        "rate" : 8.4
    }
  }
  ```

- update index documents by query

  ```sh
  POST movies/_update_by_query
  {
    "conflicts": "proceed",
    "script": {
        "source": "ctx._source.rate -= 0.1"
    }
    , "query": {
        "match_all": {}
    }
  }
  ```

- delete index documents by id

  ```sh
  DELETE movies/_doc/1
  ```

- delete index documents by query

  ```sh
  POST movies/_delete_by_query
  {
    "query": {
        "match_all": {}
    }
  }
  ```
