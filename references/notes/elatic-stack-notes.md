# Elastic Stack Notes

## What is Elasticsearch?

Elasticsearch is an open source, scalable search engine. It indexes, stores, retrieves, searches, and analyzes data.

## What is the ELK Stack?

- **ELK Stack**: Elasticsearch, Logstash, and Kibana
- **Elastic Stack**: ELK + Beats + X-Pack + other components

---

## Core Concepts

- **Cluster**: Collection of nodes
- **Node**: Physical server or VM running an Elasticsearch instance
- **Index**: Collection of documents
- **Document**: Individual record stored in Elasticsearch
- **Shard**: Partition that distributes documents within an index
- **Replication**: Copies of shards for fault tolerance (e.g., 3x replication allows N-1 failures)

**Examples:**

- 10 nodes, 10 shards: 1 shard per node
- 10 nodes, 10 shards, 3x replication: 3 shards per node

---

## Elasticsearch Installation & Setup

- **Indexing**: Loading/storing data into Elasticsearch
- **Search**: Retrieving data from Elasticsearch

### Indexing Sample Dataset (Shakespeare)

1. **Create an index**: Define number of shards, replication, mapping, analyzer
2. **Load documents**: Add documents to the index
3. **Search documents**: Use URI search, phrase search, multi-match, full-text, filtering, sorting, pagination, etc.

---

## REST API

| Method | Description     |
| ------ | --------------- |
| POST   | Create Document |
| GET    | Fetch Documents |
| PUT    | Update Document |
| DELETE | Delete Document |

**REST Clients**: Curl, Postman

**Sample Documents:**

```json
{
  "_id": "100",
  "message": "Router Connection Error",
  "marker-type": "ERROR",
  "eventTime": "2022/12/20 13:00:00"
}
{
  "_id": "101",
  "message": "Device Connection Error"
}
{
  "_id": "102",
  "message": "Connection Successful"
}
{
  "_id": "103",
  "message": "Connection Failed"
}
```

---

## Inverted Index

- **Router**: 100
- **Connection**: 100, 101, 102, 103

### Stemming

- **Connect, Connected, Connecting, Connections**: 100, 101, 102, 103

### Synonyms

- **Link, Bond**: 100, 101, 102, 103

### N-Gram

- **N Gram**: Er, rr, ro, or
- **N Gram Edge**: E 100, 101; Er 100, 101; Err 100, 101; Erro 100, 101; Error 100, 101

- **Device**: 101
- **Successful**: 102
- **Failed**: 103

**Example Query:**  
`_search?q=message.keyword:"Router Connection Error"`

---

## Relevance / Scoring

---

## Mapping

- **Mapping**: Schema definition for an index (Static/Explicit/Dynamic)
- **Data Types**: Integer, Long, Float, Double, Boolean, Keyword, Text, Date, IP, Object

---

## Analyzer

- **Purpose**: Text analysis
- **Components**:

  - **Character Filter**: Encoding, remove HTML tags, etc.
  - **Tokenizer**: Tokenizes text
  - **Token Filter**: Case-insensitive, stemming, synonyms, stop words

- **Built-in Analyzers**: Standard, Whitespace, Language (e.g., English)

**Example:**

```json
{
  "_id": "100",
  "message": "<html><body>Router Connection Error</body></html>",
  "marker-type": "ERROR",
  "eventTime": "2022/12/20 13:00:00"
}
```

**Input:**  
`<html><body>Router is having a Connection Error</body></html>`

- **Character Filter**
  - IN: `<html><body>Router Connection Error</body></html>`
  - OUT: `Router Connection Error`
- **Tokenizer**
  - IN: `Router Connection Error`
  - OUT: Router, Connection, Error, is, having, a
- **Token Filter**
  - Router → router, routers, routing, modem, switch
  - Connection → connection, connections, connected, link, join
  - Error → error, errors, errored, failure, mistake

---

## Search Types

- URI Search
- Phrase Search
- Term Search
- Query/Filter Search
- Full Text / Fuzzy Search
- Prefix Search
- Wildcard Search
- Auto Complete Search

**Examples:**

- `"connection error"`
- `"conn"`
- `"*nec*"`
- `con`, `conn`, `connect`, `connection`
- `"connection error" slop=2`
- `"fail" fuzziness=1`
- `"of" fuzziness=0`
- `"error" fuzziness=1`
- `"connection" fuzziness=2`

| Fuzziness | Letters |
| --------- | ------- |
| 0         | up to 2 |
| 1         | 3 to 5  |
| 2         | > 5     |

**Query DSL Example:**

```json
{
  "query": {
    "match": {
      "message": "connection error",
      "device-id": ""
    }
  }
}
```

---

## Kibana

- **Purpose**: Search, analyze, and visualize data in Elasticsearch

**Steps:**

1. Go to Stack Management
2. Create Data View
3. Discover Data View
4. Search / Analyze / Visualize

---

## Logstash

- **Purpose**: Collects, parses, transforms, and ingests data into Elasticsearch

### Logstash Setup & Architecture

- **Source**: File, DB, Webservice, Stream, Messaging system, etc.
- **Filter**: Parse, transform, preprocess (e.g., GROK filters)
- **Destination**: File, DB, Elasticsearch, Messaging system, etc.

**Config Example:**  
`logstash.conf | movies-csv.conf`  
Sections: input, filter, output

**Run Logstash** to index Movies CSV dataset

---

### Sample Router Log

```
-300101-00:00:43.886608 [mod=DHCPMGR, lvl=INFO] [tid=4172] Connect to bus daemon... [UNSTRUCTURED]
```

**GROK Pattern:**

```
%{TIMESTAMP:eventTime} [%{WORD:module} %{LOGLEVEL:marker}] [%{NUMBER:tid}] %{WORD:message}
```

**Structured Example:**

```json
{
  "marker": "INFO",
  "tid": 4172,
  "module": "DHCPMGR",
  "eventTime": "300101-00:00:43.886608",
  "message": "Connect to bus daemon"
}
```

---

### Sample Indexed Document

```json
{
  "_index": "movies-csv",
  "_id": "Gb6CPYUBKbLCyLSZ9JhB",
  "_score": 1,
  "_source": {
    "movieid": "2024",
    "title": "Rapture, The (1991)",
    "@version": "1",
    "message": "2024,\"Rapture, The (1991)\",Drama|Mystery\n",
    "@timestamp": "2022-12-23T05:44:28.056198855Z",
    "genres": "Drama|Mystery",
    "event": {
      "original": "2024,\"Rapture, The (1991)\",Drama|Mystery\n"
    },
    "host": {
      "name": "ip-172-31-82-74"
    },
    "log": {
      "file": {
        "path": "/home/ubuntu/movies.csv"
      }
    }
  }
}
```

---

### Apache Log Parsing

**Pattern:**

```
%{IPORHOST:clientip} %{HTTPDUSER:ident} %{USER:auth} \[%{HTTPDATE:timestamp}\] "(?:%{WORD:verb} %{NOTSPACE:request}(?: HTTP/%{NUMBER:httpversion})?|%{DATA:rawrequest})" %{NUMBER:response} (?:%{NUMBER:bytes}|-) %{QS:referrer} %{QS:agent}
```

**COMBINEDAPACHELOG Example:**

```json
{
  "clientip": "216.244.66.246",
  "agent": "Mozilla/5.0 (compatible; DotBot/1.1; http://www.opensiteexplorer.org/dotbot, help@moz.com)",
  "timestamp": "30/Apr/2017:04:28:11 +0000",
  "message": "GET /docs/triton/pages.html HTTP/1.1\" 200 5639"
}
```
