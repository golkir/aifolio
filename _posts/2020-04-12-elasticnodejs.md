---
layout: post
title: Integrating Elasticsearch Into Your Node.js Application
date: 2020-04-12 11:12:00-0400
description: 
tags: elasticsearch
categories: sample-posts
related_posts: false
---

In previous tutorials, we have already discussed how to use Elasticsearch native clients in [Rust](https://qbox.io/blog/elasticsearch-rest-client-idiomatic-rust-tutorial), [Java](https://qbox.io/blog/rest-calls-new-java-elasticsearch-client-tutorial?utm_source=qbox.io&utm_medium=article&utm_campaign=elasticsearch-rest-client-idiomatic-rust-tutorial), and [Python](https://qbox.io/blog/python-scripts-interact-elasticsearch-examples?utm_source=qbox.io&utm_medium=article&utm_campaign=elasticsearch-rest-client-idiomatic-rust-tutorial) among others. Today, we’re going to cover Node.js, - a popular JavaScript server-side solution based on event-driven architecture efficient for development of fast servers, I/O heavy applications, and real-time applications (RTAs). Node.js ecosystem is one of the fastest growing and it is now possible to interact with the Elasticsearch API using a Node.js client. 
[Elasticsearch.js](https://github.com/elastic/elasticsearch-js) is the official Elasticsearch client for Node.js that ships with:

 - one-to-one mapping with Elasticsearch REST API  
 -  intelligent handling of node/connection failures
 - support for load balancers
 - integration into modern browsers      
 - efficient support for asynchronous operations with JavaScript
   Promises.

[Elasticsearch.js](https://github.com/elastic/elasticsearch-js) has one of the widest support for standard and advanced Elasticsearch features and is regularly tested against ES releases 0.90.12 and greater. The client is regularly updated and maintained by the vibrant Node.js community. 

**Tutorial**
------------

For this tutorial, we will be using hosted Elasticsearch on [Qbox.io](https://qbox.io/). You can sign up and  [launch Elasticsearch cluster here](https://qbox.io/signup?utm_source=blog&utm_campaign=tutorial&utm_term=launch_your_cluster&utm_medium=article),  or click "Get Started" in the header navigation menu. If you need help setting up,  refer to “[Provisioning a Qbox Elasticsearch Cluster](https://qbox.io/blog/provisioning-a-qbox-elasticsearch-cluster?utm_source=tutorial&utm_term=provision&utm_medium=article&utm_campaign=index-attachments-files-elasticsearch-mapper).”
Before you proceed to the tutorial, ensure that the latest version of Node.js is installed on your computer. If you are not sure how to do this, please, refer to [Node.js installation guide](https://nodejs.org/uk/download/package-manager/) for directions. 
Once we have Node.js up and running, we need to install Elasticsearch.js in our Node.js application. You can install the official elasticsearch.js module via npm (Node Package Manager) command:

    npm install elasticsearch

Using the elasticsearch.js module in Node we can easily connect to and interact with our Elasticsearch cluster on Qbox.io:
```javascript
var elasticsearch = require('elasticsearch');
var client = new elasticsearch.Client({
   hosts: [ 'https://username:password@host:port']
});
```
The client  constructor accepts a config object/hash where you can define [defaults parameters](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/configuration.html), or even entire classes, for the client to use.
In addition, our client variable inherits all methods of the elasticsearch.js Client constructor which corresponds to the official Elasticsearch API. 

We can now ping to our Qbox.io cluster to check if everything is ok.
``` javascript
client.ping({
     requestTimeout: 30000,
 }, function(error) {
     if (error) {
         console.error('elasticsearch cluster is down!');
     } else {
         console.log('Everything is ok');
     }
 });
```
If something goes wrong, you can debug using the error object returned by the callback. Elasticsearch API also includes other useful methods to inspect your Qbox.io cluster. For example, using a cluster object you can check cluster health (**cluster.health**), stats, current state (e.g operational), and more. 

**Features of Elasticsearch.js**
--------------------------------
Now, we are going to discuss basic features of elasticsearch.js such as indexing, creating documents, and search. 

**Indexing**
------------

Unlike conventional databases, an Elasticsearch index is a place to store related documents. In this example, we’re going to create an index ‘blog’ to store ‘posts’ in it. With elasticsearch.js it can be done as easy as that: 
```javascript
 client.indices.create({
     index: 'blog'
 }, function(err, resp, status) {
     if (err) {
         console.log(err);
     } else {
         console.log("create", resp);
     }
 });
```
If the index already exists, you get **‘index_already_exists_exception’**. Otherwise, a new index ready to store your documents is created. 

**Adding Documents to an Index**
--------------------------

Now as we've got an index, we need some documents to post to it. For that purpose, we are going to index a new document with a type ‘posts’. 
```javascript
 client.index({
     index: 'blog',
     id: '1',
     type: 'posts',
     body: {
         "PostName": "Integrating Elasticsearch Into Your Node.js Application",
         "PostType": "Tutorial",
         "PostBody": "This is the text of our tutorial about using Elasticsearch in your Node.js application.",
     }
 }, function(err, resp, status) {
     console.log(resp);
 });
```

Here, the body is a typed JSON document in an index that makes it searchable. We have defined a body object with key parameters of our document: **PostName, PostType, and PostBody**. You can add whatever parameters of the blog post you need. 
The inner process of posting a new document works like this. If **id** param is not specified, Elasticsearch will auto-generate a unique **id**. In case you specify an **id**, Elasticsearch will either create a new document (if it does not exist) or update an existing one. Similarly, Elasticsearch will perform optimistic concurrency control when the version argument is used. 
If the document is successfully saved, we get the following response from the server:
```javascript
{ _index: 'blog',
  _type: 'posts',
  _id: '1',
  _version: 1,
  result: 'created',
  _shards: { total: 1, successful: 1, failed: 0 },
  created: true }
```
If the index exists and we didn’t specify a version argument, the existing document will be simply updated by the new one and specified as its second version. 
```javascript
{ _index: 'blog',
  _type: 'posts',
  _id: '1',
  _version: 2,
  result: 'updated',
  _shards: { total: 1, successful: 1, failed: 0 },
  created: false }
```
In addition to creating documents, elasticsearch.js  supports index flushing, analysis, recovery, creation of mappings, simulation, and other advanced methods.
 
**

**Search Documents Using Query Params**
--------------------------------------

**
Elasticsearch.js has a mature search functionality that supports both simple queries and Elasticsearch Query DSL. To search documents using simple query you need to specify a **‘q’** parameter in your request object. In this example, we are searching for all posts with “Node.js” in the post title. 
```javascript
client.search({
    index: 'blog',
    type: 'posts',
    q: 'PostName:Node.js'
}).then(function(resp) {
    console.log(resp);
}, function(err) {
    console.trace(err.message);
});
```


**

**Elasticsearch Query DSL**
-----------------------

**
If you want to have a fine-grained control over document search, Elasticsearch offers a query DSL. 
```javascript
client.search({
    index: 'blog',
    type: 'posts',
    body: {
        query: {
            match: {
                "PostName": 'Node.js'
            }
        }
    }
}).then(function(resp) {
    console.log(resp);
}, function(err) {
    console.trace(err.message);
});
```
A successful response looks like this:
```javascript
{ took: 407,
  timed_out: false,
  _shards: { total: 4, successful: 4, failed: 0 },
  hits: { total: 1, max_score: 0.26742277, hits: [ [Object] ] } }
```

It shows that a total number of documents with the title that matches the query (‘Node.js’) is equal to one.

If no documents match the query, Qbox ElasticSearch returns:
```javascript
{ took: 69,
  timed_out: false,
  _shards: { total: 4, successful: 4, failed: 0 },
  hits: { total: 0, max_score: null, hits: [] } }
```
Elasticsearch.js supports all Query DSL features including leaf clauses and compound clauses. As an added bonus, you can use wildcard searches and regular expressions. 
In the example below, we create a query for all matches where ‘.js’ is preceded by four characters. 
```javascript
query: { wildcard: { "PostBody": "????.js" } }
```

**Conclusion**
----------

Elasticsearch.js is a very mature Elasticsearch client for Node.js able to handle basic use cases and supporting many advanced ones. In addition to the aforementioned functionality, elasticsearch.js supports searching of shards, scrolling, bulk operations in a single API call and more. Broad coverage of low-level Elasticsearch functions and leveraging the power of Javascript Promises makes this library the best choice for your Node.js application. 


