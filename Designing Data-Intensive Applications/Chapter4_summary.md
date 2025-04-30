# Chapter 4 Summary: Encoding and Evolution (750 words)

## Evolvability and Data Evolution
Applications inevitably evolve over time—due to new features, refined requirements, or changing business needs. With such evolution, data formats and schemas often need to change too. This presents a challenge: ensuring systems remain functional despite schema and code changes. Systems must be designed with **evolvability** in mind—capable of adapting without breaking existing functionality.

Relational databases enforce a single schema at any point, requiring migrations when the schema changes. In contrast, **schema-on-read** systems (like schemaless document stores) allow data with varying formats to coexist.

## Compatibility Concerns
Schema changes usually accompany code changes. In distributed systems:
- **Server-side upgrades** are often done as **rolling upgrades** to avoid downtime.
- **Client-side upgrades** depend on user adoption, so old versions may persist longer.

Thus, old and new code, and old and new data formats, often coexist. Systems must support:
- **Backward compatibility**: New code can read old data.
- **Forward compatibility**: Old code can read new data (by ignoring unknown fields).

## Data Encoding Fundamentals
Applications deal with data in two forms:
1. **In-memory structures** (objects, lists, maps, etc.) optimized for CPU access.
2. **Serialized formats** (byte sequences) for storage or transmission.

Translation between these is:
- **Encoding** (a.k.a. serialization): From memory to bytes.
- **Decoding** (a.k.a. deserialization): From bytes to memory.

Encoding must balance compatibility, efficiency, and cross-language support.

## Language-Specific Encoding Formats
Languages like Java (`Serializable`), Python (`pickle`), and Ruby (`Marshal`) offer built-in serialization. However, these formats have drawbacks:
- Tightly coupled to the language (poor cross-language support).
- Security risks from arbitrary object deserialization.
- Poor versioning support.
- Inefficiency in size and speed (e.g., Java serialization).

Thus, they are suitable only for short-lived, internal use.

## Language-Independent Formats
### Text-Based Formats: JSON, XML, CSV
These are widely supported but have limitations:
- **JSON** is simple and browser-friendly but ambiguous with number precision and binary data.
- **XML** is verbose and complex.
- **CSV** lacks schemas, making versioning difficult and error-prone.

Issues include:
- Distinguishing between numeric types (e.g., 64-bit integers vs floats).
- Lack of binary support (workarounds like Base64 are common).
- Optional schema support (e.g., JSON Schema, XML Schema) is often ignored.

Despite limitations, these formats remain popular for external data interchange due to their simplicity and ubiquity.

### Binary Variants of JSON/XML
Binary encodings aim for compactness and speed:
- Examples: MessagePack, BSON, Smile (for JSON); WBXML (for XML).
- Tradeoff: Slight space savings (~20%) but loss of human readability.

Example JSON:
```json
{
  "userName": "Martin",
  "favoriteNumber": 1337,
  "interests": ["daydreaming", "hacking"]
}
```
MessagePack encodes this in 66 bytes (vs 81 for plain JSON).

## Thrift and Protocol Buffers (Protobuf)
Both are efficient, schema-based binary encoding libraries:
- **Thrift** (from Facebook)
- **Protobuf** (from Google)

They require defining a schema in an **Interface Definition Language (IDL)**. The schema is compiled into code for multiple languages, allowing consistent, fast encoding/decoding.

Example Protobuf schema:
```proto
message Person {
  required string user_name = 1;
  optional int64 favorite_number = 2;
  repeated string interests = 3;
}
```

### Encoding Strategy
- Field names are replaced by compact numeric **tags** (e.g., `1`, `2`, `3`).
- Field types and lengths are encoded compactly.
- Protobuf uses **varint encoding** for integers: small numbers use fewer bytes.

Encoding the earlier JSON example results in:
- **Thrift BinaryProtocol**: 59 bytes
- **Thrift CompactProtocol**: 34 bytes
- **Protobuf**: 33 bytes

## Schema Evolution
Protobuf and Thrift support **schema evolution** via:
- **Field tags**: Each field is identified by its tag number, allowing renamed fields to keep the same tag.
- **Adding/removing fields**: New fields can be added (as optional) without breaking old code.
- **Unknown fields**: Ignored by default, aiding forward compatibility.

To ensure evolution safety:
- Never reuse or repurpose field tags.
- Mark new fields as optional or repeated.

## Encoding Use Cases
- **Storage**: Format should support long-term compatibility and efficient querying.
- **Communication**:
  - **REST APIs** typically use JSON or XML.
  - **RPC systems** often use Protobuf, Thrift, or Avro.
  - **Messaging systems** benefit from compact binary formats for performance.

## Conclusion
Choosing the right encoding format depends on:
- Human readability vs. efficiency.
- Internal vs. external use.
- Need for long-term schema evolution support.

For evolvability, schema-based, compact binary formats like Protobuf or Thrift are ideal—especially for internal systems and high-performance needs. JSON/XML remain dominant for open APIs and inter-organizational communication.

---

# Apache Avro: A Schema-Based Binary Encoding Format

Apache Avro is a binary data serialization format developed as part of the Hadoop project. It was designed to better fit Hadoop's requirements compared to other formats like Protocol Buffers or Thrift. Avro employs a schema to define the structure of the data being encoded, offering two schema representations: Avro IDL for human readability and JSON for machine readability.

## Example Schema
**Avro IDL:**
```avdl
record Person {
    string               userName;
    union { null, long } favoriteNumber = null;
    array<string>        interests;
}
```

**JSON Representation:**
```json
{
    "type": "record",
    "name": "Person",
    "fields": [
        {"name": "userName", "type": "string"},
        {"name": "favoriteNumber", "type": ["null", "long"], "default": null},
        {"name": "interests", "type": {"type": "array", "items": "string"}}
    ]
}
```

## Compact Encoding
Avro omits field tags or types in the encoded binary. It relies on the schema for parsing the data. Strings are encoded as length-prefixed UTF-8 bytes, and integers use variable-length encoding. Consequently, the decoding process must have the exact schema used during encoding.

## Schema Evolution
Avro separates the writer’s schema (used during encoding) and the reader’s schema (used during decoding). Both schemas need to be compatible, not identical. Schema resolution handles translation between these schemas, matching fields by name.

### Compatibility Types
- **Forward compatibility:** New writer, old reader.
- **Backward compatibility:** Old writer, new reader.

### Evolution Rules
- Fields can be added/removed if they have default values.
- Null must be part of a union type to be a valid default.
- Field types can change if convertible.
- Field names can change using aliases.
- Adding union branches is backward compatible, not forward compatible.

## Distribution of Writer's Schema
Avro avoids embedding the entire schema in each record due to space constraints. Different distribution strategies exist:

### Use Cases
- **Large files:** Writer’s schema is included once in the file header (e.g., Avro object container files).
- **Databases:** Include a version number per record and store schemas separately.
- **Network Communication:** Negotiate schema versions during connection setup (used by Avro RPC).

Maintaining a schema registry (database of schemas) ensures compatibility checks and serves as documentation.

## Dynamically Generated Schemas
Unlike Thrift and Protocol Buffers, Avro avoids field tags, making it more suitable for dynamically generated schemas. For example, database schemas can be converted directly into Avro schemas with minimal manual intervention. This allows seamless export and import of evolving database tables.

## Code Generation
While Thrift and Protocol Buffers rely heavily on code generation for statically typed languages, Avro supports optional code generation. In dynamically typed languages (JavaScript, Python, etc.), it can be used without code generation, treating files similarly to JSON.

This flexibility is especially useful with data processing tools like Apache Pig, allowing analysis and transformation without needing schema-specific code.

## Merits of Schemas
- Binary formats omit field names, making them compact.
- Schemas act as always-updated documentation.
- Schema registries support compatibility validation.
- Type-safe code generation is possible in statically typed languages.

Avro provides schema evolution capabilities akin to schemaless systems (like JSON databases), but with stronger guarantees and better tooling.

## Broader Context: Modes of Dataflow
Data must be encoded when moving between processes that don’t share memory. Compatibility ensures independent upgrades and maintainability.

### Common Dataflow Patterns:
- **Through Databases:** Writes must be forward-compatible; reads must be backward-compatible due to concurrent code versions.
- **Via Services (REST/RPC):** Discussed elsewhere in the chapter.
- **Via Message Passing:** Discussed elsewhere as well.

Binary formats with schema-based encoding, such as Avro, provide a flexible, compact, and reliable approach to data serialization, especially in distributed systems with evolving schemas.

----

## Data Encoding, Evolution, and Service Communication Summary (Chapter 4, DDDIA)

### 1. **Data Outlives Code**
- Databases store values written at different times, potentially years apart.
- When application code is updated, data often remains in its original form.
- This persistence of legacy data underscores the principle: *data outlives code*.

### 2. **Schema Evolution and Migration**
- Updating the entire dataset to match a new schema is costly and avoided when possible.
- Relational databases allow non-intrusive schema changes (e.g., adding nullable columns).
- Systems like LinkedIn's Espresso use Avro, which supports schema evolution.
- Avro allows seamless reads across schema versions by enforcing evolution rules.

### 3. **Archival Storage**
- Backups and data exports often use a consistent, current schema.
- Immutable formats like Avro containers or Parquet (columnar) are well-suited.
- Ideal for analytics and data warehousing due to performance and format consistency.

### 4. **Dataflow Between Services**
- **Client-server model** is standard: clients make API requests to servers.
- Web services (HTTP, REST) and application services often rely on this pattern.
- RESTful communication uses standard web protocols and data formats (JSON, HTML).
- REST emphasizes simplicity and compatibility using HTTP features.

### 5. **Types of Web Service Communication**
- Clients: browsers, mobile apps, or JS using Ajax.
- Internal service-to-service calls within the same org/data center (microservices).
- External org-to-org communication via public APIs (e.g., OAuth, payment systems).

### 6. **REST vs. SOAP**
- **REST**: Design philosophy using HTTP semantics, URLs, and JSON. Lightweight, human-readable, and easily debuggable.
- **SOAP**: Protocol with XML-based WSDL definitions and WS-* standards. Heavily reliant on tooling and code generation.
- SOAP is verbose and harder to work with, especially across language ecosystems.

### 7. **Service-Oriented Architecture (SOA) & Microservices**
- Applications are split into independently deployable services.
- Teams own individual services, enabling agile development.
- Version compatibility is crucial: expect old and new versions to coexist.
- APIs must support backward (requests) and forward (responses) compatibility.

### 8. **RPC (Remote Procedure Calls) and Its Problems**
- RPC tries to make network requests look like local function calls (location transparency).
- Key issues:
  - Unpredictability due to network failures and timeouts.
  - Retrying can lead to duplicate actions if not idempotent.
  - Serialization overhead and datatype mismatches across languages.
  - Highly variable latency.
- Network requests are fundamentally different from local calls and shouldn't be abstracted away.

### 9. **Modern RPC Frameworks**
- Examples: Thrift, Avro, gRPC (Protocol Buffers), Finagle (Thrift), Rest.li (JSON).
- Explicit about asynchronous behavior and failures (use of futures/promises).
- Support streaming, parallel requests, and service discovery.
- Binary formats can be more performant but less debuggable than JSON/REST.

### 10. **REST vs. RPC**
- REST: Easier for public APIs due to tooling, readability, and support.
- RPC: Preferred for internal service-to-service communication within orgs.
- REST uses JSON, with compatibility managed via optional fields and versioning (URLs, headers).

### 11. **Service Compatibility and Versioning**
- Compatibility is difficult when services span organizational boundaries.
- API versioning strategies:
  - Version in URL (e.g., `/v1/users`)
  - HTTP headers (e.g., `Accept: application/vnd.company.v2+json`)
  - Server-side version management tied to API keys

### 12. **Asynchronous Message-Passing Systems**
- Sits between RPC and databases.
- Uses a message broker (queue) to deliver messages asynchronously.
- Messages are temporarily stored and forwarded by the broker.
- Benefits:
  - Decouples sender and receiver
  - Increases reliability
  - Supports buffering and batch processing

---
### Message Brokers and Distributed Actor Frameworks

Message brokers like RabbitMQ and Kafka enable message delivery to multiple consumers with flexible encoding formats. They support one-way dataflow, and consumers can republish messages. Distributed actor frameworks use asynchronous messaging to scale applications across nodes, ensuring location transparency but requiring careful handling for rolling upgrades.


