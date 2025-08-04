# Cypher doc
- [Doc](https://neo4j.com/docs/cypher-cheat-sheet/5/all/)

# Reset DB
```
// Delete All Nodes and Relationships
MATCH (n)
DETACH DELETE n;

// Drop All Constraints and Indexes
CALL apoc.schema.assert({}, {})
```

# Intermediate filter

- is in list
```
WHERE <value> IN <property>
```

- is null or not null
```
WHERE <property> IS NULL
WHERE <property> IS NOT NULL
```

- have label
```
WHERE <variable>:<label>
```

- exclude
```
WHERE <property> <> <value>
```

- test string
```
// RANGE INDEX
WHERE <property> STARTS WITH <value>

// TEXT INDEX
WHERE <property> ENDS WITH <value>
WHERE <property> CONTAINS WITH <value>

WHERE toLower(<property>) STARTS WITH <value>
WHERE toUpper(<property>) STARTS WITH <value>
```

- test pattern
```
// should test pattern in MATCH clause instead for performance
WHERE EXISTS {(<node_pattern>)-[<relationship_pattern>]-(<node_pattern>)}
WHERE NOT EXISTS {(<node_pattern>)-[<relationship_pattern>]-(<node_pattern>)}
```

- optional match
```
// still include row but with null
OPTIONAL MATCH <pattern>
```

- where on call
```
CALL ... YIELD <column>, ...
WITH *
WHERE ...
RETURN *
```

# Intermediate return

- return only distint value
```
RETURN DISTINCT
```

- return only properties of node
```
// all properties
RETURN <variable> { .* }

// some properties
RETURN <variable> { .<property> }
RETURN <variable> { .<property>, .<property> }
```

- order results
```
ORDER BY <property> DESC
ORDER BY <property> ASC
ORDER BY <property> ASC, <property> DESC
```

- skip/limit results
```
SKIP <value>
LIMIT <value>
SKIP <value> LIMIT <value>
```


- change value of results
```
// example `RETURN "Movie:" + m.title AS movie`
RETURN <operation> AS <column_name>
```

- condition change value of results
```
RETURN CASE
WHEN <filter> THEN <value>
ELSE <value>
END
AS <column_name>

RETURN CASE
WHEN <filter> THEN <value>
WHEN <filter> THEN <value>
ELSE <value>
END
AS <column_name>
```

- change value of results to list
```
RETURN [<property>] AS <column_name>
```

- return element in list by index
```
// return element index 0
RETURN <list_property>[0] AS <column_name>

// return slice list start at index 1
RETURN <list_property>[2 ..] AS <column_name>
```

- return new list from filter
```
RETURN [<variable> IN <property> WHERE <filter>]
AS <column_name>
```

- return pattern comprehension list
```
// like using aggregating but don't affect MATCH clause
RETURN [<pattern> WHERE <filter> | <value>]
AS <column_name>
```

- change value of results to map
```
RETURN {<key>: <value>} AS <column_name>
```

- return value in map by key
```
// return value of map key "0"
RETURN <map_property>["0"] AS <column_name>
RETURN <map_property>.0 AS <column_name>
```

- return list of key of the map
```
RETURN keys(<map_property>) AS <column_name>
```

- unwind list
```
UNWIND <property> AS <column_name>
```

# Schema

- Check performance
```
PROFILE
...

// do not execute query
EXPLAIN
...
```

- You can view the data model in the sandbox to the right by executing this query:
```
CALL db.schema.visualization()
```

- You can view the property types for nodes in the graph by executing this query:
```
CALL db.schema.nodeTypeProperties()

CALL db.schema.nodeTypeProperties() YIELD nodeType, nodeLabels, propertyName, propertyTypes, mandatory
WITH *
WHERE <label> IN nodeLabels AND propertyName = <property_name>
RETURN *
```

- You can view the property types for relationships in the graph by executing this query:
```
CALL db.schema.relTypeProperties()
```

- You can view the uniqueness constraint indexes in the graph by executing this query:
```
SHOW CONSTRAINTS
```

# Aggregating

https://neo4j.com/docs/cypher-manual/current/functions/aggregating/

- count
```
RETURN count(*) as <column_name>

// group base on property in RETURN clause
RETURN <property> count(*) AS <column_name>
```

- aggregate value to list
```
// group base on property in RETURN clause
RETURN collect(<property>) AS <column_name>

RETURN collect(DISTNCT <property>) AS <column_name>

// collect all node for later query
RETURN collect(<variable>) AS <column_name>
```

# Traversal


- match shortest pattern
```
MATCH <variable> = shortestPath((<node>)-[*]-(<node>))
```

- match node hops
```
MATCH (p:Person)-[:ACTED_IN*2]-(others:Person)
MATCH (p:Person)-[:ACTED_IN*1..6]-(others:Person)
```

# Reduce memory

- using subquery
```
CALL {
    ...
}
...

CALL {
    // passing variable
    WITH
    ...
}
...
```

- union
```
MATCH
...
UNION ALL
MATCH
...
```

## Note

- Avoid Big One Query
```
// Problem
UNWIND $rows AS row
MERGE (a:Author {name: row.author})
MERGE (b:Book {title: row.title})
MERGE (a)-[:WROTE]->(b)

// Better
UNWIND $rows AS row
CALL {
  WITH row
  MERGE (a:Author {name: row.author})
  MERGE (b:Book {title: row.title})
  MERGE (a)-[:WROTE]->(b)
}
```

- Avoid Conflicting Reads/Writes in Same Query
```
// Problem
MATCH (p:Person)
DELETE p
CREATE (p:Person {name: 'New'})  // conflict: same variable name `p`

// Better
MATCH (p:Person)
DELETE p
CREATE (:Person {name: 'New'})
```

- Avoid Dependent MERGE Chains
```
// Problem
MERGE (a:Author {name: row.author})-[:WROTE]->(b:Book {title: row.title})

// Better
MERGE (a:Author {name: row.author})
MERGE (b:Book {title: row.title})
MERGE (a)-[:WROTE]->(b)
```