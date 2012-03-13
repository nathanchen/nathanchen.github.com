---
layout: post
title: "MongoDB The Definitive Guide(4)"
category: Study Notes 
tags: ["MongoDB", "读书"]
---
{% include JB/setup %}

## Indexing

Database indexes are similar to a book's index: instead of looking through the whole book, the database takes a shortcut and just looks in the index, allowing it to do queries orders of magnitude faster. Once it finds tne entry in the index, it can jump right to the location of the desired document.

### Introduction to Indexing

You would create an index on "username". To create the index, use the ensureIndex method:
	
	> db.people.ensureIndex({"username" : 1})

An index needs to be created only once for a collection. If you try to create the same index again, nothing will happen.

An index on a key will make queries on that key fast. However, it may not make other queries fast, even if they contain the indexed key.

	> db.people.find({"date" : date1}).sort({"date" : 1, "username" : 1})
	---this query will not be affected by previous index

As a rule of thumb, you should create an index that contains all of the keys in your query.

The document you pass to ensureIndex is of the form like:
	a set of keys with value 1 or -1, depending on the direction you want the index to go. If you have only a single key in the index, direction is irrelevant.

If you have more than one key, you need to start thinking about index direction.
	
	---We index the data with {"username" : 1, "age" : -1}
	The usernames are in strictly increasing alphabetical order, and within each name group the ages are in decreasing order.

The MongoDB query optimizer will reorder query terms to take advantage of indexes: if you query for {"x" : "foo", "y" : "bar"} and you have an index on {"y" : 1, "x" : 1}, MongoDB will figure it out.

**The disadvantage to creating an index is that it puts a little bit of overhead on every insert, update, and remove.** This is because the database not only needs to do the operation but also needs to make a note of it in any indexes on the collection. Thus, the absolute minimum number of indexes should  be created. There is a built-in maximum of 64 indexes per collection, which is more than almost any application should need.

**Do not index every key. This will make insertion slow, take up lots of space, and probably not speed up your quries very much.**

**Sometimes the most efficient solution is actually not to use an index.** If a query is return a half or more of the collection, it will be more efficient for the database to juest do a table scan. For queries such as checking whether a key exists or determining whether a boolean value is true or false, it may actually be better to not use an index at all.

### Scaling Indexes

For example:
Suppose we have a collection of status messages from users. We want to query by user and date to pull up all of a user's recent statuses. Using what we've learned so far, we might create an index that looks like the following:
	
	> db.status.ensureIndex({user : 1, date : -1})
	This will make the query for user and data efficient, but it is not actually the best index choice.

Image if the application has million of users who have dozens of status updates per day. If the index entries for each user's status messages take up a page's worth of space on disk, then for every "latest statuses" query, the database will have to load a different page into memory. This will be very slow if the site becomes popular enough that not all of the index fits into memory.

If we flip the index order to {date : -1, user : 1}, the database can keep the last couple days of the index in memory, swap less, and thus query for the latest statuses for any user much more quickly.

**There are several questions to keep in mind when deciding what indexes to create:**
* What are the queries you are doing? Some of these keys will need to be indexed.
* What is the correct direction for each key?
* How is this going to scale? Is there a different ordering of keys that would keep more of the frequently used portions of the index in memory?

### Indexing Keys in Embedded Documents

Indexes can be created on keys in embedded documents in the same way that they are created on normal keys.
	
	---If we want to be able to search blog post comments by date
	> db.blog.ensureIndex({"comments.date" : 1})

### Indexing for Sorts

If you can sort on a key that is not indexed, MongoDB needs to pull all of that data into memory to sort it. Thus, there's a limit on the amount you can sort without an index. Once your collection is too big to sort in memory, MongoDB will just return an error for queries that try.

Indexing the sort allows MongoDB to pull the sorted data in order, allowing you to sort any amount of data without running out of memory.

### Uniquely Identifying Indexes

Each index in a collection has a string name that uniquely identifies the index and is used by the server to delete or manipulate it. 

You can specify your own name as one of the options to ensureIndex:
	
	> db.foo.ensureIndex({"a" : 1, "b" : 1, "c" : 1, ..., "z" : 1}, {"name" : "alphabet"})
There is limit to the number of characters in an index name, so complex indexes may need custom names to be created. A call to getLastError will show whether the index creation succeeded or why it didn't.

## Unique Indexes

Unique indexes guarantee that, for a given key, every document in the collection will have a unique value. **If you want to make sure no two documents can have the same value in the "username" key, you can create a unique index:**
	
	> db.people.ensureIndex({"username" : 1}, {"unique" : true})
Keep in mind that insert, by default, does not check whether the document was actually inserted. You may want to use safe inserts if you are inserting documents that might have a duplicate value for an unique key.

### Dropping Duplicates

When building uunique indexes for an existing collection, some values may be duplicates. If there are any duplicates, this wil cause the index building to fail. The dropDups option will save the first document found and remove any subsequent documents with duplicate values:

	> db.people.ensureIndex({"username" : 1}, {"unique" : true, "dropDups" : true})
**However, it might be better to create a script to preprocess the data if it is important.**

### Compound Unique Indexes

You can also create a compound unique index. If you do so, individual keys can have the same values, but the combination of values for all keys must be unique.

	{files_id : ObjectId("4b23c3ca7525f35f94b60a2d"), n : 1} {files_id : ObjectId("4b23c3ca7525f35f94b60a2d"), n : 2} {files_id : ObjectId("4b23c3ca7525f35f94b60a2d"), n : 3} {files_id : ObjectId("4b23c3ca7525f35f94b60a2d"), n : 4}

## Using Explain and Hint

explain will give you lots of information about your queries.

	> db.foo.find().explain()
	---explain will return information about the indexes used for the query and stats about timing and the number of documents scanned.

For example:

**Example 1:**

	> db.people.find().explain()
	{
		"cursor" : "BasicCursor", 
		"indexBounds" : [ ], 
		"nscanned" : 64, 
		"nscannedObjects" : 64, 
		"n" : 64,
		"millis" : 0,
		"allPlans" : [
			{
				"cursor" : "BasicCursor",
				"indexBounds" : [ ] 
			}
		] 
	}
**Important rules**

"cursor" : "BasicCursor"
	This means that the query did not use an index
"nscanned" : 64
	This is the number of documents that the database looked through
"n" : 64
	This is the number of documents returned.
"millis" : 0
	The number of milliseconds it took the database to execute the query. 0 is a good time to shoot for.

**Example 2**
	
	> db.c.find({age : {$gt : 20, $lt : 30}}).explain()
	{
		"cursor" : "BtreeCursor age_1", 
		"indexBounds" : [
			[
				{
					"age" : 20 
				},
				{
					"age" : 30 
				}
			] 
		],
		"nscanned" : 14, 
		"nscannedObjects" : 12, 
		"n" : 12,
		"millis" : 1, 
		"allPlans" : [
			{
				"cursor" : "BtreeCursor age_1", 
				"indexBounds" : [
					[
						{
							"age" : 20 
						},
						{
							"age" : 30 
						}
					] 
				]
			} 
		]
	}
**Important rules**

"cursor" : "BtreeCursor age_1"
	Indexes are stored in data structures called B-trees, when a query uses an index, it uses a spacial type of cursor called a BtreeCursor. This value also identifies the name of the index being used: age_1

"allPlans" : [ ... ]
	This key lists all of the plans MongoDB considered for the query.
	If we had multiple overlapping indexes and a more complex query, "allPlans" would contain information about all of the possible plans that could have been used.

**When we are querying for an exact match on "username" and a range of values for "age", so the database chooses to use the {"username" : 1, "age" : 1} index, reversing the term of the query.**

**If, on the other hand, we query for an exact age and a range of name, MongoDB will use the other index.**

You can force it to use a certain index by using hint. 

If you want to make sure MongoDB uses the {"username" : 1, "age" : 1} index on the previous query, you could say the following:

	> db.c.find({"age" : 14, "username" : /.*/}).hint({"username" : 1, "age" : 1})

**Hinting is usually unnecessary.** MongoDB has a query optimizer and is very clever about choosing which index to use. When you first do a query, the query optimizer tries out a number of query plans concurrently. The first one to finish will be used, and the rest of the query executions are terminated. That query plan will be remembered for future queries on the same keys. The query optimizer periodically retries other plans, in case you've added new data and the previously chosen plan is no longer best.

## Index Administration

1. Metainformation about indexes is stored in the system.indexes collection of each database. This is a reserved collection, so you cannot insert documents into it or remove documents from it. You can manipulate its documents only through ensureIndex and the dropIndexes database command.

2. Collection names are limited to 121 bytes in length.

3. It is important to remember that the size of the collection name plus the index name.

### Changing Indexes

You can add new indexes to existing collections at any time with ensureIndex:
	
	> db.people.ensureIndex({"username" : 1}, {"background" : true})
Building indexes is time-consuming and resource-intensive. Using the {"background" : true} option builds the index in the background, while handling incoming requests. If you do not include the background option, the database will block all other requests while the index is being built.

Even building an index in the background can take a toll on normal operations, so it is best to do it during an "off" time; building indexes in the background will still add extra load to your database, but it will not bring it grinding to a halt.

Creating indexes on existing documents is slightly faster than creating the index first and then inserting all of the documents.

If an index is no longer necessary, you can remove it with the dropIndexes command and the index name.

	> db.runCommand({"dropIndexes" : "foo", "index" : "alphabet"})
You can drop all indexes on a collection by passing in * as the value for the index key:

	> db.runCommand({"dropIndexes" : "foo", "index" : "*"})


## Geospatial Indexing

There is another type of query that is becoming increasingly common (especially with the emergence of mobile device): finding the nearest N things to a current location. MongoDB provides a special tupe of index for coordinate plane queries, called a geospatial index.

A geospatial index can be created using the ensureIndex function, but by passing "2d" as a value instead of 1 or -1:
	
	> db.map.ensureIndex({"gps" : "2d"})
The "gps" key must be some type of pair value, that is, a two-element array or embedded document with two keys.

	{ "gps" : [ 0, 100 ] }
	{ "gps" : { "x" : -30, "y" : 30 } }
	{ "gps" : { "latitude" : -180, "longitude" : 180 } }
The keys can be anything; for example,
	
	{"gps" : {"foo" : 0, "bar" : 1}} is valid

By default, geospatial indexing assumes that your values are going to range from -180 to 180. You can specify what the minimum and maximum values will be as options to ensureIndex:

	> db.star.trek.ensureIndex({"light-years" : "2d"}, {"min" : -1000, "max" : 1000})

Geospatial queries can be performed in two ways: as a normal query (suing find) or as a database command

	> db.map.find({"gps" : {"$near" : [40, -73]}})
This finds all of the documents in the map collection, in order by distance from the point (40, -73). A default limit of 100 documents is applied if no limit is specified. If you don't need that many results,

	> db.map.find({"gps" : {"$near" : [40, -73]}}).limit(10)

### Compound Geospatial Indexes

For example, a user might want to find all coffee shops or pizza parlors near where they are. To facilitate thi type of query, you can combine geospatial indexes with normal indexes.

	> db.map.find({"location" : {"$near" : [-70, 30]}, "desc" : "coffeeshop"}).limit(1)
	{
		"_id" : ObjectId("4c0d1348928a815a720a0000"),
		"name" : "Mud",
		"location" : [x, y],
		"desc" : ["coffee", "coffeeshop", "muffins", "espresso"]
	}


##REFERENCE
K. Chodorow, M. Dirolf, "MongoDB: The Definitive Guide", O'Reilly, 2010, US


