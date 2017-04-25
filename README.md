# Neo4j: Cypher basics #

Using the docke-compose file we can run nodejs on the fly. Download the project and navigate to the directory where the docker-compose file has been placed and execute
```
docker-compose up
```
This will download the neo4j image and run in the foreground. use the flag -d to run it in the background.

The docker file looks like the following:
```
version: '2'
services:
  neo4j:
    image: neo4j:3.0
    ports:
     - "7474:7474"
    volumes:     
     - /dev/neo4j/data:/data
     - /dev/neo4j/plugins:/plugins   
```
We map the ports from the container to the host machine. The mapping format is as follows {host machine:container}
If we want to use a different port on the host, port 1000 for example then we will modify the mapping to look something like this
`"1000:7474"`.

Finally we have volumes that can be used to persiste data. Containers don't maintain data unless we store it somewhere. 
In this example we have two volumes:
1. Data volume: Used to store the data created by the database. Here we are mapping `dev/neo4j/data`on the host to the directory `data` on the container.
2. Plugin volume: Plugins are used to extend neo4j. Any plugins are placed in `/dev/neo4j/plugins` will be mounted on the container in the directory `/plugins`.

## Example: Creating nodes and linking them ##
In this example we will do the following:
1. Create two people. (2 nodes of type `Person`)
2. Create 4 houses. (4 nodes of type `House`)
3. Create 2 streets. (2 nodes of type `Street`)
4. Create 1 city. (1 nodes of type `City`)

Each of nodes will have different properties.

### Create the first house node ###

```
CREATE (house1:House {number:1})
RETURN house1
```

### Create the other 3 houses ###
Create 3 nodes of type House and return them. Each of the nodes will have a number property that represents the `house number` portion of the address.
```
CREATE (house1:House {number:12})
CREATE (house2:House {number:54})
CREATE (house3:House {number:73})
RETURN house1, house2, house3
```

### Get all the nodes ###

In order to get all nodes and display them, execute the following command that matches all nodes. It's like doing SELECT * in SQL.
```
MATCH(n) RETURN n
```

### Creating nodes and connecting them on the fly ###
Here we will creates two nodes of type `Person` and link them using a `Brother` relationship. Please notice that relationship is directional. From John towards Jane.
```
CREATE (john:Person{name:"John Doe"})-[r:Brother]->(jane:Person{name:"Jane Doe"})
RETURN john, jane, r
```

### Linking existing nodes ###
In this example we will place Jane and John in different houses. We will link each one of them with a house using a `LIVES` relationship.
```
MATCH(jane:Person{name:"Jane Doe"})
MATCH(john:Person{name:"John Doe"})
MATCH(h1:House{number:12})
MATCH(h2:House{number:1})
CREATE UNIQUE (jane)-[r1:LIVES]->(h1)
CREATE UNIQUE (john)-[r2:LIVES]->(h2)
return r1, r2
```
### Create more nodes ###
We will create a city and two streets and place the streets in the city.
```
CREATE(luxCity:City  {name: "Luxembourg City" })
CREATE(street1:Street{name:"rue evrard ketten"})-[:LOCATED]->(luxCity)
CREATE(street2:Street{name:"Rue Henri VII"    })-[:LOCATED]->(luxCity)
return luxCity
```

### Connecting more nodes ###
In this example we will connect all the nodes having the type `House` with the nodes having the type `Street`
>The `CREATE UNIQUE` is used to create a single relationship of a given type. We use this to avoid duplicates.
```
MATCH(rue_henri:Street{name:"Rue Henri VII"}) 
MATCH(rue_evrard:Street{name:"rue evrard ketten"}) 
MATCH(h1:House{number:12})
MATCH(h2:House{number:1})
MATCH(h3:House{number:54})
MATCH(h4:House{number:73})
CREATE UNIQUE (h1)-[:LOCATED]->(rue_henri)
CREATE UNIQUE (h2)-[:LOCATED]->(rue_henri)
CREATE UNIQUE (h3)-[:LOCATED]->(rue_evrard)
CREATE UNIQUE (h4)-[:LOCATED]->(rue_evrard)
RETURN rue_henri, rue_evrard
```
