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
Without specifying body, `Search API` behave like using `Match All`. It will get all data in index.
Paging and Sorting also can be added using request body.
```
    POST http://localhost:9200/products/_search
    Content-Type: application/json

    {
        "query":{
            "match_all":{}
        },
        "size": 10,
        "from": 10,
        "sort":[
            {
                "username" : {
                    "order":"desc"
                }
            }
        ]
    }
```
## Count Query
To count total document in a index
```
POST http://localhost:9200/categories/_count
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
Kind of query to match exact value, like username, product id, price, etc. Not suitable for text, because it has text analysis that make the text data inconsistent.
```
    POST http://localhost:9200/customer/_search
    Content-Type: application/json

    {
        "query":{
            "term":{
                "gender" : "Female"
            }
        },
    }
```
If we want to query text using `term` we should act like we have standard analyize the text that we query, like lowercasing and remove all symbols, because the text is already saved in lucene document like that, this approach will 'exact match' the text.

### Note
Text in lucene document will be saved with tokenization by standard analyzer. Example:
```
"Hello this is text!" -> ["hello", "this","is","text"]
```
If the data type is keyword and not text it will retain raw data
```
"Hello this is text!" -> ["Hello this is text!"]
```
## Match Query
Similiar as term query but using  `standard analyzer` matching the data type, such as text.
```
    ### Match Query
    POST http://localhost:9200/customer/_search
    Content-Type: application/json

    {
        "query":{
            "match":{
                "banks.name" : "BCA"
            }
        },
    }
```
This will match "BCA" and "BCA Digital" document, banks.name is text attribute.

We also can use `Operator`. Default operator is `OR`, so when we match text "BCA DIGITAL" it will search "bca" OR "digital" in index. To modify operator:
```
    ### Match Query
    POST http://localhost:9200/customer/_search
    Content-Type: application/json

    {
        "query":{
            "match":{
                "banks.name" : {
                    "query": "bca digital",
                    "operator": "AND"
                }
            }
        },
    }
```


## Terms Query
Just like `IN` query in SQL, and it is similiar to `Term Query`, only matching exact value also not suitable for text data type.
```
    POST http://localhost:9200/customer/_search
    Content-Type: application/json

    {
        "query": {
            "terms": {
                "username": [
                    "username1",
                    "username2",
                    "username3"
                ]
            }
        }
    }
```

## Boolean Query
Query to support multiple field condition

| Occur            | Description                                   |
|------------------|------------------------------------------------------------|
| must             | Query must appear in the search results and contribute to the score. |
| filter           | Query must appear in the search results but does not contribute to the score. |
| must_not         | Query must not appear in the search results.                |
| should           | Query may or may not appear in the search results.           |

`must` and `filter` is jut like `AND` and `should` is just like `OR`.
Score contribute to the result relevance. If we prefer sorting manually, `filter` is a good choice.
`must` is more computational expensive than `filter` because it calculate the relevance score.


```
    ### Search Customer Where Hobbies = 'Gaming' AND Banks Name = 'BCA'
    POST http://localhost:9200/customer/_search
    Content-Type: application/json

    {
        "query": {
            "bool": {
                "filter": [
                    {
                        "term": {
                            "hobbies": "gaming"
                        }
                    },
                    {
                        "match": {
                            "banks.name": {
                                "query" : "bca digital",
                                "operator": "AND"
                            }
                        }
                    }    
                ]
            }
        }
    }
```

The boolean can be combined under `bool` object. There are *minimum should match* rule when using `should`, when we only use `should` the *msm* is 1 (one), but when using with other boolean it becomes 0 (zero). But we can modify this

```
    POST http://localhost:9200/customer/_search
    Content-Type: application/json

    {
    "query": {
        "bool": {
            "must": [
                {
                "term": {
                    "hobbies": "gaming"
                }
                }
            ],
            "should": [
                {
                "term": {
                    "banks.name": "bca"
                }
                },
                {
                "term": {
                    "banks.name": "bni"
                }
                }
            ],
            "minimum_should_match": 1
        }
    }
    }
```
Query above will resulting all document that has `hobbies = gaming` with `banks.name = bca` or `banks.name = bni`. If we not set the *msn*, the result will also return document that has `hobbies = gaming` only, beside that has `banks.name: bca` or `banks.name = bni`, this because the *msn* is set to be zero by default.

## Another Query
To be read: range, wildcard, regexp, geo search, exist, etc. 
- Elasticsearch Query DSL (Domain Specific Language)

## Search After
Elasticsearch limit query result only 10k document by default. This limitation can be overrided but it is not recommended because it will be more computationally expensive. This because *Deep Paging Problem*.

The previous alternate to handle this is using `scroll api` but it is no longer recommended. Now `search after` is preferable.

`search after` will always query documents in first page, and using last document sort index from the previous query for its offset. Of course it needs sorting.
```
    POST http://localhost:9200/categories/_search
    Content-Type: application/json

    {
        "size": 100,
        "from": 0,
        "sort": [
            {
                "id": {
                    "order": "asc"
                }
            }
        ],
        "query": {
            "match_all": {}
        },
        "search_after": [
            "10087"
        ]
    }
```
From query above, "10087" is obtained from sort field of last document from previous query. To take all documents, we just iterate the query, use previous last sort index to do next query using `search after` parameter.

# Advanced Indexing Features
## Score
By default, query result will be sorted using score, this is a metric that measure relevance of document to the request query. Score calculated using BM25 algorithm.

We can know where the score come up with `Explain API
```
    POST http://localhost:9200/customer/_explain/username126
    Content-Type: application/json

    {
    "query": {
        // query
    }
    }
```
## Boost Score
Is a weight assigned to each query that determine the final score, the default boost is 1. We can modify it
```
    POST http://localhost:9200/customer/_search
    Content-Type: application/json

    {
        "query": {
            "bool": {
                "should": [
                    {
                        "term": {
                            "banks.name": "bca"
                        }
                    },
                    {
                        "term": {
                            "banks.name": {
                                "value": "bni",
                                "boost": 2
                            }
                        }
                    }
                ]
            }
        }
    }
```
## Flattened Field
Is a Elasticsearch feature, suitable for data type that has dynamic field so we are not defining many mapping. Flatened field saving all the value from each key as `keyword` type in a flat (usual array) field in lucene document.
```
    PUT http://localhost:9200/customer/_mapping
    Content-Type: application/json

    {
        "properties": {
            "labels": {
                "type": "flattened"
            }
        }
    }
``` 
Example of dynamic field, this key-value inside `labels` can be any data type
```
    POST http://localhost:9200/customer/_update/username1
    Content-Type: application/json

    {
        "doc": {
            "labels": {
                "priority": "vip",
                "discount": "10% discount",
                "complaint": "always complaint",
                "priority": 1
            }
        }
    }
```
Flatened field save nested attribute differ from embbeded document, it flatten the nested attribute with only its values.
```
### in lucene.doc
### labels : ["vip","10% discount", "always complaint", "1"] 
```
So we can search it like normal term
```
    POST http://localhost:9200/customer/_search
    Content-Type: application/json

    {
        "query" : {
            "term" : {
                "labels": "vip"
            }
        }
    }
```
But we can also search by the key (nested object) like this
```
    POST http://localhost:9200/customer/_search
    Content-Type: application/json

    {
        "query": {
            "bool": {
                "should": [
                    {
                        "term": {
                            "labels.priority": "vip"
                        }
                    },
                    {
                        "term": {
                            "labels.priority": "regular"
                        }
                    }
                ]
            }
        }
    }
```
A flattened field is a flexible field that behaves similarly to an embedded document when querying it. The trade-off is that the data type becomes exclusively `keyword`.
## Nested Field
A special data type to store object in an array as independent document. A query that applied to nested field is considered a very expensive operation.
```
    PUT http://localhost:9200/parents/_mapping
    Content-Type: application/json

    {
        "properties": {
            "first_name": {
                "type": "text"
            },
            "last_name": {
                "type": "text"
            },
            "children": {
                "type": "nested",
                "properties": {
                    "first_name": {
                        "type": "text"
                    },
                    "last_name": {
                        "type": "text"
                    }
                }
            }
        }
    }
```
To query, we should use nested query
```
    POST http://localhost:9200/parents/_search
    Content-Type: application/json

    {
    "query": {
        "nested": {
            "path" : "children",
            "query":{
                "bool": {
                    "must": [
                        {
                            "match": {
                                "children.first_name": "Joko"
                            }
                        },
                        {
                            "match": {
                                "children.last_name": "Nugraha"
                            }
                        }
                    ]
                }
            }
        }
    }
    }
```
## Multi Fields
From one field to become several different data types. When a field has multi field, the multi field will be automatically inserted when we insert data to the main field. We define multi fields schema in the mapping
```
    PUT http://localhost:9200/categories/_mapping
    Content-Type: application/json

    {
    "properties": {
        "name": {
        "type": "text",
        "fields": {
            "raw": {
            "type": "keyword"
            }
        }
        }
    }
    }
```
To query, access it like this
```
    POST http://localhost:9200/categories/_search
    Content-Type: application/json

    {
    "query": {
        "match": {
        "name.raw": "Laptop Murah"
        }
    }
    }
```
This usefull to provide another field that serve some purpose, like for sorting we should have field that has keyword data type from the original text type.
```
    POST http://localhost:9200/categories/_search
    Content-Type: application/json

    {
    "query": {
        "match": {
        "name": "Laptop"
        }
    },
    "sort": [
        {
        "name.raw": {
            "order": "asc"
        }
        }
    ]
    }
```
From the query above, `name` is text and `name.raw` is keyword.

## Update by Query API
In contrast to the scenario where we have defined a schema as `multi-field`, upon data insertion into the main field, data will also be inserted into the `multi-field`. However, if we later update a field to be a multi field, the `multi-field` in existing documents will remain empty by default. `Update by Query` can handle this issue.

```
    POST http://localhost:9200/products/_update_by_query
    Content-Type: application/json
    {
        "query": {
            "match_all": {}
        }
    }
```
Query above will trigger an update to the `lucene document`, and the data from the main field will be populated into the `multi-field`. This method enables the updating of the `multi-field` without the need for manual updates or reindexing of the data.

Those query will updated all document, but we also can update only the document that match the query (if we specify the query more specific).
## Delete by Query API
Every document that match to the query, will be deleted.
```
    POST http://localhost:9200/categories/_delete_by_query
    Content-Type: application/json
    {
        "query": {
            "match": {
                "name": "salah"
            }
        }
    }
```

# Administrative Operations
## Cat API
Cat stand for *Compact and aligneed text*, a query for see Elasticsearch information like index, shard, nodes, etc.
```
    ### LIST CAT API
    GET http://localhost:9200/_cat/

    ### Using API
    GET http://localhost:9200/_cat/indices?v
```
Full documentation here https://www.elastic.co/guide/en/elasticsearch/reference/current/cat.html
## Snapshot
Is a backup for Elasticsearch cluster
### Snapshot Repository
Location where we save snapshot like AWS S3, GCP Storage, Azure Storage, or using File Storage (local)

To make local repository, modify elasticsearch.yml first and restart. Dont forget to create `backup` folder in Elasticsearch folder.
```
    cluster.name: diccode-cluster
    node.name: diccode-1
    xpack.security.enabled: false
    http.port: 9200
    path.data: data
    path. logs: logs
    # lokasi repository
    path.repo: backup
```

Make repository
```
    PUT http://localhost:9200/_snapshot/first_backup
    Content-Type: application/json

    {    
        "type": "fs",
        "settings": {
            "location": "first_backup"
        }
    }
```
List repository
```
    GET http://localhost:9200/_snapshot
```

Create snapshot, if we not specify the `indices` list, it will snapshot all index
```
    PUT http://localhost:9200/_snapshot/first_backup/snapshot1
    Content-Type: application/json

    {
        "indices": [],
        "metadata": {
            "taken_by": "dicky",
            "taken_because": "backup before upgrading"
        }
    }
```

Listing the snapshot
```
    ### get snapshot
    GET http://localhost:9200/_snapshot/first_backup/snapshot1
    ### list snapshots
    GET http://localhost:9200/_cat/snapshots?v
```
## Restore