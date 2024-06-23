---
layout: distill
title: Using Advanced Elasticsearch Methods With Node.js Elasticsearch Client
description: 
tags: elasticsearch nodejs
giscus_comments: true
date: 2019-12-12
featured: true

authors:
  - name: Kirill Goltsman
    url: "https://en.wikipedia.org/wiki/Albert_Einstein"
    affiliations:
      name: IAS, Princeton

bibliography: 2018-12-22-distill.bib

# Optionally, you can add a table of contents to your post.
# NOTES:
#   - make sure that TOC names match the actual section names
#     for hyperlinks within the post to work correctly.
#   - we may want to automate TOC generation in the future using
#     jekyll-toc plugin (https://github.com/toshimaru/jekyll-toc).
toc:
  - name: Scrolling
    # if a section has subsections, you can add them as follows:
    # subsections:
    #   - name: Example Child Subsection 1
    #   - name: Example Child Subsection 2
  - name: Aggregations
  - name: Language Analyzers

# Below is an example of injecting additional post-specific styles.
# If you use this post as a template, delete this _styles block.
_styles: >
  .fake-img {
    background: #bbb;
    border: 1px solid rgba(0, 0, 0, 0.1);
    box-shadow: 0 0px 4px rgba(0, 0, 0, 0.1);
    margin-bottom: 12px;
  }
  .fake-img p {
    font-family: monospace;
    color: white;
    text-align: left;
    margin: 12px 0;
    text-align: center;
    font-size: 16px;
  }
---

In the previous [tutorial](https://qbox.io/blog/integrating-elasticsearch-into-node-js-application), we have discussed how to use elasticsearch.js, the official Node.js client for Elasticsearch, to index, add documents, and search them using simple queries and Query DSL. In this tutorial, we're going to dive deeper into elasticsearch.js describing more advanced methods and concepts like scrolling, aggregations, and analyzers. 

As always, we will be using hosted Elasticsearch on Qbox.io. We assume that you have installed the latest version of Node.js, downloaded the elasticsearch.js module into your Node.js application and connected it to your Elasticsearch cluster as described in the previous tutorial. 

### **Scrolling **

Search requests discussed in the [previous tutorial](https://qbox.io/blog/integrating-elasticsearch-into-node-js-application) are convenient if you want to retrieve a single ‘*page*’ of results. However, what if you need to return a large number of results from a single request? Then, you would probably need a cursor implemented in the Elasticsearch as a Scroll API. Keep in mind that scrolling is not intended for the real-time user requests (normally, you would paginate the content for users), but rather for batch processing of a large amount of data or re-indexing your content. 

Below is an example of how we can use scrolling to retrieve all blog posts featuring 'Node' in their title and save them to the **allTitles** array for later processing. 



```javascript
var allTitles = [];
client.search({
            index: 'posts',
            scroll: '10s', // keep the search results "scrollable" for 10 seconds
            source: ['title'], // filter the source to only include the title field
            q: 'title:Node'
        }, function getMoreUntilDone(error, response) {
            // collect the title from each response
            response.hits.hits.forEach(function(hit) {
                allTitles.push(hit._source.title);
            });
```



Here, we are using a scroll parameter with a value **‘10s’**, which tells Elasticsearch how long it should keep the “search context”. This value should not be big enough because it is used only to process the previous batch of results, not the entire set of results.  Also, the callback function **getMoreUntilDone** returns a response object with **_scroll_id** which indicates the id of a current scroll.  This id should then be passed to the Scroll API in order to retrieve the next batch of results. The **_scroll_id** looks like this:

```javascript
POST /_search/scroll 
{
    "scroll" : "10s", 
    "scroll_id" : "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAD4WYm9laVYtZndUQlNsdDcwakFMNjU1QQ==" 
}
```

After we get **_scroll_id**, we can continue with retrieving our posts. 

```javascript
if (response.hits.total > allTitles.length) {
    //ask elasticsearch for the next set of hits from this search
    client.scroll({
        scrollId: response._scroll_id,
        scroll: '10s'
    }, getMoreUntilDone);
} else {
    console.log('every Node title', allTitles);
}
});
```

What this code actually does is specifying a scroll id for the next batch of pasts (and because **_scroll_id** changes with each request, we need to specify the newest one) and recursively calling **getMoreUntilDone** function as long as the total number of hits in our response is greater than the length of the **allTitles ** array. 

### Aggregations

Aggregations are useful whenever you want to build analytic and quantitative information over a set of documents. Elasticsearch.js supports key types of Elasticsearch aggregations including metrics aggregations for computing numerical parameters of your documents (e.g average value, max/min value or percentile ranks over a set of documents), bucketing aggregations for defining sets of documents associated with certain criterion, matrix aggregations operating on multiple fields and producing matrix result based on values extracted from those, and pipeline aggregations that aggregate the output of other aggregations and their associated metrics. 

To understand how aggregations work in elasticsearch.js, let’s address the case of metrics aggregations. Assuming we have an index named **‘grades’**  where we store student grades in specific subjects (e.g math), our task is to calculate the average grade and max/min grades earned by students in our collection. To accomplish this, we first need to define the aggregation parameters in our search query. 

A typical structure of aggregations definition looks like this:

```javascript
"aggregations" : {
    "<aggregation_name>" : {
        "<aggregation_type>" : {
            <aggregation_body>
        }
        [,"meta" : {  [<meta_data_body>] } ]?
        [,"aggregations" : { [<sub_aggregation>]+ } ]?
    }
    [,"<aggregation_name_2>" : { ... } ]*
}
```

Using elasticsearch.js, we can define aggregations as the object within search request body:

```javascript
client.search({
    index: 'grades',
    body: {
        "aggs": {
            "avg_grade": {
                "avg": {
                    "field": "grade"
                }
            }
        }
    }
}).then(function(resp) {
    console.log(resp);
}, function(err) {
    console.trace(err.message);
});
```

In this code, **avg_grade** is a variable we define to store the result of our average grade computation and a 'field' property specifies the field that should be aggregated. In our case, this is a 'grade' field that stores student grades.

If the request is successful, we get a response that looks like this:

```javascript
aggregations:{ avg_grade: { value: 70.33333333333333 } } }
```

We can include as many aggregation types in our **aggs** object as we want. For example, in the code snippet below we simultaneously calculate the average, min and max values of our grades.

```javascript
client.search({
            index: 'grades',
            body: {
                "aggs": {
                    "avg_grade":
                    {
                        "avg": {
                            "field": "grade"
                        }
                    },
                    "min_grade": {
                        "min": {
                            "field": "grade"
                        }
                    },
                    "max_grade": {
                        "max": {
                            "field": "grade"
                        }
                    }
                }
            }
```



If the response is successful, we get the following results:

```javascript
  { max_grade: { value: 94 },
    min_grade:{ value: 43 },
    avg_grade:{ value: 70.33333333333333 } } }
```

We can also use cardinality aggregation to calculate a number of unique entities in our collection. For example, if you are indexing musical albums you can count unique bands that match the user query.

```javascript
{
    "aggs" : {
        "band_count" : {
            "cardinality" : {
                "field" : "band"
            }
        }
    }
}
```

The successful response below means that there are 3 unique bands in our albums collection. 

```javascript
{
    "aggregations" : {
        "type_count" : {
            "value" : 3
        }
    }
}
```



In addition, Metrics aggregations support Geo points and their centroid calculation, percentiles, the sum of numeric values extracted from aggregated documents and other useful methods. 

### Language Analyzers

Language analyzers convert text, like a musical album’s title into tokens or terms which are added to the inverted index for searching. This index will include not only exact words but plural word forms, infinitive verbs etc. This conversion makes it easy to match user queries (which might not be exactly correct) with documents in your collection.

For example, the in-built *english* analyzer will transform 

```
“the QUICK brown foxes jumped over the lazy dog!”
```

into the following inverted index:

```
[quick, brown, fox, jump, over, lazy, dog]
```

To set a language analyzer to *english* we should specify a mapping for a searchable property of our document. Let’s assume that we have created an index named “albums” to store musical albums. Then, we can insert a new mapping with *english* analyzer for a property “title” of our document “album”. This makes our album titles searchable according to the *english* analyzer rules mentioned above. 

```javascript
client.indices.putMapping({
    index: 'albums',
    type: 'album',
    body: {
        properties: {
            'title': {
                'type': 'string', // type is a required attribute if index is specified
                'analyzer': 'english'
            },
        }
    }
}, function(err, resp, status) {
    if (err) {
        console.log(err);
    } else {
        console.log(resp);
    }
});
```



Now, if we have a Pink Floyd’s 1973 album “The Dark Side of the Moon” in our album list, users can find it using the following query. 

```javascript
client.search({
    index: 'albums',
    type: 'album',
    q: "title:moons"
}).then(function(resp) {
    console.log(resp);
}, function(err) {
    console.trace(err.message);
});
```



Even though the exact word “**moons**” used in the query string does not appear in the album’s title (it reads “The Dark side of the **moon**”), since we have applied the same analyzer both to the ‘title’ property and the query string, the query matches an inverted index and returns Pink Floyd’s album successfully. 

```
hits:{ total: 1, max_score: 0.25316024, hits: [ [Object] ] } }
```

### Conclusion 

Hopefully, you've got some intuition about using advanced Elasticsearch methods with Node.js elasticsearch.js client. Elasticsearch.js is a very powerful tool that provides one-to-one mapping with Elasticsearch REST API which means it covers the vast majority of methods used in the native Elasticsearch. Broad coverage of low-level Elasticsearch features, support for load balancers, intelligent handling of node/connection failures and easy integration into modern browsers makes elasticsearch.js the best solution for your Node.js applications. 