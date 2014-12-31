---
layout: post
title: Cross-type joins in Elasticsearch
category: posts
comments: true
description: An example of using parent-child relationship in Elasticsearch for doing cross-type "joins".  
---
When modeling data in Elasticsearch, a common question is how to design the data to capture relationships between entities, to allow at least some level of "joins".

Elasticsearch has a good guide about [data modeling](http://www.elasticsearch.org/guide/en/elasticsearch/guide/current/modeling-your-data.html). One of the options provided for expressing relationships is the [parent-child](http://www.elasticsearch.org/guide/en/elasticsearch/guide/current/parent-child.html) model.

A parent-child relationship in Elasticsearch is a way to express a *one-to-many* relationship (a parent with many children). The parent and child are *separate* Elasticsearch types, bounded only by specifying the parent type on the child mapping, and by giving the parent ID for every child index operation (this is used for routing the child to the shard of the parent). 

It's a useful model when a parent has many children and when the child update pattern is different from that of the parent. (Since every child is a separate document, updating the child does not require re-indexing the parent).

But this model also provides an interesting (if limited) way to capture relationships between *sibling* types.  

Lets consider the following data: 

![My helpful screenshot](/images/es_joins.jpg)

Bill has two children - Adam and Eve, and a Dog (Apple).   
Bob has no children or pets (ah, freedom!).   
Mary has a little newborn child called Lamb.   
Jane has a boy named Xander, a cat (Buffy) and a dog (Willow).

Lets create this data in Elasticsearch.   
We will have a parent type - "person", and two child types - "children" and "pets".   
First we'll create the mapping for the child types.

```bash
    #!/bin/bash
    
    export ELASTICSEARCH_ENDPOINT="http://localhost:9200"
    
    # Create indexes
    
    curl -XPUT "$ELASTICSEARCH_ENDPOINT/es-joins" -d '{
        "mappings": {
            "children": {
                "_parent": {
                    "type": "person"
                }
            },
            "pets": {
                "_parent": {
                    "type": "person"
                }
            }
        }
    }' 
```

Next, index all the documents - parents, children and pets.

```bash
    # Index documents
    curl -XPOST "$ELASTICSEARCH_ENDPOINT/_bulk?refresh=true" -d '
    {"index":{"_index":"es-joins","_type":"person","_id":1}}
    {"name":"Bill","gender":"male"}
    {"index":{"_index":"es-joins","_type":"person","_id":2}}
    {"name":"Bob","gender":"male"}
    {"index":{"_index":"es-joins","_type":"person","_id":3}}
    {"name":"Mary","gender":"female"}
    {"index":{"_index":"es-joins","_type":"person","_id":4}}
    {"name":"Jane","gender":"female"}
    {"index":{"_index":"es-joins","_type":"children","_parent":1,"_id":1}}
    {"name":"Adam","gender":"male"}
    {"index":{"_index":"es-joins","_type":"children","_parent":1,"_id":2}}
    {"name":"Eve","gender":"female"}
    {"index":{"_index":"es-joins","_type":"children","_parent":3,"_id":3}}
    {"name":"Lamb","gender":"male"}
    {"index":{"_index":"es-joins","_type":"children","_parent":4,"_id":4}}
    {"name":"Xander","gender":"male"}
    {"index":{"_index":"es-joins","_type":"pets","_parent":1,"_id":1}}
    {"name":"Apple","type":"dog"}
    {"index":{"_index":"es-joins","_type":"pets","_parent":4,"_id":2}}
    {"name":"Buffy","type":"cat"}
    {"index":{"_index":"es-joins","_type":"pets","_parent":4,"_id":3}}
    {"name":"Willow","type":"dog"}
    '
```

Now we can do some searches on it.    
The usual example will be searching a parent by its children. Lets find all the parents that has a girl. We expect to get back only Bill. 

```bash
    curl -XPOST "$ELASTICSEARCH_ENDPOINT/es-joins/person/_search?pretty" -d '
    {
        "query": {
            "filtered": {
                "filter": {
                    "and": [
                        {
                            "has_child": {
                                "type": "children",
                                "query": {
                                    "term": {
                                        "gender": "female"
                                    }
                                }
                            }
                        }
                    ]
                }
            }
        }
    }
    '
```

We can also combine conditions on multiple child types.    
Lets find parents that have a boy and a dog. This time we expect to get back both Bill and Jane. 

```bash 
    curl -XPOST "$ELASTICSEARCH_ENDPOINT/es-joins/person/_search?pretty" -d '
    {
        "query": {
            "filtered": {
                "filter": {
                    "and": [
                        {
                            "has_child": {
                                "type": "children",
                                "query": {
                                    "term": {
                                        "gender": "male"
                                    }
                                }
                            }
                        },
                        {
                            "has_child": {
                                "type": "pets",
                                "query": {
                                    "term": {
                                        "type": "dog"
                                    }
                                }
                            }
                        }
                    ]
                }
            }
        }
    }
    '
```

Another commonly used option is finding [children by their parents](http://www.elasticsearch.org/guide/en/elasticsearch/guide/current/has-parent.html).   
But a more interesting possibility is finding children _**by their siblings**_.   
Lets lookup all boys that have a dog. To do that we're searching on the "children" type, and doing a has\_parent filter that contains a has\_child filter on the "pets" type.   
This time we expect to get back the children - Adam and Xander.

```bash 
    curl -XPOST "$ELASTICSEARCH_ENDPOINT/es-joins/children/_search?pretty" -d '
    {
        "query": {
            "filtered": {
                "filter": {
                    "and": [
                        {
                            "has_parent": {
                                "parent_type": "person",
                                "filter": {
                                    "has_child": {
                                        "type": "pets",
                                        "query": {
                                            "term": {
                                                "type": "dog"
                                            }
                                        }
                                    }
                                }
                            }
                        },
                        {
                            "term": {
                                "gender": "male"
                            }
                        }
                    ]
                }
            }
        }
    }
    '
```

Of course, our data model here is a bit simplified as it allows only a single parent. If we were to extend it, we would create a "family" parent type, with child types - "parents", "children" and "pets".

Currently, in order to get the details of the "joined" entity, another query is needed. For example, when searching "all boys that have a dog", if we want the details of the dogs we need a second search for "all dogs with parents that have children with \_id=..." (and the \_ids of the children from the first search).   
This will change with the new upcoming [inner hits](http://www.elasticsearch.org/guide/en/elasticsearch/reference/1.x/search-request-inner-hits.html) feature that will allow getting the data of the inner entities in a single query. 

One should note that this method is not exactly recommended by Elasticsearch. Because of the memory requirements and performance hit, the [official recommendation](http://www.elasticsearch.org/guide/en/elasticsearch/guide/current/parent-child-performance.html) is: "Avoid using multiple parent-child joins in a single query". So as always, test, measure and choose your modeling wisely.