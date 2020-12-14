---
title: 'Combining properites from different nodes in Gremlin'
date: Sun, 13 Dec 2020 15:14:10 -0800
draft: false
tags: ['tinkerpop', 'gremlin']
---

One of the key differences between SQL databases and graph databases is the concept of 
joining information from different nodes.
In a tinkerpop-enabled graph database nodes have labels that define their type and properties
that are part of that type. It's natural to draw the comparison to a label being an SQL table
and the properties of the nodes being the columns of that table. But trying to extend that 
analogy to compare joining in SQL is a little murky. Insead of joining two tables together, 
it's better to think about visiting different nodes and
saving their information for later use. In gremlin you can achieve this in a few differnt ways
but the most common is to use the `as`-step. The example below shows how to combine information
using the "Modern" graph that is shipped with tinkerpop. Here there are two types of nodes, 
person and software; each have a "name" property. If you want a mapping of which people created
which software you could do:

```groovy
g.V().hasLabel('person').as('s').
    out('created').hasLabel('software').
    project('creator_name', 'software_name').
        by(select('s').values('name')).
        by('name')
```

In the example above the traversal starts with all of the people and saves the information
in the variable, `s`, that's what the `as('s')` is doing. It then travels to the
software nodes, using `out('created')`. To get the information from both the software
and the person I use the `project`-step, which creates a new structure. In this case it will be 
a mapping with `creator_name` and `software_name` as the keys. To get the creator name I use 
the `select`-step that recalls the information that was stored previously using the variable, `s`.
The traversal above will produce a pair-wise mapping like so:

```
{creator_name=marko, software_name="lop"}
{creator_name=peter, software_name="lop"}
{creator_name=josh, software_name="lop"}
```

The other method of combining data is to use the `group`-step. In this case the 
returned structure will have a mapping based on how you group:

1. with the person as the key
    ```groovy

    g.V().hasLabel('person').group().
        by('name').
        by(out('created').hasLabel('software').values('name').fold())
    ```

2. or with the software name as the key
    ```groovy
    g.V().hasLabel('software').group().
        by('name').
        by(__.in('created').hasLabel('person').values('name').fold()).toList()
    ```

The first query gives you a dictionary where the persons name is the key and the values are
 a list of software names created by that person, like the following:
 ```
{
    marko: [lop],
    josh: [lop, ripple],
    peter: [lop]
}
 ``` 
The second version turns it around so that the software is the key and the creators are
the list of values:
```
{
    lop: [marko, peter, josh],
    ripple: [josh]
}
```

Whether to use the query based on the `as`-step or the `gorup`-step is dependant on 
the structure that you want at the end and the performance. In my experience, grouping 
takes longer than using the `as`&ndash;`project` method
and is more flexible too. You can save multiple steps in your traversal with different
variable names and recall them at the appropriate time.  