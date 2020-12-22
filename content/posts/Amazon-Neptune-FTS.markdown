---
title: 'Using Amazon Neptune full text search'
date: Sun, 23 Aug 2020 22:14:10 -0700
draft: false
tags: ['Neptune', 'AWS', 'full text search', 'elasticsearch', 'tinkerpop', 'gremlin']
---

I've been trying out [Amazon Neptune's](https://aws.amazon.com/neptune/) 
[full text search](https://docs.aws.amazon.com/neptune/latest/userguide/full-text-search.html)
feature. Overall it's been a great experience although there are a few caveats when
searching that means that you'll have to craft your queries carefully to make full use
of the feature.

The tinkerpop standard has [some text searching features](http://tinkerpop.apache.org/docs/current/reference/#a-note-on-predicates)
however it lacks any advanced features such as searching using regular expressions or even
case-insensitive searching. It's left to different implementations to augment this text
searching capability. 

The developers at Amazon Neptune chose to integrate [Elasticsearch](https://www.elastic.co) as 
their text searching engine, which offers a rich searching ability. 


The integration between Elasticsearch and Neptune is pretty seamless with clear documentation.
I used AWS's managed elastic search service and I'm not sure if an external elasticsearch 
setup could work. I already had a Neptune database however to work with elastic search I needed
to turn on the [streams feature](https://docs.aws.amazon.com/neptune/latest/userguide/streams.html)
so that data could be replicated from one service to another. After turning on the streams feature
I had to manually reboot my Neptune instances for it to take effect. With the streams feature
turned on I used the [export to elasticsearch](https://github.com/awslabs/amazon-neptune-tools/tree/master/export-neptune-to-elasticsearch) 
cloudformation template to mirror the data into elastic search. This is a one time operation 
as long as the you also use the [second cloudformation template](https://docs.aws.amazon.com/neptune/latest/userguide/full-text-search-cfn-create.html)
that uses the Neptune stream to constantly update the elasticsearch index when changes are made
to the neptune database.

Using the feature with gremlin also works great. When you want to run a query using full text search
you need to add a `withSideEffect` step at the begining of the traversal

```groovy
g.withSideEffect("Neptune#fts.endpoint", "<ENDPOINT_URL>")
```
And then after that you can use full text searching in a `has` step. The AWS docs have some
[good examples](https://docs.aws.amazon.com/neptune/latest/userguide/full-text-search-gremlin.html)
that show most of the features. Below are a couple of additional observations I made.

Full text searching is overloaded in the `has`-step. You by default you can search in a single property
by using the form `has("<property_key>", "Neptune#fts <query_string>"`.

The following searches for all nodes in the graph that have `foobar` in their `name` property.

```groovy
g.withSideEffect("Neptune#fts.endpoint", "<ENDPOINT_URL>").
    V().
    has('name', 'Neptune#fts foobar').
    valueMap()
```

But you don't have to specify a property. Using a `*` character in place of the property key
allows you to search for the term in all of the properties of the nodes.

```groovy
g.withSideEffect("Neptune#fts.endpoint", "<ENDPOINT_URL>").
    V().
    has('*', 'Neptune#fts foobar').
    valueMap()
```

Using this simple method is an all or one approach, you can't restrict the search to multiple, known
properties. To get this functionality you need to be a little more low-level and use the Lucene syntax,
which the [docs has some examples of near the end](https://docs.aws.amazon.com/neptune/latest/userguide/full-text-search-gremlin.html).
To make use of this you'll need to know a bit about [how Neptune data is translated into elasticsearch documents](https://docs.aws.amazon.com/neptune/latest/userguide/full-text-search-model.html)
and also read up on the [query string](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html)
syntax of elastic search. 

Below the `has`-step makes use of a full text search looking at both the value of the `name` property
and the `other_name` property. Due to the way Neptune data is translated into elasticsearch documents
we need to refer to them as `predicates.name.value` and `predicates.other_name.value`

```groovy
// note that there is an extra query hint to Neptune to use the query_string syntax for full text search
g.withSideEffect("Neptune#fts.endpoint", "<ENDPOINT_URL>").
    withSideEffect("Neptune#fts.queryType", "query_string").
    V().
    has('*', 'Neptune#fts predicates.name.value:foo* OR predicates.other_name.value:bar~').
    valueMap()
```

When looking at the query above you could try to formulate it in a more "gremlin" way by 
having the separate search terms inside a `union`-step, like below.

```groovy
g.withSideEffect("Neptune#fts.endpoint", "<ENDPOINT_URL>").
    V().
    union(
        has('name', 'Neptune#fts foo*'),
        has('other_name', 'Neptune#fts bar~')
    ).
    valueMap()
```

However in my tests, I've found this to be very slow in comparison. Although I don't know
why that is we do have to remember that these queries are using two different engines and 
any full text search traversal has to go from Neptune through elasticsearch's API and back again; 
so it's probably best to make use of as much as the elasticsearch query language can offer
so the two services have to interact the least amount.