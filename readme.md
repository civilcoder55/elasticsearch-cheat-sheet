<p align="center">
  <img src="img.png" height="300" >
</p>

<p align="center">
  <h1 align="center">Elasticsearch Cheat Sheet </h1>
  <h2 align="center">"will be updated continuously" </h2>
</p>

you can test these queries on kibana dev tools ,
if you will test them with curl or postman don't forget to add correct headers

# Table of contents

- [General](#general)
- [Index Operations](#index-operations)
- [Documents CRUD](#documents-crud)
- [Mapping](#mapping)
- [Term Queries](#term-queries)
- [Full Text Queries](#full-text-queries)
- [Bool Queries](#bool-queries)
- [Query Result](#query-result)

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

- insert document to index with specific id

  ```sh
  POST movies/_doc/1
  {
    "name" : "dune",
    "rate": 8.3
  }

  ```

- insert document to index without id (elastic will generate id)

  ```sh
  POST movies/_doc
  {
    "name" : "dune",
    "rate": 8.3
  }
  ```

- replace document by id if found in index with new document

  ```sh
  PUT movies/_doc/1
  {
    "name" : "dune",
    "rate": 8.3
  }
  ```

- insert bulk documents

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

- scripted update index document by id

  ```sh
  POST movies/_update/1
  {
    "script" : {
        "source" : "ctx._source.rate = 8.3"
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

- upsert index document by id (updateOrInsert)

  ```sh
  POST movies/_update/1
  {
    "script": {
      "source": "ctx._source.rate = 8.3"
    },
    "upsert": {
      "name" : "dune",
      "rate" : 8.3
    }
  }
  ```

- delete index document by id

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

## Mapping

- show index mapping

  ```sh
  GET movies/_mapping
  ```

- add new index with mapping

  ```sh
  PUT movies
    {
    "mappings": {
      "properties": {
        "name": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword"
            }
          }
        },
        "rate": {
          "type": "float"
        },
        "status": {
          "type": "keyword"
        }
        "year": {
          "type": "date",
          "format": "yyyy"
        },
        "genres": {
          "type": "keyword"
        }
      }
    }
  }
  ```

- add new mapping field to existing index

  ```sh
  PUT movies/_mapping
  {
    "properties": {
      "description":{
        "type": "text"
      }
    }
  }
  ```

## Term Queries

- search for single term

  ```sh
  GET movies/_search
  {
    "query": {
      "term": {
        "name.keyword": "dune"
      }
    }
  }
  ```

- search for multi terms

  ```sh
  GET movies/_search
  {
    "query": {
      "terms": {
        "genres": ["Action" , "Drama"]
      }
    }
  }
  ```

- search for array of ids

  ```sh
  GET movies/_search
  {
    "query": {
      "ids": {
        "values": [1 , 2 , 3]
      }
    }
  }
  ```

- search for range of dates or numbers

  ```sh
  GET movies/_search
  {
    "query": {
      "range": {
        "rate": {
          "gte": 6,
          "lte": 10
        }
      }
    }
  }
  ```

- search for not-null field

  ```sh
  GET movies/_search
  {
    "query": {
      "exists": {
        "field": "genres"
      }
    }
  }
  ```

- search for field with prefix

  ```sh
  GET movies/_search
  {
    "query": {
      "prefix": {
        "name.keyword": {
          "value": "the"
        }
      }
    }
  }
  ```

- search for field with wildcards

  ```sh
  GET movies/_search
  {
    "query": {
      "wildcard": {
        "name.keyword": "*rix"
      }
    }
  }
  ```

- search for field with regex

  ```sh
  GET movies/_search
  {
    "query": {
      "regexp": {
        "name.keyword": "[a-z]+"
      }
    }
  }
  ```

## Full Text Queries

- search for text with flexible match

  ```sh
  GET movies/_search
  {
    "query": {
      "match": {
        "description": "neo saves zion"
      }
    }
  }
  ```

  - search for text with phrase

  ```sh
  GET movies/_search
  {
    "query": {
      "match_phrase": {
        "description": "city of zion"
      }
    }
  }
  ```

- search for text in multi fields

  ```sh
  GET movies/_search
  {
    "query": {
      "multi_match": {
        "query": "matrix",
        "fields": ["name","description"]
      }
    }
  }
  ```

## Bool Queries

- search with bool queries aka (where clause ^\_^)

  ```sh
  GET movies/_search
  {
    "query": {
      "bool": {
        "must": [
          {"match": {"name": "matrix"}}
        ],
        "should": [
          {"match": {"name": "the matrix 2"}}
        ],
        "must_not": [
          {"term": {"genres": "Drama"}}
        ],
        "filter": [
          { "range": { "rate": { "gte": "7", "lte": "10" } } }
        ]
      }
    }
  }
  ```

## Query Result

- change result format

  ```sh
  GET movies/_search?format=yaml
  {
    "query": {
      "match_all": {}
    }
  }
  ```

- filter result fields

  ```sh
  GET movies/_search
  {
    "_source": ["name","rate"],
    "query": {
      "match_all": {}
    }
  }
  ```

- change result records size aka (limit ^\_^)

  ```sh
  GET movies/_search
  {
    "size": 1,
    "query": {
      "match_all": {}
    }
  }
  ```

- change result records offset

  ```sh
  GET movies/_search
  {
    "size": 1,
    "from": 1,
    "query": {
      "match_all": {}
    }
  }
  ```

- sort result records

  ```sh
  GET movies/_search
  {
    "query": {
      "match_all": {}
    }, 
    "sort": [
      {"rate": {"order": "desc"}}
    ]
  }
  ```
