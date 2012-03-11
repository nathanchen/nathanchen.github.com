---
layout: post
title: "Mongodb The Definitive Guide(2)"
category: Study Notes 
tags: ["MongoDB", "读书"]
---
{% include JB/setup %}

## Chapter 3 Creating, Updating, and Deleting Documents

### Inserting and Saving Documents

To insert a document into a collection, use the collection's insert method:

		> db.foo.insert({"bar" : "baz"})
**Batch Insert**

1. If you have a situation where you are inserting multiple documents into a colection, you can make the insert faster by using batch inserts. Batch inserts allow you to pass an array of documents to the database.

2. A batch insert is a single TCP request, meaning that you do not incur the overhead of doing hundreds of individual requests. It can also cut insert time by eliminating a lot of the header processing that gets done for each messages.

3. If you are just importing raw data (for example, from a data feed or MySQL), there are command-line tools like mongoimport that can be used instead of batch insert. There is a limit to how much can be inserted in a single batch insert (16MB).

**Inserts: Internals and Implications**

1. You can start the database server with the --objcheck option, and it will examine each document's structural validity before inserting it (at the cost of slower performance).

2. MongoDB does not do any sort of code execution on insers, so they are not vulnerable to injection attacks. 

### Removing Documents
		
		---This will remove all of the documents in the users collection. This doesn't actually remove the collection, and any indexes created on it will still exist.
		> db.users.remove()

		The remove function optionally takes a query document as a parameter
		> db.mailing.list.remove({"opt-out" : true})

**Once data has been removed, it is gone forever. There is no way to undo the remove or recover deleted documents.**

**If you want to clear an entire collection, it is faster to drop it (and then re-create any indexes).**

		> db.drop_collection("bar")
		---But it comes at the expense of granularity: we cannot specify any criteria. The whole collection is dropped, and all of its indexes are deleted.

### Updating Documents

1. Update takes two parameters: a query document, which locates documents to update, and a modifier document, which describes the changes to make to the documents found.

2. Updates are atomic: if two updates happen at the same time, whichever one reaches the server first will be applied, and then the next on will be applied.
For example:

		---We want to make changes on this
		{
			"_id" : ObjectId("4b2b9f67a1f631733d917a7a"), 
			"name" : "joe",
			"friends" : 32,
			"enemies" : 2
		}

		---We want to change into the following:
		{
			"_id" : ObjectId("4b2b9f67a1f631733d917a7a"), 
			"username" : "joe",
			"relationships" :
			{
				"friends" : 32,
				"enemies" : 2 
			}
		}

		-- we can make this change by replacing the document using an update:
		> var joe = db.users.findOne({"name" : "joe"});
		> joe.relationships = {"friends" : joe.friends, "enemies" : joe.enemies};
		{
			"friends" : 32,
			"enemies" : 2 
		}
		> joe.username = joe.name; 
		"joe"
		> delete joe.friends;
		true
		> delete joe.enemies; 
		true
		> delete joe.name;
 		true
		> db.users.update({"name" : "joe"}, joe);

**A common mistake is matching more than one document with the criteria and then create a duplicate "_id" value with the second parameter. The database will throw an error for this, and nothing will be changed.**

**Using Modifiers**

1. Partial updates can be done extremely effieciently by using atomic update modifiers. 

2. We can use update modifier to do this increment atomically. Each URL and its number of page views is stored in a document that looks like this:

		{
			"_id" : ObjectId("4b253b067525f35f94b60a31"), 
			"url" : "www.example.com",
			"pageviews" : 52
		}

		> db.analytics.update({"url" : "www.example.com"},
		... {"$inc" : {"pageviews" : 1}})	
3. "$set" sets the value of a key. If the key does not yet exist, it will be created.
	
		---Original document
		> db.users.findOne()
		{
			"_id" : ObjectId("4b253b067525f35f94b60a31"), 
			"name" : "joe",
			"age" : 30,
			"sex" : "male",
			"location" : "Wisconsin"
		}

		> db.users.update({"_id" : ObjectId("4b253b067525f35f94b60a31")}, 
		... {"$set" : {"favorite book" : "war and peace"}})

		> db.users.findOne()
		{
			"_id" : ObjectId("4b253b067525f35f94b60a31"), 
			"name" : "joe",
			"age" : 30,
			"sex" : "male",
			"location" : "Wisconsin",
			"favorite book" : "war and peace"
		}

		---If the user decides that he actually enjoys a different book, "$set" can be used to change the value:

		> db.users.update({"name" : "joe"},
		... {"$set" : {"favorite book" : "green eggs and ham"}})

		---"$set" can even changes the type of the key it modifies
		> db.users.update({"name" : "joe"},
		... {"$set" : {"favorite book" :
		... ["cat's cradle", "foundation trilogy", "ender's game"]}})	

		---can remove the key altogether with "$unset":
		> db.users.update({"name" : "joe"},
		... {"$unset" : {"favorite book" : 1}})

		---Now the document will be the same as the original copy

		---You can also use "$set" to reach in and change embedded documents:
		> db.blog.posts.findOne()
		{
			"_id" : ObjectId("4b253b067525f35f94b60a31"), "title" : "A Blog Post",
			"content" : "...",
			"author" : {
				"name" : "joe",
				"email" : "joe@example.com" 
			}
		}

		> db.blog.posts.update({"author.name" : "joe"}, {"$set" : {"author.name" : "joe schmoe"}})

		> db.blog.posts.findOne() 
		{
			"_id" : ObjectId("4b253b067525f35f94b60a31"), "title" : "A Blog Post",
			"content" : "...",
			"author" : {
				"name" : "joe schmoe",
				"email" : "joe@example.com" 
			}
		}

		**You must always use a $ modifier for coding, changing, or removing keys.
		--A common error people often make when starting out is to try to set the value of "foo" to "bar" by doing an update that looks like this:
		> db.coll.update(criteria, {"foo" : "bar"})

		---It actually does a full-document replacement, replacing the matched document with {"foo" : "bar"}. Always use $ operators for modifying individual key/ value pairs.

		> db.blog.insert({"name" : "test", "ff" : "fee"})
		> db.blog.update({"name" : "test"}, {"ff" : "ww"})
		> db.blog.find()
		{ "_id" : ObjectId("4f5b0b6b9f9bdc3684bd3e51"), "ff" : "ww" }

4. Incrementing and decrementing
The "$inc" modifier can be used to change the value for an existing key or to create a new key if it does not already exist.

		> db.games.insert({"game" : "pinball", "user" : "joe"})

		> db.games.update({"game" : "pinball", "user" : "joe"}, ... {"$inc" : {"score" : 50}})

		> db.games.findOne()
		{
			"_id" : ObjectId("4b2d75476cc613d5ee930164"), 
			"game" : "pinball",
			"name" : "joe",
			"score" : 50
		}
		---The score key did not already exist, so it was created by "$inc" and set to the increment amount: 50

		> db.games.update({"game" : "pinball", "user" : "joe"},
		... {"$inc" : {"score" : 10000}})
		> db.games.find()
		{
			"_id" : ObjectId("4b2d75476cc613d5ee930164"), 
			"game" : "pinball",
			"name" : "joe",
			"score" : 10050
		}
		---The "score" key existed and had a numeric value, so the server added 10000 to it.

		**"$inc" can be used only on values of type integer, long, or double. If it is used on any type of value, it will fail.**

5. Array modifiers

---Array operators can be used only on keys with array values. 

---"$push" adds an element to the end of an array if the specified key already exists and creates a new array if it does not. 

		---push a comment onto the nonexistent "comments" array
		> db.blog.posts.findOne()
		{
			"_id" : ObjectId("4b2d75476cc613d5ee930164"), 
			"title" : "A blog post",
			"content" : "..."
		}

		> db.blog.posts.update({"title" : "A blog post"}, {$push : {"comments" :
		... {"name" : "joe", "email" : "joe@example.com", "content" : "nice post."}}}) 
		> db.blog.posts.findOne()
		{
			"_id" : ObjectId("4b2d75476cc613d5ee930164"), 
			"title" : "A blog post",
			"content" : "...",
			"comments" : [
				{
				"name" : "joe",
				"email" : "joe@example.com",
				"content" : "nice post." 
				}] 
		}
		---If we want to add another comment, we can simply use "$push" again:
		> db.blog.posts.update({"title" : "A blog post"}, {$push : {"comments" :
		... {"name" : "bob", "email" : "bob@example.com", "content" : "good post."}}})

		---A common use is wanting to add a value to an array only if the value is not already present. This can be done using a "$ne" in the query document.

		---To push an author onto a list of citations, but only if he isn't already there, use the following:
		> db.papers.update({"authors cited" : {"$ne" : "Richie"}}, 
		... {$push : {"authors cited" : "Richie"}})

		---This can also be done with "$addToSet"

		---origin
		> db.users.findOne({"_id" : ObjectId("4b2d75476cc613d5ee930164")})
		{
			"_id" : ObjectId("4b2d75476cc613d5ee930164"), "username" : "joe",
			"emails" : [
				"joe@example.com", 
				"joe@gmail.com", 
				"joe@yahoo.com"
			] 
		}

		> db.users.update({"_id" : ObjectId("4b2d75476cc613d5ee930164")}, 
		... {"$addToSet" : {"emails" : "joe@gmail.com"}})
		> db.users.findOne({"_id" : ObjectId("4b2d75476cc613d5ee930164")}) {
			"_id" : ObjectId("4b2d75476cc613d5ee930164"), 
			"username" : "joe",
			"emails" : [
				"joe@example.com", 
				"joe@gmail.com", 
				"joe@yahoo.com",
			] 
		}
		> db.users.update({"_id" : ObjectId("4b2d75476cc613d5ee930164")}, 
		... {"$addToSet" : {"emails" : "joe@hotmail.com"}})
		> db.users.findOne({"_id" : ObjectId("4b2d75476cc613d5ee930164")}) {
			"_id" : ObjectId("4b2d75476cc613d5ee930164"), 
			"username" : "joe",
			"emails" : [
				"joe@example.com", 
				"joe@gmail.com", 
				"joe@yahoo.com", 
				"joe@hotmail.com"
			] 
		}

		---You can also use "$addToSet" in conjunction with "$each" to add multiple unique values, which cannot be done with the "$ne"/ "$push" combination.
		> db.users.update({"_id" : ObjectId("4b2d75476cc613d5ee930164")}, {"$addToSet" : 
		... {"emails" : {"$each" : ["joe@php.net", "joe@example.com", "joe@python.org"]}}})

		> db.users.findOne({"_id" : ObjectId("4b2d75476cc613d5ee930164")})
		{
			"_id" : ObjectId("4b2d75476cc613d5ee930164"), "username" : "joe",
			"emails" : [
				"joe@example.com", 
				"joe@gmail.com", 
				"joe@yahoo.com", 
				"joe@hotmail.com" 
				"joe@php.net" 
				"joe@python.org"
			] 
		}

**There are a few ways to remove elements from an array.**

---If you want to treat the array like a queue or s stack, you can use "$pop", which can remove elements from either end.
- {$pop : {key : 1}} removes an element from the end of the array
- {$pop : {key : -1}} removes it from the beginning.

---"$pull" is used to remove elements of an array that match the given criteria.

		> db.lists.insert({"todo" : ["dishes", "laundry", "dry cleaning"]})
		---If we do the laundry first, we can remove it from the list with the following.
		> db.lists.update({}, {"$pull" : {"todo" : "laundry"}})

**Pulling removes all matching documents, not just a single match. If you have an array that looks like \[ 1, 1, 2, 1 \] and pull 1, you'll end up with a single-element array, \[ 2 \] **

6. Positional array modifications

---There are two ways to manipulate values in arrays: by position or by using the position operator (the "$" character).

Arrays use 0- based indexing.

		> db.blog.posts.findOne()
		{
			"_id" : ObjectId("4b329a216cc613d5ee930192"),
			"content" : "...", 
			"comments" : [
				{
					"comment" : "good post", 
					"author" : "John", 
					"votes" : 0
				}, 
				{
					"comment" : "i thought it was too short", 
					"author" : "Claire",
					"votes" : 3
				}, 
				{
					"comment" : "free watches", 
					"author" : "Alice",
					"votes" : -1
				} 
			]
		}

		---If you want to increment the number of votes for the first comment, we can say the following:
		> db.blog.update({"post" : post_id},
		... {"$inc" : {"comments.0.votes" : 1}})

		---if we dont know what index of the array to modify without querying for the document first and examing it. 
		---MongoDB has a positional operator, "$", that figures out which element of the array the query document matched and updates that element. 

		---if we have a user named John who updates his name to Jim, we can replace it in the comments by using the positional operator:
		
		db.blog.update({"comments.author" : "John"}, 
		... {"$set" : {"comments.$.author" : "Jim"}})

7. Modifier speed

---Some modifiers are faster than others. $inc modifies a document in place: it does not have to change the size of a document, only the value of a key, so it is very efficient.

---MongoDB leaves some padding around a document to allow for changes in size (figures out how much documents usually change in size and adjusts the amount of padding it leaves accordingly), but it will eventually have to allocate new space for a document if you make it much larger than it was originally. Compounding this slowdown, as arrays get longer, it takes MongoDB a longer amount of time to traverse the whole array, slowing down each array modification.

Using "$push" and other array modifiers is encouraged and often necessary, but it is good to keep in mind the trade-offs of such updates. If "$push" becomes a bottleneck, it may be worth pulling an embedded array out into a separate collection.

### Upserts

An upsert is a special type of update. If no document is found that matches the update criteria, a new document will be created by combining the criteria and update documents. If a matching document is found, it will be updated normally.

		blog = db.analytics.findOne({url : "/blog"})
		if (blog) {
			blog.pageviews++;
			db.analytics.save(blog); 
		}
 		else {
			db.analytics.save({url : "/blog", pageviews : 1}) 
		}

		---if we are running this code in multiple processes, we are also subject to a race condition where more than one document can be inserted for a given URL.
		---We can eliminate the race condition and cut down on the amount of code by just sending an upsert.

		db.analytics.update({"url" : "/blog"}, {"$inc" : {"visits" : 1}}, true)
		---This line does exactly what the previous code block does, except it's faster and atomic.

		> db.math.remove()
		> db.math.update({"count" : 25}, {"$inc" : {"count" : 3}}, true)
		---
		> db.math.update({"count" : 25}, {"$inc" : {"count" : 3}}) 
		without "true", nothing will happen, because the specified document cannot be found
		---
		> db.math.findOne() 
		{
			"_id" : ObjectId("4b3295f26cc613d5ee93018f"),
			"count" : 28 
		}
		---The upsert creates a new document with a "count" of 25 and then increments that by 3, giving us a document where "count" is 28. If the upsert option were not specified, {"count" : 25} would not match any documents, so nothing would happen.

**The save Shell Helper**
save is a shell function that lets you insert a document if it doesn't exist and update it if it does. It takes one argument: a document. If the document contains an "_id" key, save will do an upsert. Otherwise, it will do an insert.

		> var x = db.foo.findOne() 
		> x.num = 42
		42
		> db.foo.save(x)

		---Without save, the last line would have been a more cumbersome 

		db.foo.update({"_id" : x._id}, x)

### Updating Multiple Documents
Updates, by default, update only the first document found that matches the criteria. If there are more matching documents, they will remain unchanged. To modify all of the documents matching the criteria, you can pass true as **the fourth parameter** to update.
		> db.users.update({birthday : "10/13/1978"},
		... {$set : {gift : "Happy Birthday!"}}, false, true)

**To see the number of documents updated by a multiple update, you can run the getLastError database command.**

		---The 'n' key will contain the number of documents affected by the update:
		> db.runCommand({getLastError : 1})
		{
			"err" : null,
			"updatedExisting" : true, 
			"n" : 5,
			"ok" : true
		}

### Returning Updated Documents
1. findAndModify is called differently than a normal update and is a bit slower.

2. For example:

		---Suppose we have a collection of processes run in a certain order.

		{
			"_id" : ObjectId(), 
			"status" : state, 
			"priority" : N
		}

		---"status" is a string that can be "READY", "RUNNING", or "DONE". We need to find the job with the highest priority in the "READY" state, run the process function, and then update the status to "DONE"

		---We might try querying for the ready process, sorting by priority, and updating the status of the highest-priority process to mark it is "RUNNING". Once we have processed it, we update the status to "DONE".

		ps = db.processes.find({"status" : "READY").sort({"priority" : -1}).limit(1).next() 
		db.processes.update({"_id" : ps._id}, {"$set" : {"status" : "RUNNING"}}) do_something(ps);
		db.processes.update({"_id" : ps._id}, {"$set" : {"status" : "DONE"}})

		**This algorithm isn't great, because it is subject to a race condition.**
		---Suppose we have two threads running. If one thread (call it A) retrieved the document and another thread (call it B) retrieved the same document before A had updated its status to "RUNNING", then both threads would be running the same process. We can avoid this by checking the status as part of the update query,

		var cursor = db.processes.find({"status" : "READY"}).sort({"priority" : -1}).limit(1); 
		while ((ps = cursor.next()) != null) {
			ps.update({"_id" : ps._id, "status" : "READY"}, {"$set" : {"status" : "RUNNING"}});
			var lastOp = db.runCommand({getlasterror : 1}); 
			if (lastOp.n == 1) {
				do_something(ps);
				db.processes.update({"_id" : ps._id}, {"$set" : {"status" : "DONE"}}); break;
			}
			cursor = db.processes.find({"status" : "READY"}).sort({"priority" : -1}).limit(1);
		}
		--- depending on timing, one thread may end up doing all the work while another thread is uselessly trailing it.


		**findAndModify can return the item and update it in a single operation**
		> ps = db.runCommand({"findAndModify" : "processes", 
		... "query" : {"status" : "READY"},
		... "sort" : {"priority" : -1},
		... "update" : {"$set" : {"status" : "RUNNING"}})
		{
			"ok" : 1,
			"value" : {
				"_id" : ObjectId("4b3e7a18005cab32be6291f7"), 
				"priority" : 1,
				"status" : "READY"
			} 
		}



##REFERENCE
K. Chodorow, M. Dirolf, "MongoDB: The Definitive Guide", O'Reilly, 2010, US