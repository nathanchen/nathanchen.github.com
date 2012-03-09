---
layout: post
title: "MongoDB The Definitive Guide(1)"
category: Study Notes
tags: ["MongoDB", "读书"]
---
{% include JB/setup %}

## Chapter 2 Getting Started


1. A document is the basic unit of data for MongoDB, roughly equivalent to a row in a relational database management system.
2. A collection can be thought of as the schema-free equivalent of a table.
3. MongoDB comes with a simple but powerful Javascript shell.
4. Every document has a special key, "_id", that is unique across the document's collection.

### Document

1. A document: an ordered set of keys with associated values.
Example:

		{"greeting" : "Hello world!"}

**Several Important concepts**

- Key/ value pairs in documents are ordered - the following two documents are distinct from each other, example:

		{"greeting" : "Hello world!", "foo" : 3}
		{"foo" : 3, "greeting" : "Hello world!"}

- Values in documents are not just "blobs". They can be one of several different date types.

**Key naming exceptions**

- Keys must not contain the character \0 (the null character)
- The . and $ characters have some special properties and should be used only in certain circumstances. In general, they should be considered reserved, and drivers will complain if they are used inappropriately.
- Keys starting with _ should be considered reserved; although this is not strictly enforced.

These documents are distince:
	
		{"foo" : 3}
		{"foo" : "3"}
As are as these:

		{"foo" : 3}
		{"Foo" : 3}

**MongoDB cannot contain duplicate keys.**

### Collections

1. A collection is a group of documents. If a document is the MongoDB analog of a row in a relational database, then a collection can be thought of as the analog to a table.

2. Collection are schema-free. This means that the documents within a single collection can have any number of different "shapes".

3. Any document can be put into any collection.

4. Why should we use more than one collection? There are several good reasons:

- Keeping different kinds of documents in the same collection can be a nightmare for developers and admins. 
- It is much faster to get a list of collections than to extract a list of the types in a collection.
- Grouping documents of the same kind together in the same collection allows for data locality. Getting several blog posts from a collection containing only posts will likely require fewer disk seeks than getting the same posts from a collection containing posts and author data. 
- By putting only documents of a single type into the same collection, we can index our collections more efficiently.

**Collection naming exceptions**

A collection is identified by its name. Collection names can be any UTF-8 string, with a few restrictions:

- The empty string ("") is not a valid collection name.
- Collection names may not contain the character \0 (the null character) because this delineates the end of a collection name.
- You should not create any collections that start with system., a prefix reserved for system collections. 
- User-created collections should not contain the reserved character $ in the name. You should not use $ in a name unless you are accessing one of these collections.

**One convention for organizing collections is to use namespaced subcollections separeted by the . character.**

### Databases

1. In addition to grouping documents by collection, MongoDB groups collections into databases.
2. A single instance of MongoDB can host several databases, each of which can be thought as completely independent.
3. Each database is stored in separate files on disk.
4. A good rule of thumb is to store all data for a single application in the same database. Separate databases are useful when storing the data for several application or users on the same MongoDB server.

**Database naming exceptions**

- The empty string ("") is not a valid database name.
- A database name cannot contain any of these characters: '',,,
- Database names should be all lowercase.
- Database names are limited to a maximum of 64 bytes.

##Starting MongoDB

1. mongod will use the default data directory, /data/db/ and port 27017
2. mongod also sets up a very basic HTTP server that listens on a port 1000 higher than the main port. in this case 28017.
3. You can safely stop mongod by typing Ctrl+c in the shell that is running the server.

		--- select which database to use---
		> use foobar
		Switched to db foobar

		--- look at the db variable---
		> db
		foobar
		---

##Basic operations with MongoDB Shell
1. Create

		> post = {"title" : "My Blog Post",
		... "content" : "Here's my blog post.", 
		... "date" : new Date()}
		{
			"title" : "My Blog Post",
			"content" : "Here's my blog post.",
			"date" : "Sat Dec 12 2009 11:23:21 GMT-0500 (EST)"
		}

		> db.blog.insert(post)
		
		---We can see it by calling find on the collection:
		> db.blog.find()
		{
			"_id" : ObjectId("4b23c3ca7525f35f94b60a2d"), 
			"title" : "My Blog Post",
			"content" : "Here's my blog post.",
			"date" : "Sat Dec 12 2009 11:23:21 GMT-0500 (EST)"
		}

2. Read
find returns all of the documents in a collection. If we just want to see one document from a collection, we can use findone:
		
		> db.blog.findOne()
		{
			"_id" : ObjectId("4b23c3ca7525f35f94b60a2d"),
			"title" : "My Blog Post",
			"content" : "Here's my blog post.",
			"date" : "Sat Dec 12 2009 11:23:21 GMT-0500 (EST)"
		}

3. Update
Update takes (at least) two parameters: the first is the criteria to find which document to update, and the second is the new document.
Example:

		---We decide to enable comments on the blog post
		
		---The first step is to modify the variable post and add a "comments" key:
		> post.comments = []
		[]
		---Then we perform the update, replacing the post titled "My Blog Post" with our new version of the document:
		> db.blog.update({title : "My Blog Post"}, post)

		> db.blog.find()
		{
			"_id" : ObjectId("4b23c3ca7525f35f94b60a2d"), 
			"title" : "My Blog Post",
			"content" : "Here's my blog post.",
			"date" : "Sat Dec 12 2009 11:23:21 GMT-0500 (EST)" 
			"comments" : [ ]
		}

4. Dalete
remove deletes documents permanently from the database. Called with no parameters, it removes all documents from a collection. It can also take a document specifying criteria for removal.

		> db.blog.remove({title : "My Blog Post"})


5. Help
Help for database-level commands is provided by db.help(); and help at the collections can be accessed with db.foo.help().

A good way fo figuring out what a function is doing is to type it without the parentheses. This will print the JavaScript source code for the function.

		---if you needed to perform some operation on every blog subcollection, you could iterate through them with something like this:
		var collections = ["posts", "comments", "authors"];
		for (i in collections) {
			doStuff(db.blog[collections[i]]);
		 }
		
		---instead of this:
		doStuff(db.blog.posts); 
		doStuff(db.blog.comments); 
		doStuff(db.blog.authors);
