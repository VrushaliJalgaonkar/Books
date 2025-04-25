# Chapter 2: Data Models and Query Languages – Summary

Data models are foundational in software development, deeply influencing both how applications are written and how we conceptualize the problems we're solving. Most software systems involve layers of data models, from high-level objects down to bytes on disk or memory.

## Layered Data Representations

Applications start by modeling the real world using objects or data structures. These are stored using general-purpose data models like:
- **JSON/XML documents**
- **Relational tables**
- **Graphs**

Each layer abstracts the complexity beneath it, allowing teams (e.g., app developers vs. DB engineers) to work independently.

---

## Relational vs. Document Models

### **Relational Model (SQL)**
Introduced by **Edgar Codd in 1970**, the relational model organizes data in *tables* (relations) of *rows* (tuples). Its core strength lies in abstraction: developers need not worry about internal storage formats.

Relational databases dominated for decades due to their:
- Efficiency in structured, business data
- Easy handling of joins and normalization
- Widespread standardization (SQL)

Early alternatives—like the hierarchical and network models—faded away despite initial traction. Later models (e.g., object databases, XML databases) also saw limited success.

Relational databases have surprisingly adapted well to modern applications such as:
- E-commerce
- Social media
- SaaS platforms

---

## The Rise of NoSQL

By the 2010s, new requirements spurred interest in **NoSQL** databases. Though the term began as a hashtag, it came to encompass a range of *non-relational* systems. Drivers included:
- Demand for scalability and performance (big data, high write throughput)
- Preference for open source
- Need for flexible schemas and expressive data models

NoSQL reintroduced many ideas from older models, especially the **document model**, which stores self-contained objects (e.g., JSON).

---

## Object-Relational Mismatch

Most applications are built using object-oriented languages, creating a mismatch when data is stored in relational tables. This **impedance mismatch** requires boilerplate-heavy mapping layers (ORMs like Hibernate or ActiveRecord).

A **resume profile** is a good example of how the object model doesn't align neatly with the relational model. For example:
- **User data** (first name, last name) can fit into SQL tables easily.
- **Jobs, education, and contacts** need to be split into separate tables, linked via foreign keys.
- The mismatch between the object-oriented code and SQL tables often leads to a disconnect between developers and databases.

---

## Relational Model vs. Document Model

### Relational Model
The relational model, pioneered by Edgar Codd, organizes data in tables and rows. It abstracts away the complexities of how data is stored. Relational databases have dominated business data processing since the 1980s due to their:
- Flexibility for various use cases
- Support for queries with complex relationships (via joins)
- Consistency and data integrity

Despite this, the relational model struggles when data is not naturally tabular, such as with hierarchical or complex object data.

### Document Model
Document models (e.g., **JSON**, **XML**) represent data as self-contained documents. This allows for flexibility in storing complex data structures without requiring a rigid schema. NoSQL databases like MongoDB use this model, storing data as JSON-like documents.

Documents can store complex, nested relationships and one-to-many structures without needing multiple tables or joins. However, they can face challenges when dealing with **many-to-many relationships**, as joins are not easily supported. In such cases, developers might need to handle the relationships in application code.

---

## The Evolution of NoSQL

In the 2010s, NoSQL databases rose to prominence with the need for high scalability, flexibility, and performance. Unlike relational databases, NoSQL databases:
- Offer greater flexibility in schema design
- Provide high availability and partition tolerance in distributed environments
- Scale horizontally, making them ideal for handling large datasets or high throughput

Despite its popularity, NoSQL does not provide the same level of consistency as relational databases, leading to **trade-offs** depending on the use case.

---

## Object-Relational Mapping (ORM)

Object-Oriented Programming (OOP) languages and relational databases often clash due to **impedance mismatch**. To bridge this gap, developers use **ORM frameworks** like ActiveRecord or Hibernate. These tools automatically map objects in the code to relational tables, reducing the amount of boilerplate code needed.

For example, in a resume profile, OOP languages store a user’s **job history** and **education** as objects. However, the relational model would require creating separate tables for these data points, complicating the structure.

While ORMs help with basic mapping, they don’t completely eliminate the differences between OOP and relational models.

---

## Many-to-Many and One-to-Many Relationships

In both relational and document models, **relationships** between data entities (e.g., users, jobs, schools) are critical. In SQL, relationships are established through **foreign keys** and **joins**. Document databases handle these relationships differently by embedding data or using **references**. However, embedding data can lead to **duplication** and **maintenance overhead**.

For example, in a resume profile:
- **One-to-many relationships** (a user has multiple jobs) are easy to model in both SQL and document models.
- **Many-to-many relationships** (a user has multiple recommendations, a job involves multiple users) become complex in document databases due to the lack of join support.

---

## Conclusion: Polyglot Persistence

The rise of NoSQL and document databases highlights that there is no one-size-fits-all solution. Different applications have different needs, and the best database technology depends on those requirements. The future likely holds a combination of relational and NoSQL systems in a practice called **polyglot persistence**, where multiple data models are used within a single application to leverage their strengths.

---
# The Network Model

The **network model** was standardized by the **Conference on Data Systems Languages (CODASYL)** and implemented by several database vendors. It was a generalization of the hierarchical model. In the hierarchical model, every record had exactly one parent, but in the network model, a record could have multiple parents. For example, a record for the "Greater Seattle Area" region could be linked to multiple users living there, allowing for many-to-one and many-to-many relationships.

In the network model, the links between records were not foreign keys but more like pointers in a programming language, stored on disk. The only way to access a record was to follow a path from a root record through these chains of links, called an **access path**.

In simple terms, an access path was like traversing a linked list: starting from the head of the list and looking at one record at a time. However, in a world of many-to-many relationships, multiple paths could lead to the same record. Programmers had to manually track all possible paths.

Querying in CODASYL involved moving a cursor through the database by iterating over record lists and following access paths. If a record had multiple parents, the application code had to track all these relationships. Even CODASYL committee members admitted that navigating the model was like working in an **n-dimensional data space**.

While this manual access path selection was efficient given the limited hardware of the 1970s (such as slow tape drives), it made database queries and updates complicated and inflexible. If the access path was unavailable, programmers had to rewrite the query code to handle new paths, making changes to the data model difficult.

---

# The Relational Model

The **relational model**, by contrast, laid out data simply in the form of relations (tables), which are collections of tuples (rows). There are no nested structures or complicated access paths. Data can be read in any order, and specific rows can be accessed by matching keys.

In relational databases, the **query optimizer** automatically decides how to execute queries, including which indexes to use. These decisions are the “access paths,” but unlike in the network model, they are handled by the optimizer rather than the developer, making database interaction much simpler.

Relational databases make it easy to add new features. If you want to query data in new ways, you can simply create a new index, and the query optimizer will use it without needing changes to the queries themselves. This approach eliminates the need for developers to manage complex access paths manually.

Relational query optimizers have been developed over many years, but once created, they benefit all applications that use the database. Without a query optimizer, it would be easier to handcode specific access paths for individual queries, but in the long run, a general-purpose optimizer is more efficient.

---

# Comparison to Document Databases

Document databases have a resemblance to the hierarchical model in that they store nested records (one-to-many relationships) within their parent record. However, when it comes to many-to-one and many-to-many relationships, both relational and document databases are similar. In both models, related items are referenced by unique identifiers (foreign keys in relational, document references in document databases), resolved at read time through joins or follow-up queries.

While document databases have schema flexibility and better performance due to locality, the relational model excels in supporting joins and many-to-many relationships. **Document databases** are advantageous when the data structure resembles a tree of one-to-many relationships, but relational databases handle many-to-many relationships and joins more efficiently.

---

# Schema Flexibility in the Document Model

Most document databases and the **JSON support** in relational databases do not enforce a schema on the data in documents. This is called **schema-on-read**, where the data structure is implicit and interpreted when read, as opposed to **schema-on-write** in relational databases, where the structure is enforced at write time.

The schema-on-read approach is beneficial when the data has different structures or when external systems, over which you have no control, may change the data format. For example, changing the structure of user data in a document database requires no migration; new documents with the new fields can be written, and the application code can handle reading older documents.

In relational databases, altering the schema requires explicit changes like `ALTER TABLE`, which can take time and cause downtime for large tables. However, the relational model enforces consistency across all data, which can be advantageous when records have a uniform structure.

---

# Data Locality for Queries

Document databases store documents as a continuous string (e.g., JSON, XML, BSON), which can improve performance if the entire document is often accessed at once. However, if only part of the document is needed, accessing the entire document may be inefficient.

Relational databases, by splitting data across multiple tables, may require more disk seeks and time to retrieve data. However, document databases can be inefficient in situations where updates to large documents require rewriting the entire document.

The concept of **data locality** is not exclusive to document models. Relational databases, like Google’s **Spanner** or Oracle, also support locality by allowing tables to be nested or interleaved, improving access speed for related data.

---

# Convergence of Document and Relational Databases

Relational databases like **PostgreSQL** and **MySQL** now support XML and JSON, allowing them to handle document-like data. Similarly, document databases like **RethinkDB** support relational-like joins, and **MongoDB** performs client-side joins. This convergence indicates that the two models are complementing each other, and combining their features may provide the best solutions for future databases.
----


# The Cypher Query Language

Cypher is a declarative query language for property graphs, created for the Neo4j graph database. (It is named after a character in the movie *The Matrix* and is not related to ciphers in cryptography.)

## Cypher Example: Creating Data in a Graph

```cypher
CREATE
  (NAmerica:Location {name:'North America', type:'continent'}),
  (USA:Location {name:'United States', type:'country'}),
  (Idaho:Location {name:'Idaho', type:'state'}),
  (Lucy:Person {name:'Lucy'}),
  (Idaho) -[:WITHIN]-> (USA) -[:WITHIN]-> (NAmerica),
  (Lucy) -[:BORN_IN]-> (Idaho)
```

This Cypher query inserts data into the graph, establishing nodes such as *North America*, *United States*, *Idaho*, and a person named *Lucy*. The relationships between these nodes, such as `WITHIN` and `BORN_IN`, are also represented.

## Query Example: Finding People Who Moved from the US to Europe

```cypher
MATCH
  (person) -[:BORN_IN]-> () -[:WITHIN*0..]-> (us:Location {name:'United States'}),
  (person) -[:LIVES_IN]-> () -[:WITHIN*0..]-> (eu:Location {name:'Europe'})
RETURN person.name
```

This query finds people who were born in the US and live in Europe, traversing the graph using relationships like `BORN_IN` and `LIVES_IN`. The `*0..` allows for variable-length traversal of the `WITHIN` relationship.

## Graph Queries in SQL

Graph data can be represented in a relational database, but querying it with SQL is cumbersome due to the difficulty in specifying variable-length joins.

### SQL Example: Same Query in SQL

```sql
WITH RECURSIVE
  in_usa(vertex_id) AS (
      SELECT vertex_id FROM vertices WHERE properties->>'name' = 'United States'
      UNION
      SELECT edges.tail_vertex FROM edges JOIN in_usa ON edges.head_vertex = in_usa.vertex_id WHERE edges.label = 'within'
  ),
  in_europe(vertex_id) AS (
      SELECT vertex_id FROM vertices WHERE properties->>'name' = 'Europe'
      UNION
      SELECT edges.tail_vertex FROM edges JOIN in_europe ON edges.head_vertex = in_europe.vertex_id WHERE edges.label = 'within'
  ),
  born_in_usa(vertex_id) AS (
      SELECT edges.tail_vertex FROM edges JOIN in_usa ON edges.head_vertex = in_usa.vertex_id WHERE edges.label = 'born_in'
  ),
  lives_in_europe(vertex_id) AS (
      SELECT edges.tail_vertex FROM edges JOIN in_europe ON edges.head_vertex = in_europe.vertex_id WHERE edges.label = 'lives_in'
  )
SELECT vertices.properties->>'name'
FROM vertices
JOIN born_in_usa ON vertices.vertex_id = born_in_usa.vertex_id
JOIN lives_in_europe ON vertices.vertex_id = lives_in_europe.vertex_id;
```

This SQL version of the query uses recursive common table expressions (CTEs) to traverse the `WITHIN` relationships and find people born in the US and living in Europe.

## Triple-Stores and SPARQL

Triple-stores are similar to property graphs, but they represent data as subject-predicate-object triples. Each triple is akin to a graph edge connecting two vertices.

### Example Triple (Turtle format):

```turtle
@prefix : <urn:example:>.
_:lucy a :Person; :name "Lucy"; :bornIn _:idaho.
_:idaho a :Location; :name "Idaho"; :type "state"; :within _:usa.
_:usa a :Location; :name "United States"; :type "country"; :within _:namerica.
_:namerica a :Location; :name "North America"; :type "continent".
```

### SPARQL Query Example

```sparql
PREFIX : <urn:example:>
SELECT ?personName WHERE {
  ?person :name ?personName.
  ?person :bornIn / :within* / :name "United States".
  ?person :livesIn / :within* / :name "Europe".
}
```

The SPARQL query is concise and uses path expressions, similar to Cypher's variable-length traversal.

## RDF and SPARQL

RDF (Resource Description Framework) is often used for representing triples. SPARQL is the query language for RDF data, enabling sophisticated graph querying. 

### RDF/XML Example

```xml
<rdf:RDF xmlns="urn:example:" xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#">
  <Location rdf:nodeID="idaho">
    <name>Idaho</name>
    <type>state</type>
    <within>
      <Location rdf:nodeID="usa">
        <name>United States</name>
        <type>country</type>
        <within>
          <Location rdf:nodeID="namerica">
            <name>North America</name>
            <type>continent</type>
          </Location>
        </within>
      </Location>
    </within>
  </Location>
</rdf:RDF>
```

## Graph Databases vs Network Model

Graph databases differ significantly from the older CODASYL network model, offering greater flexibility. Key differences include:

1. **Schema Flexibility**: Graph databases don't require predefined relationships between record types.
2. **Direct Access**: You can refer directly to any vertex, unlike CODASYL where you had to follow specific access paths.
3. **Declarative Querying**: Graph databases often support high-level, declarative queries (e.g., Cypher, SPARQL), unlike the imperative style of CODASYL.
------

# The Foundation: Datalog

Datalog is a much older language than SPARQL or Cypher, having been studied extensively by academics in the 1980s [44,45,46]. It is less well known among software engineers, but it is nevertheless important, because it provides the foundation that later query languages build upon.

In practice, Datalog is used in a few data systems: for example, it is the query language of Datomic [40], and Cascalog [47] is a Datalog implementation for querying large datasets in Hadoop.viii

## Datalog Data Model

Datalog’s data model is similar to the triple‑store model, generalized a bit. Instead of writing a triple as `(subject, predicate, object)`, we write it as `predicate(subject, object)`.

### Example 2‑10. A subset of the data in Figure 2‑5, represented as Datalog facts

```prolog
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

## Defining Queries with Rules

Now that we have defined the data, we can write the same query as before. Datalog is a subset of Prolog, so we define rules rather than issuing a `SELECT` directly.

### Example 2‑11. The same query as Example 2‑4, expressed in Datalog

```prolog
within_recursive(Location, Name) :- name(Location, Name).     /* Rule 1 */

within_recursive(Location, Name) :- within(Location, Via),    /* Rule 2 */
                                    within_recursive(Via, Name).

migrated(Name, BornIn, LivingIn) :- name(Person, Name),       /* Rule 3 */
                                    born_in(Person, BornLoc),
                                    within_recursive(BornLoc, BornIn),
                                    lives_in(Person, LivingLoc),
                                    within_recursive(LivingLoc, LivingIn).

?- migrated(Who, 'United States', 'Europe').
/* Who = 'Lucy'. */
```

Cypher and SPARQL jump in right away with `SELECT`, but Datalog takes a small step at a time. We define rules that tell the database about new predicates: here, we define two new predicates, `within_recursive` and `migrated`. These predicates aren’t stored triples but are derived from data or other rules. Rules can refer to one another or call themselves recursively, allowing complex queries to be built up incrementally.

In rules, words that start with an uppercase letter are variables, and predicates are matched like in Cypher and SPARQL. For example, `name(Location, Name)` matches the fact

```prolog
name(namerica, 'North America').
```

with variable bindings `Location = namerica` and `Name = 'North America'`.

A rule applies if the system can find matches for all predicates on the right side of `:-`. When it applies, it’s as though the left side of `:-` was added to the database (with variables replaced by their matched values).

One possible way of applying the rules is:

1. `name(namerica, 'North America')` exists, so Rule 1 generates:
   ```prolog
   within_recursive(namerica, 'North America')
   ```
2. `within(usa, namerica)` exists and the previous step generated `within_recursive(namerica, 'North America')`, so Rule 2 generates:
   ```prolog
   within_recursive(usa, 'North America')
   ```
3. `within(idaho, usa)` exists and the previous step generated `within_recursive(usa, 'North America')`, so Rule 2 generates:
   ```prolog
   within_recursive(idaho, 'North America')
   ```

By repeated application of Rules 1 and 2, the `within_recursive` predicate can infer all locations contained within a given place. This process is illustrated in Figure 2‑6.

Now Rule 3 can find people who were born in some location `BornIn` and live in some location `LivingIn`. By querying with `BornIn = 'United States'` and `LivingIn = 'Europe'`, and leaving the person as a variable `Who`, we ask the Datalog system to discover which values `Who` can take. The result matches the answer from the earlier Cypher and SPARQL queries.

The Datalog approach requires a different way of thinking than other query languages, but it’s very powerful: rules can be combined and reused across queries. It may be less convenient for simple one-off queries, but it scales elegantly when dealing with complex, interconnected data.
