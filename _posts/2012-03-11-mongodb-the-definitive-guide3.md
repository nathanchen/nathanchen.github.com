---
layout: post
title: "Mongodb The Definitive Guide(3)"
category: Study Notes 
tags: ["MongoDB", "读书"]
---
{% include JB/setup %}

## Chapter 4 Querying

### Introduction to find

1. The find method is used to perform queries in MongoDB. Querying returns a subset of documents in a collection, from no documents at all to the entire collection.

2. If find isn't given a query document, it default to {}
		
		> db.c.find()
		---returns everything in the collection c.

### Specifying Which Keys to Return

1. Sometimes, you do not need all of the key/ value pairs in a document returned. If this is the case, you can pass a second argument to find (or findOne) specifying the keys you want.
	
		> db.users.find({}, {"username" : 1, "email" : 1})
		{
			"_id" : ObjectId("4ba0f0dfd22aa494fd523620"), 
			"username" : "joe",
			"email" : "joe@example.com"
		}

		---You can also use this second parameter to exclude specific key/ value pairs from the results of a query.
		> db.users.find({}, {"username" : 1, "_id" : 0})
		{
		"username" : "joe",
		}

### Limitations

1. The value of a query document must be a constant as far as the database is concerned. 
		
		> db.stock.find({"in_stock" : "this.num_sold"}) // doesn't work

**Restructing your document slightly will give you a better performance.**

### Query Conditionals

		> db.users.find({"age" : {"$gte" : 18, "$lte" : 30}})

---For example, to find people who registered before January 1, 2007
		
		> start = new Date("01/01/2007")
		> db.users.find({"registered" : {"$lt" : start}})

**An exact match on a date is less useful, because dates are only stored with millisecond precision. Often you want a whole day, week, or month, making a range query necessary.

		> db.users.find({"username" : {"$ne" : "joe"}})
		---"$ne" can be used with any type

### OR Queries

		> db.raffle.find({"ticket_no" : {"$in" : [725, 542, 390]}})

If "$in" is given an array with a single value, it behaves the same as directly matching the value.

		{ticket_no : {$in} : [725]}} matches the same documents as {ticket_no : 725}

The opposite of "$in" is "$nin", which returns documents that don't match any of the criteria in the array.

		> db.raffle.find({"ticket_no" : {"$nin" : [725, 542, 390]}})

If we need to find documents where "ticket_no" is 725 or "winner" is true. We'll need to use the "$or" condition.

		> db.raffle.find({"$or" : [{"ticket_no" : 725}, {"winner" : true}]})

"$or" can contain other conditions. If we want to match any of the three "ticker_no" value or the "winner" key,
		
		> db.raffle.find({"$or" : [{"ticket_no" : {"$in" : [725, 542, 390]}}, {"winner" : true}]})

### $not
1. "$not" is a metaconditional: it can be applied on top of any other criteria.

		> db.users.find({"id_num" : {"$not" : {"$mod" : [5, 1]}}})

### Rules for Conditionals
1. **Conditionals are an inner document key, and modifiers are always a key in the outer document.** In the query, "$lt" is in the inner document; in the update, "$inc" is the key for the outer document.

2. Multiple update modifiers cannot be used on a single key. You cannot have a modifier document such as {"$inc" : {"age" : 1}, "$set" : {age : 40}} because it modifies "age" twice. With query conditionals, no such rule appliers.

### null
		
		> db.c.find()
		{ "_id" : ObjectId("4ba0f0dfd22aa494fd523621"), "y" : null }
		{ "_id" : ObjectId("4ba0f0dfd22aa494fd523622"), "y" : 1 } 
		{ "_id" : ObjectId("4ba0f148d22aa494fd523623"), "y" : 2 }
We can query for documentss whose "y" key is null

		> db.c.find({"y" : null})
		{ "_id" : ObjectId("4ba0f0dfd22aa494fd523621"), "y" : null }

However, null not only matches itself but also matches "does not exist"

		> db.c.find({"z" : null})
		{ "_id" : ObjectId("4ba0f0dfd22aa494fd523621"), "y" : null } 
		{ "_id" : ObjectId("4ba0f0dfd22aa494fd523622"), "y" : 1 }
		{ "_id" : ObjectId("4ba0f148d22aa494fd523623"), "y" : 2 }

Unfortunately, there is no "$eq" operator, but "$in" with one element is equivalent.
		
		---If we only want to find keys whose value is null, we can check that the key is null and exists using the "$exists" conditional:
		> db.c.find({"z" : {"$in" : [null], "$exists" : true}})

### Regular Expressions
		
		---if we want to find all users with the name Joe or joe.
		> db.users.find({"name" : /joe/i})

		---if we want to match not only various capitalizations of joe
		> db.users.find({"name" : /joey?/i})

### Querying Arrays
---Querying for elements of an array is simple

		> db.food.insert({"fruit" : ["apple", "banana", "peach"]})

		> db.food.find({"fruit" : "banana"})

1. $all
If you need to match arrays by more than one element, you can use "$all". This allows you to match a list of elements.

		---original
		> db.food.insert({"_id" : 1, "fruit" : ["apple", "banana", "peach"]})
		> db.food.insert({"_id" : 2, "fruit" : ["apple", "kumquat", "orange"]}) 
		> db.food.insert({"_id" : 3, "fruit" : ["cherry", "banana", "apple"]})

Then we can find all documents with both "apple" and "banana" elements by querying with "$all"
		
		> db.food.find({fruit : {$all : ["apple", "banana"]}}) 
		{"_id" : 1, "fruit" : ["apple", "banana", "peach"]} 
		{"_id" : 3, "fruit" : ["cherry", "banana", "apple"]}

If you want to query for a specific element of an array, you can specify an index using the syntax key.index:

		> db.food.find({"fruit.2" : "peach"})
		---Arrays are always 0-indexed, so this would match the third array element against the sting "peach"

2. $size

		> db.food.find({"fruit" : {"$size" : 3}})

"$size" cannot be combined with another $ conditional, but this query can be accomplished by adding a "size" key to the document

3. The $slice operator

Suppose we had a blog post document and we wanted to return the first 10 comments:

		> db.blog.posts.findOne(criteria, {"comments" : {"$slice" : 10}})
If we wanted the last 10 comments, we could use -10:

		> db.blog.posts.findOne(criteria, {"comments" : {"$slice" : -10}})
"$slice" can also return pages in the middle of the results by taking an offset and the number of elements to return:

		> db.blog.posts.findOne(criteria, {"comments" : {"$slice" : [23, 10]}})

### Querying on Embeded Documents
1. There are two ways of querying for an embedded document: querying for the whole document or querying for its individual key/ value pairs.

2. Querying for an entire embedded document works identically to a normal query.

		{
			"name" : {
				"first" : "Joe",
				"last" : "Schmoe" 
			},
			"age" : 45 
		}
We can query

		> db.people.find({"name" : {"first" : "Joe", "last" : "Schmoe"}})
If Joe decides to add a middle name key, suddenly this query won't work anymore; it doesn't match the entire embedded document. This type of query is also order- sensitive; {"last" : "Schmoe", "first" : "Joe"} would not be a match.

**Instead, you can query for embedded keys using dot-notation:
		
		> db.people.find({"name.first" : "Joe", "name.last" : "Schmoe"})
		---Now, if Joe adds more keys, this query will still match his first and last names.
**Embedded document matches have to match the whole document**

		---Origin
		> db.blog.find()
		{
			"content" : "...", 
			"comments" : [
				{
					"author" : "joe", 
					"score" : 3,
					"comment" : "nice post"
				}, 
				{
					"author" : "mary",
					"score" : 6,
					"comment" : "terrible post"
				}
			]
		}

		---This will not work
		> db.blog.find({"comments.author" : "joe", "comments.score" : {"$gte" : 5}})

		---It would match "author" : "joe" in the first comment and "score" : 6 in the second comment

To correctly group criteria without needing to specify every key, use "$elemMatch". 
		
		---"$elemMatch" allows us to "group" our criteria.
		> db.blog.find({"comments" : {"$elemMatch" : {"author" : "joe", "score" : {"$gte" : 5}}}})

### Cursors

		> for(i=0; i<100; i++) {
		... db.c.insert({x : i});
		... }
		> var cursor = db.collection.find();
The advantage of doing this is that you can look at one result at a time

To iterate through the results, you can use the next method on the cursor. You can use hasNext to check whether there is another result. A typlical loop through results looks like the following:
		
		> while (cursor.hasNext()) { 
		... obj = cursor.next();
		... // do stuff
		... }

		---cursor.hasNext() checks that the next result exists, and cursor.next() fetches it.

### Limits, Skips and Sorts
1. The most common query options are limiting the number of results returned, skipping a number of results, and sorting. 

		> db.c.find().limit(3)
2. skip works similarly to limit:
		
		> db.c.find().skip(3)
		---This will skip the first three matching documents and return the rest of the matches.
3. sort takes an object: a set of key/ value pairs where the keys are key names and the values are the sort directions. Sort direction can be 1 (ascending) or -1 (descending). If multiple keys are given, the results will be sorted in that order.

		> db.c.find().sort({username : 1, age : -1})
4. These three methods can be combined.

		> db.stock.find({"desc" : "mp3"}).limit(50).sort({"price" : -1})

### Avoiding Large Skips
1. Using skip for a small number of documents is fine. For a large number of results, skip can be slow and should be avoided.

2. The easiest way to do pagination is to return the first page of results using limit and then return each subsequent page as an offset from the beginning.

		> // do not use: slow for large skips
		> var page1 = db.foo.find(criteria).limit(100)
		> var page2 = db.foo.find(criteria).skip(100).limit(100) 
		> var page3 = db.foo.find(criteria).skip(200).limit(100) 
		...
3. You can usually find a way to paginate without skips.
For example, suppose we want to display documents in descending order based on "date"
		
		> var page1 = db.foo.find().sort({"date" : -1}).limit(100)
		---Then, we can use the "date" value of the last document as the criteria for fetching the next page:
		var latest = null;
		// display first page 
		while (page1.hasNext()) { 
			latest = page1.next();
			display(latest); 
		}

		// get next page
		var page2 = db.foo.find({"date" : {"$gt" : latest.date}}); 
		page2.sort({"date" : -1}).limit(100);

		**Now the query does not need to include a skip**

##REFERENCE
K. Chodorow, M. Dirolf, "MongoDB: The Definitive Guide", O'Reilly, 2010, US
