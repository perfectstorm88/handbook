- [Mongodb官方文档学习笔记](#Mongodb%E5%AE%98%E6%96%B9%E6%96%87%E6%A1%A3%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0)
  - [MongoDB Limits and Thresholds mongodb的上限和下限](#MongoDB-Limits-and-Thresholds-mongodb%E7%9A%84%E4%B8%8A%E9%99%90%E5%92%8C%E4%B8%8B%E9%99%90)
  - [4.1 Data Modeling Introduction](#41-Data-Modeling-Introduction)
    - [4.1.1 Document Structure](#411-Document-Structure)
    - [4.1.2 Atomicity of Write Operations](#412-Atomicity-of-Write-Operations)
    - [4.1.3 Document Growth](#413-Document-Growth)
    - [4.1.4 Data Use and Performance](#414-Data-Use-and-Performance)
  - [4.2 Data Modeling Concepts](#42-Data-Modeling-Concepts)
    - [4.2.1 Data Model Design](#421-Data-Model-Design)
    - [4.2.2 Operational Factors and Data Models](#422-Operational-Factors-and-Data-Models)
    - [4.2.3 GridFS](#423-GridFS)
  - [4.3 Data Model Examples and Patterns](#43-Data-Model-Examples-and-Patterns)
    - [4.3.1 Model Relationships Between Documents](#431-Model-Relationships-Between-Documents)
    - [4.3.2 Model Tree Structures](#432-Model-Tree-Structures)
    - [4.3.3 Model Specific Application Contexts](#433-Model-Specific-Application-Contexts)
  - [4.4 Data Model Reference](#44-Data-Model-Reference)
    - [4.4.2 Database References](#442-Database-References)
  
# Mongodb官方文档学习笔记
## MongoDB Limits and Thresholds mongodb的上限和下限
[MongoDB Limits and Thresholds](https://docs.mongodb.com/manual/reference/limits)

- Indexes
  - 每个collection的最大索引数 **64个**
- 最多的collections数
  - WiredTiger引擎没有限制
  - 对于MMAPv1引擎，= f(the size of the namespace file，the number of indexes of collections in the database)
    - 名称空间的数量限制为名称空间的文件大小/628
    - 16M命名空间文件，可以支持约24,000的空间(16*1024*1024/628~~26715)
    - 每个collection和索引都是namespace
  

## 4.1 Data Modeling Introduction
The key challenge in data modeling is balancing the needs of the application, the performance characteristics of the database engine, and the data retrieval patterns. When designing data models, always consider the application usage of the data (i.e. queries, updates, and processing of the data) as well as the inherent structure of the data itself.
数据模型的调整，在于均衡：
- 业务需求
- 数据引擎的性能
- 数据获取模式
要考虑两点
- 数据的业务使用
- 数据本身的内在需求
### 4.1.1 Document Structure
The key decision in designing data models for MongoDB applications revolves around the structure of documents and how the application represents relationships between data. There are two tools that allow applications to represent these relationships: references and embedded documents.
应用如何表达数据关系，提供了两个工具：引用和嵌入对象

normalized data models/denormalized

### 4.1.2 Atomicity of Write Operations
In MongoDB, write operations are atomic at the document level, and no single write operation can atomically affect more than one document or more than one collection. 
写操作的原子性是文档级别
The Atomicity Considerations (page 152) documentation describes the challenge of designing a schema that balances flexibility and atomicity.
如何平衡灵活性和原子性
### 4.1.3 Document Growth
Some updates, such as pushing elements to an array or adding new fields, increase a document’s size.
添加数组元素或者增加新字段，会增加文档大小

For the MMAPv1 storage engine, if the document size exceeds the allocated space for that document, MongoDB relocates the document on disk.
超过大小，会数据搬迁
See Document Growth Considerations (page 152) for more about planning for and managing document growth for MMAPv1.

### 4.1.4 Data Use and Performance
See Operational Factors and Data Models (page 152) for more information on these and other operational considera- tions that affect data model designs.

## 4.2 Data Modeling Concepts
### 4.2.1 Data Model Design
In general, use embedded data models when:
- you have “contains” relationships between entities.
- you have one-to-many relationships between entities. In these relationships the “many” or child documents always appear with or are viewed in the context of the “one” or parent documents. 
自包含的实体关系
一对多的关系

In general, embedding provides better performance for read operations. Embedded data models make it possible to update related data in a single atomic write operation.
However, embedding related data in documents may lead to situations where documents grow after creation. With the MMAPv1 storage engine, document growth can impact write performance and lead to data fragmentation.
嵌入方式提供更好地读写性能。
但是嵌入方式会导致 grow after creation。会影响写性能，导致数据碎片

In version 3.0.0, MongoDB uses Power of 2 Sized Allocations (page 93) as the default allocation strategy for MMAPv1 in order to account for document growth, minimizing the likelihood of data fragmentation. 


In general, use normalized data models:
• when embedding would result in duplication of data but would not provide sufficient read performance advantages to outweigh the implications of the duplication.
• to represent more complex many-to-many relationships.
• to model large hierarchical data sets.
正常化的数据模型
- 嵌入方式会导致数据重复。或者读性能的优势并没有超过数据重复的影响
- 表示更复杂的多对多关系
- 高层次的数据集建模

References provides more flexibility than embedding. However, client-side applications must issue follow-up queries to resolve the references. In other words, normalized data models can require more round trips to the server.
应用比嵌入式更灵活，但是客户测会发起更多查询处理引用

### 4.2.2 Operational Factors and Data Models
These factors are operational or address requirements that arise outside of the application but impact the performance of MongoDB based applications. When developing a data model, analyze all of your application’s read operations (page 62) and write operations (page 75) in conjunction with the following considerations.
出现在应用之外的操作性或者地址需求，会影响到MongoDB的。写操作、读操作和下面因素：
Document Growth
You may also use a pre-allocation strategy to explicitly avoid document growth
可以使用预分配测试避免增长

For each database, a single namespace file (i.e. <database>.ns) stores all meta-data for that database, and each index and collection has its own entry in the namespace file.MongoDB places limits on the size of namespace files.
每个数据库有个命名空间文件，存储元数据。这个文件有大小限制

MongoDB using the mmapv1 storage engine has limits on the number of namespaces. You may wish to know the current number of namespaces in order to determine how many additional namespaces the database can support. To get the current number of namespaces, run the following in the mongo shell:
     db.system.namespaces.count()
The limit on the number of namespaces depend on the <database>.ns size. The namespace file defaults to 16 MB.
限制namespaces。默认16MB

Data Lifecycle Managemen-- Time to Live or TTL feature (pag

### 4.2.3 GridFS
GridFS is a specification for storing and retrieving files that exceed the BSON-document size limit of 16MB.
超过16MB的限制
把文件分为许多块，每块255kb，使用两个集合存储文件，一个是文件块，一个是文件元数据。

## 4.3 Data Model Examples and Patterns
### 4.3.1 Model Relationships Between Documents
Model One-to-One Relationships with Embedded Documents
Model One-to-Many Relationships with Embedded Documents
上面两个例子是 赞助人和多个地址的关系
Model One-to-Many Relationships with Document References
书和出版人的关系

一个例外：Otherwise, if the number of books per publisher is unbounded, this data model would lead to mutable, growing arrays, as in the following example:
如果出版人有很多书，可以反向表达：书上有个字段表示出版人

### 4.3.2 Model Tree Structures
Model Tree Structures with Parent References,,子引用父节点
The Parent Links pattern provides a simple solution to tree storage but requires multiple queries to retrieve subtrees.
简单，但是要求多次查询，去获得子树
Model Tree Structures with Child References  父节点引用子节点
The Child References pattern provides a suitable solution to tree storage as long as no operations on subtrees are necessary. This pattern may also provide a suitable solution for storing graphs where a node may have multiple parents.
也可以存储图关系


Model Tree Structures with an Array of Ancestors 存储祖先数组
可以快速找到后代和祖先。
Model Tree Structures with Materialized Paths 路径作为一个String;需要额外步骤，正则表达式。比上面的数组方式性能要高。

Model Tree Structures with Nested Sets  存储left和right
方便找子树，但是修改低效。适合静态树。
### 4.3.3 Model Specific Application Contexts
Model Data to Support Keyword Search 通过数组存储keyword，建立索引。
An array with a large number of elements, such as one with several hundreds or thousands of keywords will incur greater indexing costs on insertion.
如果数组有成百上千个，将严重影响索引的插入性能

Limitations of Keyword Indexes
Stemming. Keyword queries in MongoDB can not parse keywords for root or related words.
• Synonyms. Keyword-based search features must provide support for synonym or related queries in the applica-
tion layer.
• Ranking. The keyword look ups described in this document do not provide a way to weight results.
• Asynchronous Indexing. MongoDB builds indexes synchronously, which means that the indexes used for key- word indexes are always current and can operate in real-time. However, asynchronous bulk indexes may be more efficient for some kinds of content and workloads.
•词干。 MongoDB中的关键字查询无法解析关键字根或相关的词。
•同义词。基于关键字的搜索功能，必须提供支持同义词或相关查询的应用化层。
•排名。本文档中描述的关键词查询不提供途径来加权的结果。
•异步索引。 实时创建。然而异步大量索引更高效

**Model Monetary Data**
MongoDB stores numeric data as either IEEE 754 standard 64-bit floating point numbers or as 32-bit or 64-bit signed integers. Applications that handle monetary data often require capturing fractional units of currency. However, arith- metic on floating point numbers, as implemented in modern hardware, often does not conform to requirements for monetary arithmetic. In addition, some fractional numeric quantities, such as one third and one tenth, have no exact representation in binary floating point numbers.

**Model Time Data**


## 4.4 Data Model Reference
Most MongoDB driver clients will include the _id field and generate an ObjectId before sending theinsert operation to MongoDB; however, if the client sends a document without an _id field, the mongod will add the _id
field and generate the ObjectId.
有些client会在插入mongodb前创建_id字段
### 4.4.2 Database References

 Manual references
The only limitation of manual linking is that these references do not convey the database and collection names. 
手动索引，不能应用数据库和几何名称

 The drivers do not auto-matically resolve DBRefs into documents.
 Unless you have a compelling reason to use DBRefs, use manual references instead.






