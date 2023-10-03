Repository notes from [Tutorial Elasticsearch Dasar (Bahasa Indonesia)](https://youtu.be/JfW7tg0yWsc?si=-WyReWV3ZBTCBFfC) video by Programmer Zaman Now on Youtube.

# Introduction
ElasticSearch is a popular **search engine database** currently in use, widely adopted by many e-commerce platforms in Indonesia. ElasticSearch is built from an Information Retrieval library named Apache Lucene, developed using the Java programming language.

## RDBMS Terms vs ElasticSearch
| Relational DB Term | Elasticsearch Term |
|--------------------|--------------------|
| Database           |                    |
| Table              |   Index |
| Column             | Field / Attribute  |
| Row / Record       | Document (JSON)    |
| Join Table         | Embedded Document, Reference |
| SQL                | JSON |

## Embedded Document
We store data in JSON format, using either the reference or embedded approach. Embedded document is where a document's field has nested document. Elasticsearch automatically converts the JSON document structure into a Lucene document, formatted as a `Map<String, List<T>>`. Below is the example of Embedded Document.

<table>
<tr>
<th>elasticsearch.json</th>
<th>lucene.doc</th>
</tr>

<tr>
<td>

```json
{
    "_id": "12345",
    "first_name" : "Eko",
    "last_name" :  "Khannedy",
    "email": "eko@example.com",
    "age": 25,
    "hobbies": [
        "Coding" ,
        "Reading" ],
    "addresses" : [
        {
            "street": "Jl. Belum Jadi",
            "city": "Jakarta",
            "country": "Indonesia"
        },
        {
            "street": "Jalan Belakang",
            "city": "Subang",
            "country": "Indonesia"
        }
    ]
}
```
</td>
<td>

```json
{
    "_id": ["12345"],
    "first_name" : ["Eko"],
    "last_name" : ["Khannedy"],
    "email": ["eko@example.com" ],
    "age": [25],
    "hobbies": ["Coding", "Reading"],
    "addresses.street": ["Jl. Belum Jadi", "Jalan Belakang"],
    "addresses.city" : ["Jakarta", "Subang"],
    "addresses.country": ["Indonesia"]
}
```

</td>
</tr>
</table>

# Setup and Configuration
## Instalation
Download archive file from official website (for Linux), then extract
```bash
    wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.10.2-linux-x86_64.tar.gz
    tar -xzvf elasticsearch-8.10.2-linux-x86_64.tar.gz
```

Open the extracted folder then configure `config/elasticsearch.yml`
```yml
    cluster.name: diccode-cluster
    node.name: diccode-1
    xpack.security.enabled: false
    http.port:9200
    path.data: data
    path.logs: logs
```

## Running
ElasticSearch will run on port 9200.
```bash
    bin/elasticsearch
```
Below is the respons from `localhost:9200`.
```JSON
    {
    "name" : "diccode-1",
    "cluster_name" : "diccode-cluster",
    "cluster_uuid" : "mBppz7bVQcqzBS_CuhraXQ",
    "version" : {
        "number" : "8.10.2",
        "build_flavor" : "default",
        "build_type" : "tar",
        "build_hash" : "6d20dd8ce62365be9b1aca96427de4622e970e9e",
        "build_date" : "2023-09-19T08:16:24.564900370Z",
        "build_snapshot" : false,
        "lucene_version" : "9.7.0",
        "minimum_wire_compatibility_version" : "7.17.0",
        "minimum_index_compatibility_version" : "7.0.0"
    },
    "tagline" : "You Know, for Search"
    }
```


# Index Management
## Index
Creating index name should be using lowercase, no special character (except +, -, _). Additionally we can add apps name as prefix (optional) to differentiate index since ElasticSearch do not has database.
```
    PUT http://localhost:9200/products
```

Listing all index
```
    GET http://localhost:9200/_cat/indices?v
```
Deleting index automatically delete all documents inside it
```
    DELETE http://localhost:9200/products
```

## Dynamic Mapping
Ability of ElasticSearch to automatically map or convert data type. Especially from string to either date, float, long, or text (in order).

- Date mapping, automatically active with yyyy/MM/dd HH:mm:ss format. But we can modify it with multiply dynamic formats
```
    PUT http://localhost:9200/customers/_mapping
    Content-Type: application/json

    {
        "date_detection": true,
        "dynamic_date_formats" : [
            "yyyy-MM-dd HH:mm:ss",
            "yyyy-MM-dd",
            "yyyy/MM/dd HH:mm:ss",
            "yyyy/MM/dd"
        ]
    }
```
- Numeric mapping, automatically inactive, so we need to activate it first
```
    PUT http://localhost:9200/customers/_mapping
    Content-Type: application/json

    {
        "numeric_detection": true
    }
```
- Get all the mapping
```
    GET http://localhost:9200/customers/_mapping
```
```
    HTTP/1.1 200 OK
    X-elastic-product: Elasticsearch
    content-type: application/json
    content-encoding: gzip
    transfer-encoding: chunked

    {
    "customers": {
        "mappings": {
        "dynamic_date_formats": [
            "yyyy-MM-dd HH:mm:ss",
            "yyyy-MM-dd",
            "yyyy/MM/dd HH:mm:ss",
            "yyyy/MM/dd"
        ],
        "date_detection": true,
        "numeric_detection": true
        }
    }
    }
```
## Alias
In ElasticSearch, we cannot alter column table like many other database. So when there are schema changes, usually we create new index. Alias will help client so they not confuse when we change the index name.
```
    POST http://localhost:9200/_aliases
    Content-Type: application/json

    {
        "actions": [
            {
                "add": {
                    "index": "customers",
                    "alias": "customer"
                }
            }
        ]
    }

```

## Reindex API
To move data from index to other index. Usually used when new index is created then we should move the data from previous index. Don't forget to change the alias to new index.
```
    POST http://localhost:9200/_reindex
    Content-Type: application/json

    {
        "source": {
            "index": "orders"
        },
        "dest": {
            "index": "orders_v2"
        }
    }
```
## Explicit Mapping
Define index schema manually. Its okay to define schema and using dynamic mapping together.
```
    PUT http://localhost:9200/customers_v2/_mapping
    Content-Type: application/json

    {
        "numeric_detection": true,
        "date_detection": true,
        "dynamic_date_formats": [
            "yyyy-MM-dd HH:mm:ss",
            "yyyy-MM-dd",
            "yyyy/MM/dd HH : mm:ss",
            "yyyy/MM/dd"
        ],
        "properties": {
            "username": {
                "type": "keyword"
                },
        }
    }
```

## Object Field
Defined in schema using type properties, then make nested attribute. Existing schema can be updated using `PUT`.
```
    PUT http://localhost:9200/customers_v2/_mapping
    Content-Type: application/json
    {
        "properties": {
            "address": {
                "properties": {
                    "street": {
                        "type": "text" 
                    },
                    "city": {
                        "type": "text" 
                    },
                    "province": {
                        "type": "text"
                    },
                    "country": {
                        "type": "text"
                    },
                    "zip_code": {
                        "type": "keyword"
                    }
                }
            }
        }
    }
```

## Array Field
As lucene document save data as array, defining array in schema is just like normal field.
```
    PUT http://localhost:9200/customers_v2/_mapping
    Content-Type: application/json

    {
        "properties": {
            "hobbies": {
                "type":"text"
            },
            "banks": {
                "properties": {
                    "name": {
                        "type": "text"
                    },
                    "account_number": {
                        "type": "keyword"
                    }
                }
            }
        }
    }
```

# Document Operations
## Create API
Adding document to index with unique id
```
    POST http://localhost:9200/customers/_create/khannedy
```
Upon creating the initial document in an index, the schema is also defined. Therefore, adding subsequent documents with differing data types will result in an error.

It is recomended to use `GET API` before using `CREATE API` to avoid conflict when the document is already exist.

## Index API
Simmiliar to the `CREATE API` but has replace behavior when document `_id` exist. When replaced, document will increase its `_version`
```
    POST http://localhost:9200/products/_doc/3
    Content-Type: application/json

    {
        "name": "Pop Mie Rasa Bakso",
        "price": 2500
    }
```
```json
    {
        "_index": "products",
        "_id": "3",
        "_version": 2,
        "result": "updated",
        "_shards": {
            "total": 2,
            "successful": 1,
            "failed": 0
        },
        "_seq_no": 5,
        "_primary_term": 4
    }
```

## Get API
Retrieve document with specific id, with metadata
```
    GET http://localhost:9200/customers/_doc/khannedy
```
Get only source, without metadata
```
    GET http://localhost:9200/customers/_source/khannedy
```
Check if document exist, it will response status either `200 OK` or `404 Not Found` with empty body
```
    HEAD http://localhost:9200/customers/_doc/khannedy
```
Multiget, to retrieve several document at once either from same index or various index
```
    # Same index
    POST http://localhost:9200/products/_mget
    Content-Type: application/json

    {
        "ids": [
        "1","2"
        ]
    }
```
```
    # Various index
    POST http://localhost:9200/_mget
    Content-Type: application/json

    {
        "docs": [
            {
                "_id": "1",
                "_index": "orders"
            },
            {
                "_id": "khannedy",
                "_index": "customers"
            }
        ]
    }
```

## Update API
When we only want to update some of attribute without re-send and replace all attribute, this also usefull to avoid deleting another attribute when sending document with same `_id` with only some attribute. This will also increase document `_version`.

```
    POST http://localhost:9200/products/_update/5
    Content-Type: application/json

    {
        "doc": {
        "price": 30000000
    }
    }
```

## Delete API
```
    DELETE http://localhost:9200/customers/_doc/spammer
```

## Bulk API
Streamline Elasticsearch responses to avoid answering each request individually, especially when dealing with very large requests. This API can combine many operations at once. When one request fail in bulk, it won't continue. Please also note that each action/body require newline at the end.
```
    ### Bulk documents
    POST http://localhost:9200/_bulk
    Content-Type: application/json

    { "create" : { "_index" : "customers", "_id" : "joko" } }
    { "name" : "Joko Morro", "register_at" : "2023-01-01 00:00:00" }
    { "index" : { "_index" : "customers", "_id" : "budi" } }
    { "name" : "Budi Nugraha", "register_at" : "2023-01-01 00:00:00" }
    { "update" : {"_index" : "products", "_id" : "1"} }
    { "doc" : {"price" : 2500} }
    { "create" : { "_index" : "customers", "_id" : "spammer" } }
    { "name" : "Spammer", "register_at" : "2023-01-01 00:00:00" }
    { "delete" : { "_index" : "customers", "_id" : "spammer" } }

```

# Searching and Querying

## Search API
When we want to get/search documents not using _id.
```
    POST http://localhost:9200/products/_search
```
Pagination, helping search document to determine document offset and limit. Default offset is 0 and limit is 10.
```
    POST http://localhost:9200/products/_search?size=1&from=0
```
Sorting, if we want sort in multiply field, use comma as delimiter
```
    POST http://localhost:9200/products/_search?sort=price:asc
```

## Source Field
Beside being stored in Lucene Document, JSON that we have sent is saved in `_source` field originally. Thats why the size of data might be doubled because ElasticSearch save the data in both Lucene Document and in Source Field.
```json
    {
    "_index": "customers",
    "_id": "khannedy",
    "_version": 1,
    "_seq_no": 0,
    "_primary_term": 3,
    "found": true,
    "_source": {
        "name": "Eko Kurniawan Khannedy",
        "register_at": "2023-01-01 00:00:00"
    }
    }
```

## Select Field
Feature to include (like SQL SELECT) or exclude field in `_source`.
```
    GET http://localhost:9200/order/_doc/1?_source_includes=total,customer_id
```
```
    POST http://localhost:9200/products/_search?_source_excludes=price
```

## Term Query
## Match Query
## Terms Query
## Boolean Query
## Another Query
## Search After

# Advanced Indexing Features
## Score
## Boost Score
## Flattened Field
## Nested Field
## Multi Fields

# Administrative Operations
## Cat API
## Snapshot
## Restore