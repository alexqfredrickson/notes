<!-- from Designing Data-Intensive Applications by Martin Kleppmann -->

# Chapter 1: Reliable, Scalable, and Maintainable Applications

*Data-intensive* applications are applications that aren't *compute-intensive*, but do have a large volume of potentially complex data which changes often. They are built from:
  - *Databases* to store data for later
  - *Caches* to remember expensive operations and speed up reads
  - *Search indexes* to allow search
  - *Stream processing* to send messages to other processes, which are handled asynchronously
  - *Batch processing* to crunch large amounts of data

Three main concerns for data systems are:
  - *Reliability*, or working correctly even with adversity and error
  - *Scalability*, or the ability to grow
  - *Maintainability*, or the ability for engineers to productively work with a system

## Reliability

*Fault-tolerance* means resilience to certain kinds of faults - but not total system failure. Faults can be triggered directly to test error handling (i.e. Chaos Monkey). 

Redundancy can help prevent hardware faults. AWS prioritizes flexibility and elasticity over single-machine reliability (... citation needed?). There's tons of different types of software faults, obviously. Human (or "operator" failures) cause issues for ISPs (configuration errors from operators are the leading source of faults). They can be obviated by non-prod sandbox environments, well-designed abstractions and APIs, testing, quick configuration roll-backs, monitoring, and good management.

## Scalability

Scalability is a function of *load* - like - requests per second, read/write ratios, simultaneously active users, hit rates on caches, etc.

In the case of Twitter, users post tweets to a global tweet database, and view tweets from people they follow from that database. The native relational database approach doesn't work quite as well when the number of tweets reaches hundreds of billions, so they switched to a mode where each user gets a "home timeline cache" - and when a tweet is posted, it is posted only to their followers' home timeline caches. If a user has 30 million followers, this is an expensive approach. So they moved to a hybrid approach: celebrities are exempted from "home timeline cache fan-out".

Batch processing systems (like Hadoop) use *throughput* as an important performance metric - i.e. records-processed-per-second. Online systems use response time. Median response time is a good metric (*not* mean).

*Elastic* systems can automatically increase compute resources when load increases are detected (auto-scaling).

Distributed *stateless* operations are straightforward, but *stateful* operations are more complicated - so until recently, databases were generally one-node and scaled-up-not-scaled-out.

## Maintainability

Maintainability consists of:

  - *Operatability*, or how easy it is to keep a system running smoothly
  - *Simplicity*, or how easy it is for a new engineer to learn a system
  - *Evolvability*, or how interchangeable the system's parts are (or how easy they are to change)

*"Good operations can often work around the limitations of bad software, but good software cannot run reliability with bad operations".*

Good operations looks like:

  - Monitoring system health and quickly restoring service
  - Tracking down causes of problems like system failures or degraded performance
  - Keeping servers and applications patched and up-to-date
  - Keeping tabs on how systems affect *each other*
  - Capacity planning (and other forms of future problem anticipation)
  - Automated deployments and configuration management
  - Complex maintenance tasks like moving applications across different OSes
  - Maintaining security of systems when configuration changes are made
  - Defining production change processes
  - _Preserving an organization's knowledge about a system even if the good engineers leave_
  - Avoiding dependency on individual machines, so machines can actually be taken down and/or patched
  - Good documentation 
  - No surprises
  - Self-healing, with the ability for administrators to override default behavior
 
# Chapter 2: Data Models and Query Languages

Real-world things are modeled as objects, or data structures. APIs manipulate those structures. When stored, the structures are converted to general-purpose data models, such as JSON, XML, YAML, database tables, graph models, etc. Database engineers represent those in terms of memory or network or disk-persisted data structures. Hardware engineers represent bytes through electrical currents, light pulses, magnetic fields, etc. All of these cohere into tiered layers of complex abstractions.

SQL is the most common data model. Network models and hierarchical models are alternatives for data storage and querying. NoSQL (or non-relational databases) were designed for large datasets requiring high write throughput. ORM frameworks arose from an "impedance mismatch" between application objects and database tables.

Database IDs are used for consistent style and spelling, avoiding ambiguity (if things are same-or-similarly named), ease of updating redundant data, localization support, and enhancing search operations. Removing redundancy/duplication in databases is essentially what normalization is all about.

One-to-many relationships are difficult for document databases such as MongoDB/CouchDB/RethinkDB/etc. Many-to-many relationships are annoying for JSON because you can't form tree-like structures without duplicating data. 

JSON is a form of a *hierarchical model* in the sense that it represents data as a tree. *Network models* would model objects in graph structures, with complicated series of pointers to other objects through the use of *access paths*. *Relational models* also use access paths (sort of), in the form of query optimizers, which do that automatically. But in network models, this would be done manually, which made it hard to update records, but very optimal for slow network drives (i.e. tape records in the 1970s). 

Document data models are nice choices when flexible schemas are required, or the underlying data model of the application closely resembles the back-end data model. For example: a database which records log lines or events. Relational models are better where one-to-many or many-to-many relationships are required, or when you need joins. If application data is fundamentally *tree-like*, then document models aren't a bad choice. If a database natively supports JSON/XML datatypes, then it can be considered a hybrid document/relational model.

SQL uses a declarative syntax. MapReduce uses a hybrid imperative/declarative functional syntax, which is designed to target distributed clusters of machines. `Map` collects data, and `Reduce` filters down. This requires two queries instead of one (vis-a-vis SQL).

Graph models are highly suitable for data with many many-to-many relationships. Some graph databases use *property models*, and other use *triple-store models*. 

Property graph models are directed, with key/value pairs at vertices. Edges are labeled, and themselves have key/value pairs. Cypher Query Language is one example: `(Idaho) -[:WITHIN]-> (USA)` is a valid statement in a Cypher `CREATE` function. If you use a lot of recursive common table expressions (lol), then Cypher might be a better choice than MS-SQL. 

In triple-store models, all information is stored in subject-predicate-object notation - like - "Jim likes bananas". *Resource Description Framework* (RDF) tried to formalize this for semantic web purposes. *SPARQL Protocol and RDF Query Language* (SPARQL) was designed to query RDF documents. Hadoop uses Datalog-type queries - like `name(namerica, 'North America')`.

# Chapter 3: Storage and Retrieval

You can make a database with a text file and a shell script that does an `echo >>` for sets and a `grep | sed` for gets with a `tail -n 1` because there's no delete/overwrite methods. Many databases use append-only log files. But the `grep | sed` has poor performance at *O(n)*. 

*Indexes* are used to speed up reads. One simple indexing method is to use an in-memory hash map, mapping database keys to their byte offsets from the beginning of the logfile. This slows down writes a little.

*Compaction* is used to prevent the database from filling up. Data is partitioned into *segments*, where each segment is a collection of key-value pairs. Segments are periodically merged together, and the older keys are forgotten. Then, during a lookup, each segment is assigned its own index, and the search iterates over each segment until a key is found.

CSVs are bad for logs - binary format is better for the aforementioned reasons. When a record needs to be deleted, a new "delete" record is added to the data file, and the key is deleted during the next segment merge. If a database crashes, then the in-memory hash maps are lost. Those can be persisted to disk instead. Checksums can be used to ensure that records aren't *partially* written (i.e. the database crashes during a write). There is generally only one write thread, and multiple read threads (as data file segments are read-only and immutable).

Sequential writes are faster than random writes - even on SSDs. Crash recovery is easier if writes are sequential. Segment merges avoid fragmentation. Therefore, the append-only model is preferable to the update-in-place model.

If keys are sorted within segments, they can be merge-sorted into new compacted segments. This is called a *Sorted String Table* or an *SSTable*. In-memory indices can still help if some key byte offsets are marked, but you can `grep` between them, so you don't need a lot of them.  

Using the SSTable approach, incoming writes can be added to in-memory balanced tree structures (e.g. red-black trees), or *memtables*. Memtables are periodically written to disk as SSTables once some threshold is hit. Reads come in through the memtables, and reads *fall back* on the SSTable structures. SSTables are then periodically compacted. If the memtables crash before the SSTable segments are generated, this is a problem - so a smaller memtable log can be persisted to disk for recovery. This approach is called a *log-structured merge tree*, or an *LSM-Tree*. LSM-Trees are often coupled with *Bloom filters*, which approximate the contents of a set, to determine if elements *do not exist* in the set, in order to save reads.

*B-Trees* are a different indexing strategy. They use keep key-value pairs sorted by keys. However, they do not use multi-megabyte segments - instead, they use small, fixed-size *blocks*, or *pages*. Pages contain pointers to other pages, which contain pointers to other pages, etc. - until a page contains actual values. For example: if you are looking up key 561, you look at the page containing 100, 200, 300, 400, 500, 600, etc. The 500 key has a reference to a page containing 500, 510, 520, 530, etc. The 560 key contains a reference pointing to the page containing the key 561, which is associated with some value. The number of "reference hops" is the "*branching number*". When a new key needs to be added, but the page does not have enough space, it splits into two half-full pages, thus it remains balanced. A B-tree with `n` keys has `O(log(n))` depth. In this way, keys are overwritten - in contrast to the LSM-Tree approach. B-Trees may use a *write-ahead log*, which is append only, and written to before changes are applied to the tree itself, to aid in crash recovery.

LSM-Trees are faster for writes, and B-Trees are faster for reads. In-memory databases (such as MemCached) can be used for small datasets that can fit in memory - and tend to be faster because they do not write-to-disk prior to performing a write.

*Online transaction processing systems* (*OLTP*) are more for reads/writes forming a logical "transaction", whereas *online analytic processing systems* (*OLAP*) are designed for business intelligence-type queries. You don't want to run super long-running queries against an production OLTP database at-scale. B-Trees and LSM-Trees are great for OLTP systems, but poorer choices for OLAP systems. Data is generally ETL'ed into an OLAP system (or a *data warehouse*) in batches for analysis.

A *fact table* in an OLAP schema generally is one big master table with lots of foreign keys eminating outwards. For example: a *sales* table with foreign keys to `Date`, `Product`, `Customer`, `Store`, etc. This is called a *star schema*. If a star schema is more normalized, it can become a *snowflake schema*, where dimensions have more sub-dimensions. Star schemas are easiest to work with.

OLTP databases are *row-oriented* in the sense that row data is stored contiguously on-disk or in-memory. However, if you organize data by *column* instead, then you tend to see lots of patterns (for instance: `StoreId: 4`), which is well-suited for compression. *Bitmap encoding* is a good technique, wherein a sequential list of keys is broken down into binary sequences which demarcate per-key-occurrences, and then are further compressed using run-time compression to describe the length of 1- or 0-sequences. For example: 000011110001 becomes 4, 4, 3, 1.

OLAPs use *materialized views* a bit more than OLTP databases - materialized views cache aggregate data such as `COUNT()` and `SUM()`. A *data cube* is a materalized view that contains multiple axes representing different fact table columns, and may cohere into materialized data hypercubes (for instance).

OLTP is user-facing, and only small amounts of data are touched at a time. Records are requested by key. Indexes are used for retrieval. Disk seek time is the main bottleneck. In OLAP, there are less queries, but more demanding ones, and disk bandwidth is the main bottleneck. OLTP may use log-structured schemas (which are immutable and append-only) or update-in-place schemas (in which pages are fixed-size and B-trees are used for indexing).

# Chapter 4: Encoding and Evolution

# todo