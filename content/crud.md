+++
date = "2014-10-22T16:57:07-04:00"
title = "CRUD"
+++

In the last article [link needed], we covered the basics of installing and connecting to MongoDB via a Java application.  In this post, 
I'll give an introduction to CRUD (Create, Read, Update, Delete) operations using the Java driver.  As in the previous article, 
if you want to play along and code as we go, you can use these tips to get the tests in the 
[Getting Started project](https://github.com/trishagee/mongodb-getting-started) to go green.
 
## Creating documents

In the last article [link needed], we introduced [documents](http://docs.mongodb.org/manual/reference/glossary/#term-document) and how to 
create them from Java and insert them into MongoDB, so I'm not going to repeat that here.  But if you want a reminder, or simply want to 
skip to playing with the code, you can take a look at 
[Exercise3InsertTest](https://github.com/trishagee/mongodb-getting-started/blob/master/src/test/java/com/mechanitis/mongodb/gettingstarted/Exercise3InsertTest.java).   

## Querying

Putting stuff in the database is all well and good, but you'll probably want to query the database to get data from it.
   
In the last article we covered some basics on using [find()](http://docs.mongodb.org/manual/reference/method/db.collection.find/) to get 
data from the database.  We also showed an example in [Exercise4RetrieveTest](https://github.com/trishagee/mongodb-getting-started/blob/master/src/test/java/com/mechanitis/mongodb/gettingstarted/Exercise4RetrieveTest.java). But MongoDB supports more than simply getting a single document by ID or getting all the documents in a 
collection.  As I mentioned, you can query by example, building up a 
[query document](http://docs.mongodb.org/manual/tutorial/query-documents/) that looks a similar shape to the one you want.
 
For the following examples I'm going to assume a document which looks something like this:

    person = {
      _id: "anId",
      name: "A Name",
      address: {
        street: "Street Address",
        city: "City",
        phone: 12345
      }
      books: [ 27464, 747854, ...]
    }  

### Find a document by ID

To recap, you can easily get a document back from the database using the unique ID:

    DBCursor cursor = collection.find(new BasicDBObject("_id", "theId"));
<script src="https://gist.github.com/trishagee/11b4e539612ce5262321.js"></script>

...and you get the values out of the document (represented as a `DBObject`) using a `Map`-like syntax:

    (String) cursor.one().get("name")
<script src="https://gist.github.com/trishagee/ebf354f8de25663eae79.js"></script>

In the above example, because you've queried by ID (and you knew that ID existed), you can be sure that the cursor has a single document 
that matches the query.  Therefore you can use `cursor.one()` to get it.
 
### Find all documents matching some criteria

In the real world, you won't always know the ID of the document you want.  You could be looking for all the people with a particular name, 
for example.

In this case, you can create a query document that has the criteria you want:

    DBCursor results = collection.find(new BasicDBObject("name", "The name I want to find"));
<script src="https://gist.github.com/trishagee/d82e26bea5fccfa9cbee.js"></script>
    
You can find out the number of results:

    results.size();
<script src="https://gist.github.com/trishagee/c1328cfbc6618fad0c4c.js"></script>
    
and you can, naturally, iterate over them:

    for (DBObject result : results) {
        // do something with each result
    }
<script src="https://gist.github.com/trishagee/bd7bc1f865a6a0303cd2.js"></script>

**A note on batching**  
The cursor will fetch results in batches from the database, so if you run a query that matches a lot of documents, 
you don't have to worry that every document is loaded into memory immediately.  For most queries, 
the [first batch returned will be 101 documents](http://docs.mongodb.org/manual/core/cursors/#cursor-batches). But as you iterate over 
the cursor, the driver will automatically fetch further batches from the server.  So you don't have to worry about managing batching in 
your application. But you do need to be aware that if you iterate over the whole of the cursor (for example to put it into a `List`), 
you will end up fetching all the results and putting them in memory.

You can get started with [Exercise5SimpleQueryTest](https://github.com/trishagee/mongodb-getting-started/blob/master/src/test/java/com/mechanitis/mongodb/gettingstarted/Exercise5SimpleQueryTest.java). 

### Selecting Fields
Generally speaking, MongoDB treats a document as a whole - unlike relational databases, where you pick and choose the columns 
that you want to return from whichever tables you wish to query, MongoDB assumes that you've modelled your data so that the stuff you 
want together is in a single document, rather than scattered all over your database.
 
However, you can choose to return just the fields that you care about (for example, you might have a large document and not need all the 
values).  You do this by passing a second parameter into the `find` method that's another `DBObject` defining the fields you want to
return.  In this example, we'll search for people called "Smith", and return only the `name` field.  To do this we pass in a 
`DBObject` representing `{name: 1}`:   

    DBCursor results = collection.find(new BasicDBObject("name", "Smith"), 
                                       new BasicDBObject("name", 1));
<script src="https://gist.github.com/trishagee/440cdf337eadd99c9e04.js"></script>

You can also use this method to exclude fields from the results. Maybe we might want to exclude an unnecessary subdocument
from the results - let's say we want to find everyone called "Smith", but we don't want to return the `address`.  We do this by 
passing in a zero for this field name, i.e. `{address: 0}`:

    DBCursor results = collection.find(new BasicDBObject("name", "Smith"),
                                       new BasicDBObject("address", 0));
<script src="https://gist.github.com/trishagee/66759199d1459563ad7f.js"></script>

With this information, you're ready to tackle 
[Exercise6SelectFieldsTest](https://github.com/trishagee/mongodb-getting-started/blob/master/src/test/java/com/mechanitis/mongodb/gettingstarted/Exercise6SelectFieldsTest.java)

### Query Operators

As I mentioned in the previous article, your fields can be one of a number of types, including numeric.  This means that you can do 
queries for numeric values as well.  Let's assume, for example, that our person has a `numberOfOrders` field, 
and we wanted to find everyone who had ordered more than, let's say, 10 items.  You can do this using the 
[$gt](http://docs.mongodb.org/manual/reference/operator/query/gt/#op._S_gt) operator:

    DBCursor results = collection.find(new BasicDBObject("numberOfOrders", new BasicDBObject("$gt", 10)));
<script src="https://gist.github.com/trishagee/3ee81da8e419aca9944b.js"></script>

Note that you have to create a further subdocument containing the `$gt` condition to use this operator.  All of the query operators [are 
documented](http://docs.mongodb.org/manual/reference/operator/query/), and work in a similar way to this example.

You might be wondering what terrible things could happen if you try to perform some sort of numeric comparison on a field that is a String, 
since the database supports any type of value in any of the fields (and in Java the values are Objects so you don't get the benefit of type 
safety).  So, what happens if you do this?

    DBCursor results = collection.find(new BasicDBObject("name", new BasicDBObject("$gt", 10)));
<script src="https://gist.github.com/trishagee/8bb1e706a5d351a295b1.js"></script>

The answer is you get zero results (assuming all your documents contain names that are Strings), 
and you don't get any errors. The flexible nature of the document schema allows you to mix and match types and query without error.

You can use this technique to get the test in 
[Exercise7QueryOperatorsTest](https://github.com/trishagee/mongodb-getting-started/blob/master/src/test/java/com/mechanitis/mongodb/gettingstarted/Exercise7QueryOperatorsTest.java)
to go green - it's a bit of a daft example, but you get the idea.

### Querying Subdocuments

So far we've assumed that we only want to query values in our top-level fields.  However, we might want to query for [values in a
subdocument](http://docs.mongodb.org/manual/reference/method/db.collection.find/#query-subdocuments) - for example, 
with our person document, we might want to find everyone who lives in the same city.  We can use 
[dot notation](http://docs.mongodb.org/manual/core/document/#document-dot-notation) like this:

    DBObject findLondoners = new BasicDBObject("address.city", "London");
    collection.find(findLondoners));
<script src="https://gist.github.com/trishagee/cd7537b07a8d10e3350c.js"></script>

We're not going to use this technique in a query test, but we will use it later when we're doing updates.

### Familiar methods

I mentioned earlier that you can iterate over a cursor, and that the driver will fetch results in batches.  However, 
you can also use the familiar-looking [skip()](http://docs.mongodb.org/manual/reference/method/cursor.skip/) and 
[limit()](http://docs.mongodb.org/manual/reference/method/cursor.limit/) methods.  You can use these to fix up the test in 
[Exercise8SkipAndLimitTest](https://github.com/trishagee/mongodb-getting-started/blob/master/src/test/java/com/mechanitis/mongodb/gettingstarted/Exercise8SkipAndLimitTest.java).

### A last note on querying: Indexes

Like a traditional database, you can add indexes onto the database to improve the speed of regular queries.  There's extensive 
[documentation on indexes](http://docs.mongodb.org/manual/indexes/) which you can read at your own leisure.  However, 
it is worth pointing out that, if necessary, you can programmatically create indexes via the Java driver, 
using `createIndexes`.  For example:

    collection.createIndex(new BasicDBObject("fieldToIndex", 1));
<script src="https://gist.github.com/trishagee/e866e162f66ae7c38c6e.js"></script>

There is a very simple example for creating an index in [Exercise9IndexTest](https://github.com/trishagee/mongodb-getting-started/blob/master/src/test/java/com/mechanitis/mongodb/gettingstarted/Exercise9IndexTest.java),
but indexes are a full topic on their own, and the purpose of this part of the tutorial is to merely make you aware of their existence 
rather than provide a comprehensive tutorial on their purpose and uses.

## Updating values

Now you can insert into and read from the database.  But your data is probably not static, especially as one of the benefits of MongoDB 
is a [flexible schema](http://docs.mongodb.org/manual/data-modeling/) that can evolve with your needs over time.

In order to [update](http://docs.mongodb.org/manual/tutorial/modify-documents/) values in the database, 
you'll have to define the query criteria that states which document(s) you want to update, and you'll have to pass in the document that 
represents the updates you want to make. 

There are a few things to be aware of when you're updating documents in MongoDB, once you understand these it's as simple as everything 
else we've seen so far.

Firstly, by default only the first document that matches the query criteria is updated.

Secondly, if you pass in a document as the value to update to, this new document will replace the whole existing document.  If you think 
about it, the common use-case will be: you retrieve something from the database; you modify it based on some criteria from your 
application or the user; then you save the updated document to the database.

I'll show the various types of updates (and point you to the code in the test class) to walk you through these different cases.

### Simple Update: Find a document and replace it with an updated one

We'll carry on using our simple Person document for our examples.  Let's assume we've got a document in our database that looks like:
 
    person = {
      _id: "jo",
      name: "Jo Bloggs",
      address: {
        street: "123 Fake St",
        city: "Faketon",
        phone: 5559991234
      }
      books: [ 27464, 747854, ...]
    } 

Maybe Jo goes into witness protection and needs to change his/her name.  Assuming we've got `jo` populated in a `DBObject`, 
we can make the appropriate changes to the document and save it into the database:

    DBObject jo =                                       // get the document representing jo
    jo.put("name", "Jo In Disguise");                   // replace the old name with the new one
    collection.update(new BasicDBObject("_id", "jo"),   // find jo by ID
                      jo);                              // set the document in the DB to the new document for Jo
<script src="https://gist.github.com/trishagee/6248d507a3882da9bf52.js"></script>

You can make a start with 
[Exercise10UpdateByReplacementTest](https://github.com/trishagee/mongodb-getting-started/blob/master/src/test/java/com/mechanitis/mongodb/gettingstarted/Exercise10UpdateByReplacementTest.java).

### Update Operators: Change a field

But sometimes you won't have the whole document to replace the old one, sometimes you just want to update a single field in whichever 
document matched your criteria.

Let's imagine that we only want to change Jo's phone number, and we don't have a `DBObject` with all of Jo's details but we do have the ID 
of the document.  If we use the [$set](http://docs.mongodb.org/manual/reference/operator/update/set/) operator, 
we'll replace only the field we want to change:

    collection.update(new BasicDBObject("_id", "jo"),
                      new BasicDBObject("$set", new BasicDBObject("phone", 5559874321)));
<script src="https://gist.github.com/trishagee/d31af4fddd650e791195.js"></script>

There are a number of [other operators](http://docs.mongodb.org/manual/reference/operator/update-field/) for performing updates on 
documents, for example [$inc](http://docs.mongodb.org/manual/reference/operator/update/inc/#up._S_inc) which will increment a numeric 
field by a given amount.
 
Now you can do [Exercise11UpdateAFieldTest](https://github.com/trishagee/mongodb-getting-started/blob/master/src/test/java/com/mechanitis/mongodb/gettingstarted/Exercise11UpdateAFieldTest.java)

### Update Multiple

As I mentioned earlier, by default the `update` operation updates the first document it finds and no more.  You can, however, 
set the `multi` flag on `update` to update everything.

So maybe we want to update everyone in the database to have a country field, and for now we're going to assume all the current people are
in the UK:
 
    collection.update(new BasicDBObject(),
                      new BasicDBObject("$set", new BasicDBObject("country", "UK")), false, true);
<script src="https://gist.github.com/trishagee/9467f0a4aa19ce855a8c.js"></script>

The query parameter is an empty document which finds everything; the second boolean (set to `true`) is the flag that says to update all the 
values which were found.

Now we've learnt enough to complete the two tests in 
[Exercise12UpdateMultipleDocumentsTest](https://github.com/trishagee/mongodb-getting-started/blob/master/src/test/java/com/mechanitis/mongodb/gettingstarted/Exercise12UpdateMultipleDocumentsTest.java)

### Upsert

Finally, the last thing to mention when updating documents is Upsert (Update-or-Insert).  This will search for a document matching the 
criteria and either: update it if it's there; or insert it into the database if it wasn't.

Like "update multiple", you define an upsert operation with a magic boolean.  It shouldn't come as a surprise to find it's the first 
boolean param in the update statement (since "multi" was the second):

    collection.update(query, personDocument, true, false);
<script src="https://gist.github.com/trishagee/0ac3546c4be61a48d0fb.js"></script>

Now you know everything you need to complete the test in 
[Exercise13UpsertTest](https://github.com/trishagee/mongodb-getting-started/blob/master/src/test/java/com/mechanitis/mongodb/gettingstarted/Exercise13UpsertTest.java)

## Removing from the database

Finally the D in CRUD - Delete.  The syntax of a remove should look familiar now we've got this far, 
you pass a document that represents your selection criteria into the remove method.  So if we wanted to delete `jo` from our 
database, we'd do:

    collection.remove(new BasicDBObject("_id", "jo"));
<script src="https://gist.github.com/trishagee/23ee8cc8bec4354a1227.js"></script>

Unlike `update`, if the query matches more than one document, all those documents will be deleted (something to be aware of!). If we 
wanted to remove everyone who lives in London, we'd need to do:

    collection.remove(new BasicDBObject("address.city", "London"));
<script src="https://gist.github.com/trishagee/3d5d8a0b8bc97da91a7d.js"></script>

That's all there is to remove, you're ready to finish off 
[Exercise14RemoveTest](https://github.com/trishagee/mongodb-getting-started/blob/master/src/test/java/com/mechanitis/mongodb/gettingstarted/Exercise14RemoveTest.java)

## Conclusion

Unlike traditional databases, you don't create SQL queries in MongoDB to perform CRUD operations. Instead, 
operations are done by constructing documents both to query the database, and to define the operations to perform.

While we've covered what the basics look like in Java, there's loads more documentation on all the core concepts in the MongoDB 
documentation:

 - [Query Documents](http://docs.mongodb.org/manual/tutorial/query-documents/) 
 - [CRUD Operations](http://docs.mongodb.org/manual/core/crud-introduction/)
 - [Indexes](http://docs.mongodb.org/manual/indexes/)
 