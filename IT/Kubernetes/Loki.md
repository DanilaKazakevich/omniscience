https://aws.amazon.com/blogs/opensource/introducing-fine-grained-iam-roles-service-accounts/
https://grafana.com/blog/2023/12/28/the-concise-guide-to-loki-how-to-get-the-most-out-of-your-query-performance/

https://grafana.com/blog/2023/12/11/open-source-log-monitoring-the-concise-guide-to-grafana-loki/
https://grafana.com/blog/2023/12/11/open-source-log-monitoring-the-concise-guide-to-grafana-loki/
https://grafana.com/docs/loki/latest/get-started/components/
https://utcc.utoronto.ca/~cks/space/blog/sysadmin/GrafanaLokiCardinalityProblem?showcomments

https://lokidex.com/posts/tsdb/

TSDB - time series database. 

https://habr.com/ru/companies/kts/articles/723980/#21

1. **Группа компонентов на запись:** дистрибьюторы и инджесторы, которые пишут в Storage.
    
2. **Группа компонентов на чтение:** query-frontend, querier, index gateway. Это те компоненты, которые занимаются исполнением запросов.
    
3. **Все остальные утилитарные компоненты**: например, кеши, compactor и другие.

**Stream** — уникальный набор лейблов, несмотря на то, что логи могут идти из одного источника.

![[Pasted image 20241020204228.png]]

**Write path.** Точкой входа для записи логов в Loki является сущность под названием «Дистрибьютор». Это stateless-компонент. Его задача — распределить запрос на один или несколько инджесторов.

Инджесторы — это уже stateful-компоненты. Они объединены в так называемый [hash ring](https://en.wikipedia.org/wiki/Consistent_hashing) — систему консистентного хеширования. Она позволяет проще добавлять и удалять инджесторы из этого кластера. Все они подключены к одному Storage.

? индекс это по факту и есть лейблы. И мне кажется Локи хранит отдельно индексы и целиком. Пока не понятна разница чанков и индексов. 


- _Chunks_ contain the actual log data.
- _Indexes_ contain metadata that points to where specific log data can be found within the chunks.


Certainly! I'd be happy to explain the difference between Loki indexes and chunks.

**Overview of Loki**

First, a brief overview: Loki is a log aggregation system developed by Grafana Labs. It's designed to be cost-effective and efficient, especially for storing and querying large volumes of log data. Loki's architecture is inspired by Prometheus but optimized for log data.

**Chunks**

- **What They Are**: Chunks are the fundamental storage units in Loki where the actual log entries are stored.
- **Function**: They contain compressed blocks of log data along with their associated labels (metadata). Each chunk holds a sequence of log lines for a specific stream (a unique combination of labels).
- **Storage**: Chunks are stored in object stores like Amazon S3, Google Cloud Storage, or local filesystems.
- **Lifecycle**: As logs are ingested, they are written to chunks. When a chunk reaches a certain size or age, it is flushed to long-term storage and becomes immutable.

**Indexes**

- **What They Are**: Indexes are data structures that help Loki quickly locate which chunks contain the logs relevant to a query.
- **Function**: They map label sets and time ranges to the chunk identifiers. This mapping enables Loki to efficiently determine which chunks need to be read when processing a query.
- **Storage**: Indexes are typically stored in a key-value store like DynamoDB or Cassandra, or in files if using the BoltDB shipper.
- **Usage in Queries**: When a query is executed, the index is consulted to find the minimal set of chunks that need to be scanned, reducing I/O and speeding up query performance.

**Key Differences**

1. **Content vs. Locator**:
    
    - _Chunks_ contain the actual log data.
    - _Indexes_ contain metadata that points to where specific log data can be found within the chunks.
2. **Storage Medium**:
    
    - _Chunks_ are stored in object stores optimized for large binary data.
    - _Indexes_ are stored in databases optimized for fast key-value lookups.
3. **Role in Querying**:
    
    - _Indexes_ are used first to narrow down the search space by identifying relevant chunks based on labels and time ranges.
    - _Chunks_ are then read to retrieve and process the actual log entries that match the query criteria.

**Analogy**

Think of it like a library:

- **Indexes** are like the library's catalog system. They help you find which books (chunks) might contain the information you're looking for based on topics (labels) and publication dates (time ranges).
- **Chunks** are the actual books on the shelves. Once you've identified which books are relevant, you go to them to read the content you need.

**Why Both Are Necessary**

- **Efficiency**: Without indexes, Loki would have to scan all chunks to find relevant logs, which would be slow and resource-intensive.
- **Scalability**: By separating the index from the chunks, Loki can scale horizontally, handling more data without a linear increase in query time.
- **Cost-Effectiveness**: Storing indexes separately allows for more efficient storage mechanisms for different data types—compressed chunks in object storage and indexes in fast lookup databases.

**In Summary**

- **Indexes** help Loki **find** the relevant chunks quickly by mapping labels and time ranges to chunk identifiers.
- **Chunks** contain the **actual log entries** and are where the data is ultimately retrieved from during queries.



Энкодинг чанков тоже нельзя обойти стороной. Дефолтное значение — gzip, то есть максимальное сжатие. Grafana рекомендует переключиться на snappy — и я по опыту с ними согласен. Тогда логи может и занимают чуть-чуть больше места в сторадже, но становятся более производительными чтения и записи данных.


У нас какой-то пиздец с этими значениями:
- `querier.max_concurrent` показывает, сколько запросов в параллель может обрабатывать один querier. Рекомендовано ставить примерно удвоенное количество CPU, дефолт = 10 (будьте внимательны к этим цифрам).
- `limits_config.max_query_parallelism` показывает, сколько максимум параллельности есть у тенанта. Значения querier.max_concurrent должны матчится с max query parallelism по формуле:  
- `[queriers count] * [max_concurrent] >= [max_query_parallelism]      `В нашем примере должны быть запущены минимум 3 `querier`, чтобы обеспечить параллелизм 24.
