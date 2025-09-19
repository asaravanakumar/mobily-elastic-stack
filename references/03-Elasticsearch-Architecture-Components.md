# Elasticsearch Architecture Components and Internals

## Cluster

- The **cluster** is the highest-level container in Elasticsearch consisting of one or more nodes.
- A cluster is identified by a unique name, and all nodes in that cluster must use the same name to communicate.
- It manages the distribution and replication of data and coordinates indexing and search operations across the cluster.
- The cluster state keeps metadata about indices, nodes, shards, mappings, and configuration, ensuring consistency between nodes.

## Nodes

- A **node** is a single running instance of Elasticsearch inside the cluster.
- Nodes hold data and perform data operations (indexing, search).
- Roles of nodes:
  - **Master node**: Controls cluster-wide operations like managing metadata, node additions/removals, and shard allocations.
  - **Data node**: Stores data and handles CRUD, search, and aggregation operations.
  - **Ingest node**: Preprocesses documents before indexing (pipeline execution).
  - **Coordinating node**: Acts as a smart load balancer and query router, forwarding requests to relevant nodes and aggregating results (all non-data nodes act as coordinating).
- Nodes communicate using Elasticsearch transport protocol over TCP.

## Indices

- An **index** is a logical namespace that holds a collection of documents.
- Conceptually similar to a database in RDBMS.
- Each index is made up of one or more shards for scalability.
- Supports settings to configure analyzers, mappings, refresh intervals, and replicas.

## Documents

- A **document** is the basic unit of information indexed in Elasticsearch.
- Stored in JSON format, documents consist of field-value pairs.
- Each document belongs to a type defined in the index (deprecated in newer versions as types are being removed).
- Documents are immutable; updates happen by deleting and reindexing.

## Shards

- An index is divided into **shards** – smaller, more manageable pieces distributed across nodes.
- Each shard is a fully functional Lucene index and can operate independently.
- Sharding enables horizontal scaling by distributing data and load.
- Default shard count is 5 per index but customizable.
- Shards improve parallelism; search queries run across all shards in parallel.

## Replicas

- Elasticsearch provides **replicas** as copies of primary shards.
- Replicas increase fault tolerance and query throughput.
- Replica shards are allocated on different nodes than their primaries for high availability.
- Elasticsearch automatically handles failover if a primary shard goes down by promoting a replica.

## Inverted Index

- At the core of Elasticsearch is the **inverted index**, which maps terms (words) to documents containing those terms.
- Enables efficient full-text searching by optimizing the lookup of terms.
- Supports tokenization and analysis to break text into terms and normalize (lowercase, stemming, stop words).

## Mapping and Data Types

- **Mapping** defines the schema for documents within an index.
- Specifies data types, indexing rules, analyzers, and field configurations.
- Data types include text, keyword, date, numeric types, geo points, and more.
- Dynamic mapping allows Elasticsearch to infer field types at indexing time.

## Query Processing

- Elasticsearch supports a rich query DSL allowing:
  - Full-text queries using analyzed fields.
  - Structured queries for exact matching on keywords.
  - Aggregations for metrics, histograms, and data summaries.
- Queries are distributed across shards and results merged by coordinating nodes.

## Cluster State Management

- Master node manages cluster state updates (new indices, node join/leave, shard relocations).
- This state is propagated to all nodes to ensure consensus.
- Elasticsearch uses a publish-subscribe pattern internally for state updates.

## Data Routing and Balancing

- Documents are routed to shards based on a hash of their ID or a specified routing value.
- Elasticsearch balances shard locations to optimize resource utilization and performance.

## High Availability and Fault Tolerance

- Automatic shard replication and recovery.
- Elastic internally handles node failures by reallocating shards.
- Cluster health indicators (green, yellow, red) reflect shard and replica allocation status.

---

This detailed internal architecture ensures Elasticsearch is resilient, scalable, and fast, making it ideal for diverse applications, including real-time telecom data analysis and observability.
