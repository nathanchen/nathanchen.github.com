---
layout: post
title: "Database Systems(1)"
category: 
tags: []
---
{% include JB/setup %}

- Database: Collection of data central to some enterprise/ organisation; essential to operation of enterprise; an asset in its own right: historial data can guide enterprise strtegy, of interest to other enterprises; database is persistent; all qualfied users have access to the same data for use in a variety of activities.

- A database management system is a program that manages a database: stores the database on some mass storage providing fail safety(backup/ recovery); supports a high-level access language(SQL); provdes transaction management to guarantee correct concurrent access to shared data.

- Disadvantages of File processing:
	1) Program-data dependence: all programs contain full descriptions of each data file they use
	2) Data Redundancy: different systems/ programs have separate copies of the same data; no centralized control of data
	3) Limited data sharing: required data in several, potentially incompatible files.
	4) Excessive progrmam maintenance

- Advantages of Database:
	1) Program-data independence: metadata stored in DBMS, so applications dont need to worry about data formats; results in reduced application development time, increased maintenance productivity and efficient access.
	2) minimal data redundancy: leads to increased data integrity/ consistency
	3) Improved Data Sharing: different users get different views of the data; efficient concurrent access
	4) Enforcement of standards: all data access is done in the same way
	5) Improved data quality: integrity constraints, data validation rules
	6) Better data accessibility/ responsiveness: use of standard data query language(SQL)
	7) security, backup/recovery, concurrency

- Relational Databases

	- Stores data as rows with multiple attributes
	- Rows of the same format('type') form a table
	- A relational database is a collection of such tables (which typically are related to each other by key attributes)

- What is a transaction: When an event in the real world changes the sate of the enterprise, a transaction is executed to cause the corresponding change in the database state.

- DB system requirements:
	1) High Availability: on line -> must be operational while enterprise is functioning
	2) High Reliability: correctly tracks state, does not loose data, controlled concurrency
	3) High Throughput: many users -> many transactions per second
	4) Low Response Time: on line -> users are waiting
	5) Long lifetime: complex systems are not easily replaced
	6) Security: sensitive information must be carefully protected since system is accessible to many users.

- Data model: a collection of concepts for describing data
 
- Schema: a description of a particular collection of data at some abstraction level, using a given data model.
 
- Relational data model:
    relation, basically a table with rows and columns
    schema, which describes the columns or fields
 
- Relation is a set of tuples (is a named, two-dimensional table of data)
    1) tuple ordering immaterial
    2) No duplicates
    3) Cardinality of relation = number of tuples per entity
 
- All tuples in a relation have the same structure; constructed from the same set of attributes.
    1) Arity of relation = number of attributes
    2) attributes are named (ordering is immaterial)
 
- Not all tables qualify as a relation
    1) Every relation must have a unique name
    2) Attributes in tables must have unique names
    3) Every attribute value is atomic. The range of allowed values of an attribute is called its domain
    4) The order of the columns is irrellevant
    5) Every row is unique (cant have two rows with exactly the same values for all their fields)
    6) The order of the rows is irrelevant
 
- A relation R(表格中具体的数据) has a relation schema（表格的输入格式）, which specifies name of relation, and name and data type of each attribute
 
- Relation schema:
     Students(sid: string, name: string, login: string, addr: string, gender: char)
 
- Relation: 
        R = { (Jones, Main, Sydney),  
             (Smith, North, Perth), 
             (Curry, North, Brisbane), 
             (Lindsay, Park, Sydney) } 
 
- A relation instance: a set of tuples(table) for a schema
    #rows = cardinality
    #fields = degree (or arity) of a relation
 
- First Normal Form (1NF) is the restriction of atomic attributes
 
- Relational database management system(RDBMS) allows duplicate rows
    In a SQL-based RDBMS, it is possible to insert a row where every attribute has the same value as an existing row. The table will then contain two identical rows. 
 
- RDBMS allows a special "null" value in a column to represent facts that are not relevant, or not yet known.
 
- It is very dangerous to instead use an ordinary value with special meaning: Eg if salary = -1 is used for "unknown", then averages wont be sensible
 
- keys are special attributes that serve two main purposes:
    1) Primary keys are unique, minimal identifiers of the relation in question. 
    2) Foreign keys are identifiers that enable a dependent relation (on the many side of a relationship) to refer to its parent relation (on the one side of the relationship)
 
- Keys can be simple (a single attribute) or composite (more than one attribute)
 
- Keys usually are used as indices to speed up the response to user queries
 
- Possibly many candidate keys (specified using UNIQUE), one of which is chosen as the primary key
 
- Foreign key: set of attributes in a relation that is used to 'refer' tp a tuple in a parent relation
 
- Referential Integrity: for each tuple in the referring relation whose foreign key value is a, there must be a tuple in the referred relation whose primary key value is also a.
 
- CREATE TABLE Enrolled ( sid INTEGER, ucode CHAR(8), semester VARCHAR, 
CONSTRAINT Enrolled_FK1 FOREIGN KEY (sid) REFERENCES Student, 
CONSTRAINT Enrolled_FK2 FOREIGN KEY (ucode) REFERENCES UoS, 
CONSTRAINT Enrolled_PK  PRIMARY KEY (sid,ucode)  
);
 
- Data Structure: A relational database is a set of relations (tables) with tuples (rows) and fields (columns) - a simple and consistent structure
 
- The union of the corresponding relational schemata is the relational database schema
 
- Data manipulation: powerful operators to manipulate the data stored in relations
 
- Data Integrity: facilities to specify a variety of rules to maintain the integrity of data when it is manipulated.
 
- Integrity Constraint (IC): condition that must be true for any instance of the database.
    ICs can be declared in the schema. They are specified when schema is defined. All declared ICs are checked whenever relations are modified.
 
- A legal instance of a relation is one that satisfies all specified ICs. If ICs are declared, DBMS will not allow illegal instances.
 
- Logical schema: 
! Student(sid:integer, name:string, login:string, bdate: date, sex:char) 
! UoS(ucode:string, cname:string, credits: integer) 
! Enrolled(sid:integer, ucode :string, semester: integer, grade:string) 
 
- Physical Schema: 
! describes details of how data is stored:  
!  E.g. relations stored as unordered files 
!  Index on first column of Students 
! non-database applications typically have only on a physical schema
 
- External Schema (view) 
! UoS_info(ucode :string, enrolment: integer) 
 
- A view is a virtual relation, but we store a definition, rather than a set of tuples. A mechanism to hide data from the view of certain users.Views can be used to present necessary information, while hiding details in underlying relations. 
 
- Entity: a person, place, object, event, or concept about which you want to gather and store data
 
- Entity type (also: entity set): a collection of entities that share common properties or characteristics, which is described by a set of attributes
 
- Attribute: describe one aspect of an entity type
 
- domain: possible values of an attribute
 
- key: minimal set of attributes that uniquely identifies an entity in the set
 
- Relationship: relates two or more entities
 
- number of entities is also known as the degree of the relationship
 
- relation (relational model): set of tuples
    relationship (E-R model): describes relationship between entities
    Both entity sets and relationship sets (E-R model) may be represented as relations (in the relational model)
 
- Weak entity type: an entity type that does not have a primary key.
    1) can be seen as an exclusive 'part-of' relationship
    2) its existence depends on the existence of one or more identifying entity types
    3) it must relate to the identifying entity set vaia a total, one-to-many identifying relationship type from the identifying to the weak entity set
    Eg: child from parents, payment of a loan
 
- The discriminator (or partial key) of a weak entity type is the set of attributes that distinguishes among all the entities of a weak entity type related to the same owning entity
 
- The most important requirement is adequacy: that the design should allow representing all the important facts about the design
 
- If a design is adequate, then we seek to avoid redundancy in the data
 
- Redundant data is where the same information is repeated in several places in the db
 
- Redundancy is at the root of several problems associated with relational schemas:
    1) Insertion Anomaly: Adding new rows forces user to create duplicate data or to use null value
    2) Deletion Anomaly: Deleting rows may causes a loss of data that would be needed for other future rows.
    3) Update Anomaly: Changing data in a row forces changes to other rows because of duplication
 
- To say a table relation, it should contain unique rows and no multivalued attributes.
 
- Functional Dependency: the value of one attribute determines the value of another attribute
 
- FDs must be identified based on the semantics of an application
 
- A relation R is in First Normal Form(1NF) if the domains of all attributes of R are atomic （所有元素都是不能拆分的）
 
- Domain is atomic if its elements are considered to be indivisible units; Examples of non-atomic domains: multivalued attributes, composite attributes
 
- Second Normal Form (2NF): 1NF + every non-key attribute is fully functionally dependent on the primary key, this means no partial dependencies, no FD x -> y where x is a strict subset of some key（1NF + 所有非键元素都可以通过主键查找到）
 
- Third Normal Form (3NF)：2NF + no transitive dependencies (functional dependencies between non-primary-key attributes). No FD of the form x -> y, where x is not a key and y is not at least a subset of a key
 
- 在任何一个关系数据库中，第一范式（1NF）是对关系模式的基本要求，不满足第一范式（1NF）的数据库就不是关系数据库。在第一范式（1NF）中表的每一行只包含一个实例的信息。

- 第二范式（2NF）要求数据库表中的每个实例或行必须可以被唯一地区分。所谓完全依赖是指不能存在仅依赖主关键字一部分的属性。第二范式就是非主属性非部分依赖于主关键字。
 
- 第三范式（3NF）要求一个数据库表中不包含已在其它表中已包含的非主关键字信息。