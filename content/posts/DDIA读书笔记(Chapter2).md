---
title: "DDIA读书笔记(Chapter2)"
date: 2021-11-19T15:48:27+08:00
draft: false
original: true
categories: 
  - 书籍
tags: 
  - 读书笔记
---

# Chapter 2 Data Models and Query Languages

## **Relational Model Versus Document Model**

### **The Birth of NoSQL**

driving forces behind the adoption of NoSQL databases:

- A need for greater scalability than relational databases can easily achieve, includ‐
ing very large datasets or very high write throughput
- A widespread preference for free and open source software over commercial
database products
- Specialized query operations that are not well supported by the relational model
- Frustration with the restrictiveness of relational schemas, and a desire for a more
    
> dynamic and expressive data model
    
<!--more-->

### **The Object-Relational Mismatch**

Object-relational mapping (ORM) frameworks

There is a one-to-many relationship from the user to these items, which can be represented in various ways:

- separate tables
- added support for structured datatypes and XML data
- encode as a JSON or XML document, store it on a text column in the database

![](/DDIA/figure2-1.png)

*Representing a LinkedIn profile as a JSON document*

```sql
{
    "user_id":251,
    "first_name":"Bill",
    "last_name":"Gates",
    "summary":"Co-chair of the Bill & Melinda Gates... Active blogger.",
    "region_id":"us:91",
    "industry_id":131,
    "photo_url":"/p/7/000/253/05b/308dd6e.jpg",
    "positions":[
        {
            "job_title":"Co-chair",
            "organization":"Bill & Melinda Gates Foundation"
        },
        {
            "job_title":"Co-founder, Chairman",
            "organization":"Microsoft"
        }
    ],
    "education":[
        {
            "school_name":"Harvard University",
            "start":1973,
            "end":1975
        },
        {
            "school_name":"Lakeside School, Seattle",
            "start":null,
            "end":null
        }
    ],
    "contact_info":{
        "blog":"http://thegatesnotes.com",
        "twitter":"http://twitter.com/BillGates"
    }
}
```

one-to-many relationships imply a tree structure in the data

![](/DDIA/figure2-2.png)

The lack of a schema is often cited as an advantage; but there are also problems with JSON.

### **Many-to-One and Many-to-Many Relationships**

*many-to-one* relationships don’t fit nicely into the document model

在one-to-many的关系中，json格式其实是通过duplication来实现的。如果讲org和school也变成实体，那么json格式就需要保存许多org和school的重复数据。如果使用ID引用来关联，那么查询的时候就需要进行join。但是文档型数据库对join的支持有限。

![](/DDIA/figure2-4.png)

The data within each dotted rectangle can be grouped into one document, but the references to organizations, schools, and other users need to be represented as references, and require joins when queried.

### **Are Document Databases Repeating History?**

Various solutions were proposed to solve the limitations of the hierarchical model.

- *relational model*
- *network model*

**The network model**

also known as *CODASYL model,* in the network model, a record could have multiple parents.

if you didn’t have a path to the data you wanted, you were in a difficult situation.

**The relational model**

the query optimizer automatically decides which parts of the query to execute in which order

**Comparison to document databases**

when it comes to representing many-to-one and many-to-many relationships, relational and document databases are not fundamentally different

### **Relational Versus Document Databases Today**

The main arguments in favor of the document data model are schema flexibility, better performance due to locality

The relational model counters by providing better support for joins, and many-to-one and many-to-many relationships.

**Which data model leads to simpler application code?**

It’s not possible to say in general which data model leads to simpler application code; it depends on the kinds of relationships that exist between data items.

**Schema flexibility in the document model**

skip

**Data locality for queries**

If your application often needs to access the entire document,there is a performance advantage to this *storage locality*.

The locality advantage only applies if you need large parts of the document at the same time.

**Convergence of document and relational databases**

If a database is able to handle document-like data and also perform relational queries on it, appli‐
cations can use the combination of features that best fits their needs.

## **Query Languages for Data**

**declarative query language(like SQL)** 

- you just specify the pattern of the data you want(what conditions the results must meet)
- hides implementation details of the database engine
- makes it possible for the database system to introduce performance improvements without requiring any changes to queries
- parallel execution

**imperative language**

- tells the computer to perform certain operations in a certain order
- hard to parallelize

### **Declarative Queries on the Web**

**css**

```sql
li.selected > p { 
	background-color: blue;
}
```

**XSL instead of CSS**

```sql
<xsl:template match="li[@class='selected']/p"> 
	<fo:block background-color="blue">
    <xsl:apply-templates/>
  </fo:block>
</xsl:template>
```

### **MapReduce Querying**

MapReduce is neither a declarative query language nor a fully imperative query API,but somewhere in between

SQL Like:

```sql
SELECT date_trunc('month', observation_timestamp) AS observation_month, 
		sum(num_animals) AS total_animals
FROM observations
WHERE family = 'Sharks' 
GROUP BY observation_month;
```

MapReduce Like:

```sql
db.observations.mapReduce( 
	function map() {
		var year = this.observationTimestamp.getFullYear(); 
		var month = this.observationTimestamp.getMonth() + 1; 
		emit(year + "-" + month, this.numAnimals);
	},
	function reduce(key, values) {
		return Array.sum(values); 
	},
  {
    query: { family: "Sharks" },
    out: "monthlySharkReport"
	} 
);
```

The moral of the story is that a NoSQL system may find itself accidentally reinventing SQL, albeit in disguise.

## **Graph-Like Data Models**

- mostly one-to-many relationships → document model
- many-to-many relationships are very common → relational model
- the connections within your data become more complex → graph

graph consists of two kinds of objects: *vertices and edges*

- *Social graphs:* Vertices are people, and edges indicate which people know each other.
- *The web graph:* Vertices are web pages, and edges indicate HTML links to other pages.
- *Road or rail networks:* Vertices are junctions, and edges represent the roads or railway lines between them.

### **Property Graphs**

In the property graph model, each vertex consists of:

- A unique identifier
- A set of outgoing edges
- A set of incoming edges
- A collection of properties (key-value pairs)

Each edge consists of:

- A unique identifier
- The vertex at which the edge starts (the *tail vertex*)
- The vertex at which the edge ends (the *head vertex*)
- A label to describe the kind of relationship between the two vertices
- A collection of properties (key-value pairs)

Some important aspects of this model are:

1. Any vertex can have an edge connecting it with any other vertex.There is no schema that restricts which kinds of things can or cannot be associated.
2. Given any vertex, you can efficiently find both its incoming and its outgoing edges, and thus *traverse* the graph
3. By using different labels for different kinds of relationships, you can store several different kinds of information in a single graph, while still maintaining a clean data model.

Those features give graphs a great deal of flexibility for data modeling.

### **The Cypher Query Language**

*Cypher* is a declarative query language for property graphs, created for the Neo4j graph database.

*A subset of the data represented as a Cypher query*

```sql
CREATE
  (NAmerica:Location {name:'North America', type:'continent'}),
  (USA:Location      {name:'United States', type:'country'  }),
  (Idaho:Location    {name:'Idaho',         type:'state'    }),
  (Lucy:Person       {name:'Lucy' }),
  (Idaho) -[:WITHIN]->  (USA)  -[:WITHIN]-> (NAmerica),
  (Lucy)  -[:BORN_IN]-> (Idaho)
```

how to express that query in Cypher

```sql
MATCH
  (person) -[:BORN_IN]->  () -[:WITHIN*0..]-> (us:Location {name:'United States'}),
	(person) -[:LIVES_IN]-> () -[:WITHIN*0..]-> (eu:Location {name:'Europe'}) 
RETURN person.name
```

### **Graph Queries in SQL**

the syntax is very clumsy in comparison to Cypher.

```sql
WITH RECURSIVE
  -- in_usa is the set of vertex IDs of all locations within the United States
	in_usa(vertex_id) AS (
		SELECT vertex_id FROM vertices WHERE properties->>'name' = 'United States'
		UNION
		SELECT edges.tail_vertex FROM edges
			JOIN in_usa ON edges.head_vertex = in_usa.vertex_id
			WHERE edges.label = 'within' 
	),
	-- in_europe is the set of vertex IDs of all locations within Europe
	in_europe(vertex_id) AS (
		SELECT vertex_id FROM vertices WHERE properties->>'name' = 'Europe'
		UNION
		SELECT edges.tail_vertex FROM edges
			JOIN in_europe ON edges.head_vertex = in_europe.vertex_id
			WHERE edges.label = 'within' 
	),
	-- born_in_usa is the set of vertex IDs of all people born in the US
	born_in_usa(vertex_id) AS (
		SELECT edges.tail_vertex FROM edges
			JOIN in_usa ON edges.head_vertex = in_usa.vertex_id
			WHERE edges.label = 'born_in' 
	),
	-- lives_in_europe is the set of vertex IDs of all people living in Europe
	lives_in_europe(vertex_id) AS ( 
		SELECT edges.tail_vertex FROM edges
			JOIN in_europe ON edges.head_vertex = in_europe.vertex_id
			WHERE edges.label = 'lives_in' 
	)
SELECT vertices.properties->>'name'
FROM vertices
-- join to find those people who were both born in the US *and* live in Europe 
JOIN born_in_usa ON vertices.vertex_id = born_in_usa.vertex_id
JOIN lives_in_europe ON vertices.vertex_id = lives_in_europe.vertex_id;
```

that just shows that different data models are designed to satisfy different use cases.

### **Triple-Stores and SPARQL**

The triple-store model is mostly equivalent to the property graph model, using different words to describe the same ideas.

In a triple-store, all information is stored in the form of very simple three-part statements: (*subject*, *predicate*, *object*)

*A subset of the data represented as Turtle triples:*

```sql
@prefix : <urn:example:>.
_:lucy     a       :Person.
_:lucy     :name   "Lucy".
_:lucy     :bornIn _:idaho.
_:idaho    a       :Location.
_:idaho    :name   "Idaho".
_:idaho    :type   "state".
_:idaho.   :within _:usa.
_:usa.     a       :Location.
_:usa      :name   "United States".
_:usa      :type   "country".
_:usa      :within _:namerica.
_:namerica a.       :Location.
_:namerica :name.   "North America".
_:namerica :type.   "continent".
```

*A more concise way of writing the data*

```sql
@prefix : <urn:example:>.
_:lucy.    a :Person;   :name "Lucy";          :bornIn _:idaho.
_:idaho.   a :Location; :name "Idaho";         :type "state";   :within _:usa.
_:usa.     a :Location; :name "United States"; :type "country"; :within _:namerica
_:namerica a :Location; :name "North America"; :type "continent".
```

**The semantic web**

skip

**The RDF data model**

skip

**The SPARQL query language**

*SPARQL* is a query language for triple-stores using the RDF data model

### **Graph Databases Compared to the Network Model**

They differ in several important ways:

- In CODASYL, a database had a schema that specified which record type could be nested within which other record type. In a graph database, there is no such restriction
- In CODASYL, the only way to reach a particular record was to traverse one of the access paths to it. In a graph database, you can refer directly to any vertex by its unique ID, or you can use an index to find vertices with a particular value.
- In CODASYL, the children of a record were an ordered set, so the database had to maintain that ordering. In a graph database, vertices and edges are not ordered.
- In CODASYL, all queries were imperative, difficult to write and easily broken by changes in the schema. In a graph database, you can write your traversal in imperative code if you want to.

### **The Foundation: Datalog**

*Datalog* is less well known among software engineers, it provides the foundation that later query languages build upon

Datalog’s data model is similar to the triple-store model, generalized a bit. Instead of writing a triple as (*subject*, *predicate*, *object*), we write it as *predicate*(*subject*, *object*).

*A subset of the data represented as Datalog facts*

```sql
name(namerica, 'North America').
type(namerica, continent).
name(usa, 'United States').
type(usa, country).
within(usa, namerica).
name(idaho, 'Idaho').
type(idaho, state).
within(idaho, usa).
name(lucy, 'Lucy').
born_in(lucy, idaho).
```

*The same query expressed in Datalog*

```sql
within_recursive(Location, Name) :- name(Location, Name). /* Rule 1 */
within_recursive(Location, Name) :- within(Location, Via), /* Rule 2 */ 
																		within_recursive(Via, Name).
migrated(Name, BornIn, LivingIn) :- name(Person, Name), /* Rule 3 */ 
																		born_in(Person, BornLoc),
                                    within_recursive(BornLoc, BornIn),
                                    lives_in(Person, LivingLoc),
                                    within_recursive(LivingLoc, LivingIn).
?- migrated(Who, 'United States', 'Europe'). 
/* Who = 'Lucy'. */
```

## **Summary**

All three models (document, relational, and graph) are widely used today, and each is good in its respective domain. One model can be emulated in terms of another model—for example, graph data can be represented in a relational database—but the result is often awkward. That’s why we have different systems for different purposes, not a single one-size-fits-all solution.
