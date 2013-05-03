---
layout: post
title: What you need to know to use couchdb and python + django
tags: [python, django, nosql, linux, couchdb]
last_updated: 02/05/2013
---

** <span class="disclaimer"> Warning!! </span> this post is in construction, but it could help others as is.**


When I first tried to use CouchDB and Django I found two articles showing how,
but both lacking some explanation. You are highly recomended to check them.
These are the two:

* [lethain.com/an-introduction-to-using-couchdb-with-django](http://lethain.com/an-introduction-to-using-couchdb-with-django/)
* [couchdbkit.org/docs/django-extension.html](http://couchdbkit.org/docs/django-extension.html)


### A little brief. What to expect.

So this is a NoSQL database. This means you won.t be running queries on it. You
will basically be accessing pre-defined mapping/reducing functions. This is an
exemple of a mapping function:

	function(doc) {
		if (doc.doc_type == "Language"){
			emit(doc._id, doc);
		}
	}

It will iterate over all your rows. and map an id to an object according to
your pre-defined logic. These dudes are stored on your couchdb server, so you
just go there and create new mapping functions, they will be availiable at a
given URL, and that.s how you are going to retrieve your objects.


### Motivation

I was starting a project that would use lots of versioning. Since I am not the
type of guy that always use the same tool, no matter the problem domain, I
opted for using some document based database.

My first stop was at [http://nosql-database.org/](http://nosql-database.org/).
They have a nice and comprehensive list of no-sql databases, and separated by
categories. My choice was CouchDB, first because I come from a Rails background,
so this was the perfect oportunity to get a deeper knoloedge on something I have
always just kiddig around. Second because it has a very nice REST API, and I
find it very cool. It really is .relaxing..

My second option was MongoDB. This DB also is very interesting to me, it is also
a very popular NoSQL database, but it doesn.t seems to support REST out of the
box. Another feature from CouchDB is the Futon, missing in Mongo. This might be
an futile feature, but it cool to have a nice interface to use when running all
your mapping and reducing stuffs.

So I opted for CouchDB. What now?


### Installation


This is a django project, so:

	$ pip install couchdbkit


### Configuration

	COUCHDB_DATABASES = (
		('django_app_name', 'http://10.0.2.15:5984/database_name'),
	)

And that is it!



### About CouchDB

There are a lot of places where you can find information about couchdb and 
how it works. A nice place to start is [http://couchdb.apache.org/](http://couchdb.apache.org/).
I will not spend time talking about NoSQL so I am cutting to the point and
speak key features. You can digg it by yourself for more information.

**``Versioning``**. CouchDB is based on versioning. This means that when you alter 
some document in your database it does not update fields in a row, instead
you will create a new copy of that document and store it as a new revision.

CouchDB has some important concepts you have to know. One is the **``Document``**
and other is the **``View``**. Sure, there are more, but with these two we can
go ahead.


#### Document

Let me quote a book that is a must read if you happen to work with CouchDB:

> Documents are CouchDB's central data structure. The idea behind a document is,
> unsurprisingly, that of a real-world document.a sheet of paper such as an invoice, a
> recipe, or a business card. We already learned that CouchDB uses the JSON format to
> store documents. (...)
> 
> [CouchDB - The Definitive Guide](http://guide.couchdb.org/)



#### Views

This could be compared to your queries. But they are not. They
are mapping/reduce functions. When you query a view, it does all the map/reduce
functions you have written and return a document with the results.

View results are stored into files, and will run again only if a document
has changed.

Let's make some ``Documents`` and ``Views``.


#### Futon

Before engaging in the views development per say, let's have a look at Futon,
the default web interface for CouchDB. You can get there by openning
``http://localhost:5984/_utils/``. You should land on a screen that looks like
the one bellow.

![Initial futon screen](/images/couch_db_futon_initial.png)

One of the things you can note is that it is empty, so let's start doing some 
work on this. We are going to create a database. Click on "Create Database..."
and name it "languages" (all the examples are from the application I am 
building, and because I could NOT use the same "fruits" database from the other
articles.)

![Clean database screen](/images/couch_db_futon_new_database.png)

Now you should have a database, but with no documents. Creating documents...
Well nevermind... Let's create documents within our django application. You
should be able to create documents from inside the interface with no problem.
Be said that it is as simple as filling the fields you like. Remember, is schema
free.




### Django Project Illustration

A simple django application structure.


languages/models.py
	from couchdbkit.ext.django.schema import *

	class Language(Document):
	    name            = StringProperty(required=True)
	    license         = StringProperty(required=False)
	    description     = StringProperty(required=False)

But what it means? This is a document. CouchDB is schema free, so you won't have
to make any initial configuration, as you would using a relational database.
This class represent a document that has a name, description and a code. Of
course we kind of freeze our database saying it will have those fields, but the
documents really are not. This is just for mapping our documents to our class.

**Important**! Using CouchdbKit it will automatically add into your document
the field ``doc_type`` equal to your class name.

<br>

languages/views.py
	# some imports were hidden
	from langs.models import Language

	def new(request):
	    c = {}
	    c.update(csrf(request))
	    return render_to_response("langs_templates/new.haml", c)

	def create(request):
	    lang_desc = Language()
	    lang_desc.name = request.POST['name']
	    lang_desc.license = request.POST['license']
	    lang_desc.description = request.POST['description']

	    lang_desc.save()

	    return render_to_response('langs_templates/index.haml')


And that is it!

So another simple code that will save an object to our database. Creates an
instance of Language and set its attributes. Saves it. Render the template.

Get some view online. It can be as the following:
	<form action='/languages/create/' method='post'>
		Name
		<input type='text' placeholder='Language name' name='name' />

		license
		<input type='text' placeholder='Language licensing' name='license' />

		Description
		<textarea rows='6' placeholder='Language description' name='description'></textarea>
		<button class='btn' type='submit'>Submit</button>
	</form>

This is just some code for you to follow, it is not guaranteed to work and 
you probably have a very different enviroment. So be sure to stick with the
logic. Be sure you get your routes working ok.

Now create some documents. You can create at the ``Django shell`` and save them.

	>>> from langs.models import Language
	>>> l = Language()
	>>> l.name = "python"
	>>> l.description = "the best lang"
	>>> l.license = "I don't care"
	>>> l.save()
	>>> l = Language()
	>>> l.name = "java"
	>>> l.description = "a language that is slow"
	>>> l.license = "also dont care"
	>>> l.save()

And right now, if you open your web interface, you will see that there two
languages were created. Awesome! So it is working already.

![Two languages added](/images/couch_db_futon_two_languages.png)


##### Continue reading...
Right now you should check these documentation:

* [Document](http://couchdbkit.org/docs/api/couchdbkit.ext.django.schema.Document-class.html)
* [DocumentSchema](http://couchdbkit.org/docs/api/couchdbkit.schema.base.DocumentSchema-class.html)
* [DocumentMeta](http://couchdbkit.org/docs/api/couchdbkit.ext.django.schema.DocumentMeta-class.html)
* [SchemaProperty](http://couchdbkit.org/docs/api/couchdbkit.schema.properties_proxy.SchemaProperty-class.html)
* [SchemaListProperty](http://couchdbkit.org/docs/api/couchdbkit.schema.properties_proxy.SchemaListProperty-class.html)
* [SchemaDictProperty](http://couchdbkit.org/docs/api/couchdbkit.schema.properties_proxy.SchemaDictProperty-class.html)



#### Until here...

We have talked a little about what it is like to have a NoSQL database;
motivated our trip to the NoSQL world; installed CouchDB; Addded its
configurations to our settings.py; Contructed a simple model that represents a
Document; Created a view that will process a POST and save our object to the
database.





#### Mapping

Lets have a look at our first example:
	function(doc) {
		if (doc.doc_type == "Language"){
			emit(doc._id, doc);
		}
	}

What this does again? It iterates over ALL documents in your database, check
if they are of the Language type (this is set after the class name that
represents this document) and will return the id and the document as value.



