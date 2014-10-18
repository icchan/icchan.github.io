---
title: Auto Increment for MongoDB with the Java driver
layout: post
---

So at my new job I've been tasked with migrating out awfully designed Postgres database into MongoDB, while at the same time migrating our PHP/CodeIgniter REST service to Java/SpringMVC.

Our data is a deep/wide object graph with a complex and variable set of properties, so it is actually well suited to NoSQL or a document database or whatever you wanna call it.

Since I'm migrating a system and not creating a new one I had a few constraints. The first one I'm going to tackle is that we need to have auto incremented integer primary keys for our entities. And MongoDB has UUIDs by default, so I had to find another way. 
I found the solution 
[here](http://shiflett.org/blog/2010/jul/auto-increment-with-mongodb"), and it was all pretty easy. If you are too lazy to read it, it basically says to keep a separate collection to store your sequences (kinda like Postgres does automatically with its sequences).

I tested this out in the admin console and it was all great. But the [Java driver](http://api.mongodb.org/java/2.6-pre-/index.html) is not quite as straightforward as the console, its not just JSON, it has lots of new Objects like BSONCallback, BasicDBObject and MongoOptions which don't make much sense to n00bs like me... 

It takes a while to get your head around. Since I haven't been able to find a good example anywhere online on how to do this using the Java driver, I'll post my code here. I hope it helps someone out there.

```Java
/**
 * Get the next unique ID for a named sequence.
 * @param db Mongo database to work with
 * @param seq_name The name of your sequence (I name mine after my collections)
 * @return The next ID
 */
public static String getNextId(DB db, String seq_name) {
	String sequence_collection = "seq"; // the name of the sequence collection
	String sequence_field = "seq"; // the name of the field which holds the sequence

	DBCollection seq = db.getCollection(sequence_collection); // get the collection (this will create it if needed)

	// this object represents your "query", its analogous to a WHERE clause in SQL
	DBObject query = new BasicDBObject();
    query.put("_id", seq_name); // where _id = the input sequence name

    // this object represents the "update" or the SET blah=blah in SQL
    DBObject change = new BasicDBObject(sequence_field, 1);
    DBObject update = new BasicDBObject("$inc", change); // the $inc here is a mongodb command for increment

    // Atomically updates the sequence field and returns the value for you
    DBObject res = seq.findAndModify(query, new BasicDBObject(), new BasicDBObject(), false, update, true, true);
    return res.get(sequence_field).toString();
}
```
