
### CREATE (Create a node)
```
CREATE (p:Person);
CREATE (p:Person 
            {
                    name: "Manjit",
                    age: 22
            }
        );
```
* `CREATE` clause to create data.
* `()` parenthesis to indicate a node.
* `p:Person` a variable `p` and label `Person` for new node.
* `{}` brackets to add properties to the node.

#### CREATE (Nodes and relationship)

> `CREATE` clause can create many nodes and relationships at once.  `CREATE` or `MERGE` for writing data.
```
CREATE (mn:Person 
            {
            name: "Manjit",
            address: "Sundhara",
            learn: "Gaming"
            }
        ),
        (sd:Person
            {
            name: "Sandesh",
            address: "Sunakothi",
            hobby: "Dancing"
            }
        ),
        (kl:Person
            {
            name: "Kushal",
            address: "Kirtipur",
            hobby: "Singing" 
            }
        ),
        (rl:Person
            {
            name: "Rahil",
            address: "Kanibahal",
            hobby: "Gaming" 
            }
        ),
        (mn)-[:KNOWS {since: 2017}]->(sd),
        (mn)-[:KNOWS {rating: 5}]-> (kl),
        (kl)-[:KNOWS]->(rl),
        (kl)-[:KNOWS]->(sd),
        (sd)-[:KNOWS]->(rl);
```
* `[:KNOWS]` is a relationship type.
* `[:KNOWS {since: 2017}]` is relationship properties.
* `->` determines relationship direction.

### MATCH (Finding nodes)
> `MATCH` clause is for reading data. The `MATCH` clause describes a pattern of graph data. Neo4j will collect all paths within the graph which match this pattern.
```
MATCH (p:Person) WHERE p.name = "Manjit" RETURN p;
```
* `MATCH` clause to specify a pattern of nodes and relationships.
* `(p:Person)` a single node pattern with label `Person` which will assign matches to variable `p`.
* `WHERE` clause to constrain the results.
* `p.name = "Manjit"` compares name property to the value "Manjit"
* `RETURN` clause used to request particular results.

```
MATCH (p:Person)-[:KNOWS]-(friends) WHERE p.name = "Manjit" RETURN p, friends;
```
* `(p:Person)-[:KNOWS]-(friends)` is the pattern.
* `-[:KNOWS]-` matches  "KNOWS" relationship in either direction.
* `(friends)` will be bound to Manjit's friends.

> Pattern matching can be used to make recommendation. Manjit is learning to play games, so he may want to find a new friend who already does.

```
MATCH (mn:Person)-[:KNOWS]-()-[:KNOWS]-(gamer) WHERE mn.name = "Manjit" AND gamer.hobby = "Gaming" RETURN DISTINCT gamer;
```
* `()` empty parenthesis to ignore these nodes.
* `DISTINCT` because more than one path will match the pattern.
* `gamer` will contain Rahil, a friend of a friend who is a gamer

### SET
> The `SET` clause updates properties on nodes and relationships

```
MATCH (p:Person)-[w:LOVES]->(girl) WHERE p.name="Sandesh"
SET w.relationship = "2 years";
```

### WHERE
> The `WHERE` clause imposes conditions on the data within a potentially matching path, filtering the result set of a `MATCH`. `MATCH` describes the structure, and WHERE specifies the content of a query.

```
MATCH (p:Person)-[:CODES]->(language)
WHERE p.name = "Sandesh"
RETURN language.name;
```

### RETURN
> The `RETURN` clause defines what to include in a query result set, specified as a comma separated list of expressions.

```
MATCH (p:Person)-[:CODES]->(language)
RETURN p.name AS name, collect(language.name) as Languages;
```

### DELETE
> The `DELETE` clause is used to delete nodes and relationships identified within a `MATCH` clause, possibly qualified by a `WHERE`. Remember that you can not delete a node without also deleting relationships that start or end on said node.

```
MATCH (n)-[r]-() WHERE n.name = "Manjit" DELETE r;
```
> The query above delete all the relationships  with Manjit's friend's.

### ORDER BY
> By adding DESC or ASC after the variable to sort on, the sort will be done in reverse order for DESC and Serial format for ASC (Ascending order).
```
MATCH (n)
RETURN n.name, n.age
ORDER BY n.name DESC
```

### SKIP AND LIMIT
> By using `SKIP`, the result set will get trimmed based on value.
```
MATCH (n)
RETURN n.name
ORDER BY n.name
SKIP 1
LIMIT 2
```


### DETACH DELETE
> The `DETACH DELETE` clause is used to delete all nodes and relationships.
```
MATCH (n) DETACH DELETE n
```

### FOREACH
>The `FOREACH` clause is used to update data within a collection.
```
MATCH p = (ups)<-[DEPENDS_ON]-(device) WHERE ups.id='EPS-7001'
FOREACH (n IN nodes(p) | SET n.available = FALSE )
``` 

### WITH
>The `WITH` clause allows queries to be chained together, piping the results from one to be used as starting points or criteria in the next.

> `WITH` allows you to pass on data from one part of the query to the next. Whatever you list in `WITH` will be available in the next query part.

> That means you can chain query parts where one computes some data and the next query part can use that computed data. In your case it is what GROUP BY and HAVING would be in SQL but `WITH` is much more powerful than that.

```
MATCH (n)
WITH n
ORDER BY n.name DESC LIMIT 3
RETURN collect(n.name)
```

```
MATCH (david { name: 'David' })--(otherPerson)-->()
WITH otherPerson, count(*) AS foaf
WHERE foaf > 1
RETURN otherPerson.name
```

### LOAD CSV
> `LOAD CSV` clause loads csv from file located at given URL
```
LOAD CSV FROM "http://data.neo4j.com/examples/person.csv" AS line
MERGE (n:Person {id: toInteger(line[0])})
SET n.name = line[1]
RETURN n
```

### CREATE INDEX ON
> A database index is a redundant copy of information in the database for the purpose of making retrieving said data more efficient. This comes at the cost of additional storage space and slower writes, so deciding what to index and what not to index is an important and often non-trivial task. The `CREATE INDEX ON` clause will create and populate an index on a property for all nodes that have a label.

> Two types single-property index and composite index.
```
CREATE INDEX ON :Person(name);
CREATE INDEX ON :Person(firstname,lastname);
```
> DROP index

```
DROP INDEX ON :Person(name);
DROP INDEX ON :Person(firstname,lastname);
```
> List all indexes

```
CALL db.indexes;
CALL db.indexes YIELD description, label, properties;
```

### SET OPERATIONS

#### UNION
> Combines the result of multiple queries into a single result set. Duplicates are removed.

```
MATCH (n:Actor)
RETURN n.name AS name
UNION
MATCH (n:Movie)
RETURN n.title AS name
```

#### UNION ALL
> Combines the result of multiple queries into a single result set. Duplicates are retained.
```
MATCH (n:Actor)
RETURN n.name AS name
UNION ALL MATCH (n:Movie)
RETURN n.title AS name
```

### CONSTRAINTS
#### Unique
> Constraints are used to make sure that the data adheres to the rules of the domain. For example: "If a node has a label of Actor and a property of name, then the value of name must be unique among all nodes that have the Actor label".

```
CREATE CONSTRAINT ON (book:Book) ASSERT book.isbn IS UNIQUE
```

> Drop constraints

```
DROP CONSTRAINT ON (book:Book) ASSERT book.isbn IS UNIQUE
```
#### Node property existence

> To create a constraint that ensures that all nodes with a certain label have a certain property, use the `ASSERT exists(variable.propertyName)` syntax.

```
CREATE CONSTRAINT ON (book:Book) ASSERT exists(book.isbn)
```
> After enforcing this constraint `CREATE (book:Book { isbn: '1449356265', title: 'Graph Databases' })` will be executed but `CREATE (book:Book { title: 'Graph Databases' })` will not execute and return error message "Node(0) with label `Book` must have the property `isbn`"


> drop existence constraint

```
DROP CONSTRAINT ON (book:Book) ASSERT exists(book.isbn)
```
#### Relationship property existence

> To create a constraint that makes sure that all relationships with a certain type have a certain property, use the `ASSERT exists(variable.propertyName)` syntax.

```
CREATE CONSTRAINT ON ()-[like:LIKED]-() ASSERT exists(like.day)
```

> Above query will enforce the relationship constraint where `DROP CONSTRAINT ON ()-[like:LIKED]-() ASSERT exists(like.day)` is fired successfully but this `CREATE (user:User)-[like:LIKED]->(book:Book)` will give error "Relationship(0) with type `LIKED` must have the property `day`".

#### Node Key

> To create a Node Key ensuring that all nodes with a particular label have a set of defined properties whose combined value is unique, and where all properties in the set are present, use the `ASSERT (variable.propertyName_1, …​, variable.propertyName_n)` IS NODE KEY syntax.

```
CREATE CONSTRAINT ON (n:Person) ASSERT (n.firstname, n.surname) IS NODE KEY
```
> Drop

```
DROP CONSTRAINT ON (n:Person) ASSERT (n.firstname, n.surname) IS NODE KEY
```

> `DROP CONSTRAINT ON (n:Person) ASSERT (n.firstname, n.surname) IS NODE KEY` will fire and `CREATE (p:Person { firstname: 'Jane', age: 34 })` will fail and give error "Node(0) with label `Person` must have the properties `firstname, surname`"

> We can inspect our database to find out what constraints are defined. 

```
CALL db.constraints
```

### STARTS WITH
> Matching the start of a string. The start of strings can be matched using `STARTS WITH`. The matching is case-sensitive.
```
MATCH (p:Person)
WHERE p.name STARTS WITH 'Man'
RETURN p.name
```

### ENDS WITH
> The end of strings can be matched using `ENDS WITH`. The matching is case-sensitive.

```
MATCH (p:Person)
WHERE p.name ENDS WITH 'jit'
RETURN p.name
```

### CONTAINS
> The occurrence of a string within a string can be matched using `CONTAINS`. The matching is case-sensitive.

```
MATCH (p:Person)
WHERE p.name CONTAINS 'anj'
RETURN p.name
```

### ANALYZE (Using the visual query plan)
> Understand how your query works by prepending `EXPLAIN` or `PROFILE`
```
PROFILE MATCH (mn:Person)-[:KNOWS]-()-[:KNOWS]-(gamer) WHERE mn.name = "Manjit" AND gamer.hobby = "Gaming" RETURN DISTINCT gamer;
```

Learn about queries using
```
:help cypher
```

### Facts

1. A graph database can store any kind of data using a few simple concepts:
1. The simplest graph has just a single node with some named values called Properties.
1. Nodes can be grouped together by applying a Label to each member. Here, the label for grouping is Person, as both of them are Person type. In relational database their are Tables which store specific values instead of Labels. Based on JSON:
    
    ```
    [
        Person{
            name:"Manjit Shakya"
            age: 22
        },
        Person{
            name:"Sandesh Maharjan"
            age: 21
        }
    ]
    ```

1. A node can have zero or more labels
1. Labels do not have any properties
1. Similar bodes can have different properties
1. Properties can be strings, numbers, or booleans
1. Neo4j can store billions of nodes.
1. The real power of Neo4j is in connected data. To associate any two nodes, add a Relationship which describes how the records are related.
1. Relationships always have direction.
1. Relationships always have a type.
1. Relationships form patterns of data.
