# Neo4j Bolt Driver for Julia

Neo4j is the world’s leading Graph Database. It is a high performance graph store with all the features expected of a mature and robust database, like a friendly query language and ACID transactions. The programmer works with a flexible network structure of nodes and relationships rather than static tables — yet enjoys all the benefits of enterprise-quality database. For many applications, Neo4j offers orders of magnitude performance benefits compared to relational DBs.

Neo4jBolt.jl is a Julia port of the official [Neo4j Driver](https://github.com/neo4j/neo4j-python-driver). It supports Neo4j 3.0 and above using the fast binary Bolt protocal. 

This driver does not support HTTP REST based communications. For a Neo4j Julia driver that supports HTTP and REST, see [Neo4j.jl](https://github.com/glesica/Neo4j.jl). Additionally, encrypted SSL connections and cluster routing have not yet been implemented in this version.

## Getting Started with Neo4j

* [Introduction (The Neo4j Operations Manual v3.5)](https://neo4j.com/docs/operations-manual/current/introduction/)
* [Installation (The Neo4j Operations Manual v3.5)](https://neo4j.com/docs/operations-manual/current/installation/)

## Quick Examples

Here are a few usage examples. For a more extensive collection of examples see the integration tests in the test/integration directory of this repository.

### Run Cypher Statement

```
using Neo4jBolt  
      
driver = Neo4jBoltDriver("bolt://localhost:7687", auth=("neo4j", "password"))

session(tc.driver) do sess
    result = run(sess, "UNWIND(RANGE(1, 10)) AS z RETURN z")
    for record in result
        println(record["z"])
    end
end
```


### Run Simple Transaction

```
using Neo4jBolt  
      
driver = Neo4jBoltDriver("bolt://localhost:7687", auth=("neo4j", "password"))

session(tc.driver) do sess
    begin_transaction(sess) do tx
        result = run(tx, "CREATE (a:Person {name:'Alice'}) RETURN a")
        v = value(single(result))
        println(v.labels == Set(["Person"]))
        println(v.properties == Dict("name"=>"Alice"))
    end
end
```


### Unit of Work transactions

```
using Neo4jBolt  
      
driver = Neo4jBoltDriver("bolt://localhost:7687", auth=("neo4j", "password"))
        
function add_friend(tx, args, kwargs)
    run(tx, "MERGE (a:Person {name: \$name}) " *
            "MERGE (a)-[:KNOWS]->(friend:Person {name: \$friend_name})", name=args[1], friend_name=args[2])
end

function print_friends(tx, args, kwargs)
    for record in run(tx, "MATCH (a:Person)-[:KNOWS]->(friend) WHERE a.name = \$name " *
                          "RETURN friend.name ORDER BY friend.name", name=args[1])
        println(record["friend.name"])
    end
end        
        
session(driver) do sess
    write_transaction(sess, add_friend, "Arthur", "Guinevere")
    write_transaction(sess, add_friend, "Arthur", "Lancelot")
    write_transaction(sess, add_friend, "Arthur", "Merlin")
    read_transaction(sess, print_friends, "Arthur")
end
```