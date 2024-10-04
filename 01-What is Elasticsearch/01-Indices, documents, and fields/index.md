# Documents and fields

Elasticsearch serializes and stores data in the form of JSON documents. A document is a set of fields, which are key-value pairs that contain your data. Each document has a unique ID, which you can create or have Elasticsearch auto-generate.
   以JSON文档的形式保存
   示例：

```json
{
  "_index": "my-first-elasticsearch-index",
  "_id": "DyFpo5EBxE8fzbb95DOa",
  "_version": 1,
  "_seq_no": 0,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "email": "john@smith.com",
    "first_name": "John",
    "last_name": "Smith",
    "info": {
      "bio": "Eco-warrior and defender of the weak",
      "age": 25,
      "interests": [
        "dolphins",
        "whales"
      ]
    },
    "join_date": "2024/05/01"
  }
}
```

# Data and metadata

数据和元数据

- `_source`. Contains the original JSON document.
  数据内容
- `_index`. The name of the index where the document is stored.
- `_id`. The document’s ID. IDs must be unique per index.

## Mappings and data types

mapping定义了数据类型
A mapping defines the `data type` for each field, how the field should be indexed, and how it should be stored.
- `Dynamic mapping` Elasticsearch 自动指定
- `Explicit mapping` 自己定义。商用推荐此项。
