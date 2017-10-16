
# RDF Overview

## 1. What is RDF?
 RDF is used to represent information about remote resources (in particular, semantic  information of the data). Capturing semantic information was fuelled by the need to exchange knowledge in the web. It defines a model for describing relationships among resources in terms of uniquely identified properties (attributes) and values.

## 2. <S,P,O> - DataModel of RDF:
RDF expresses knowledge as tuple of <S,P,O> where subject S has property P with value O. S and P are resource URIs and O is either a URI or a literal value. 

### 2.1 RDF is flexible and dynamic DataModel:
Any resource can be associated with any property or type irrespective of its type. This flexibility makes RDF a perfect match to represent metadata because resource descriptions cannot necessarily be bound by fixed schemas.

### 2.2 Comparison of RDF with OO (object-oriented):
**Common points between an object and RDF:**
- RDF properties represent attributes of an object
- RDF statements (SPO tuple) expresses the attribute values of objects. 
**Difference between object and RDF:**
- The properties of an object are constrained by its type hierarchy where as RDF resources can be associated with any arbitrary property

## 3. Persistence

### 3.1 XML storage
A naïve approach might be to map the RDF data to XML and rely on the efficient storage of XML.
- **Drawback:** expensive XML serialisations  which in turn makes retrieval complex.

### 3.2. Relational/Object DB
Many RDF systems have used relational or object databases for persistent storage and retrieval. For relational databases, the schema consisted of 3 tables: statement table, a literals table and a resources table.
![image](https://user-images.githubusercontent.com/22542670/31597018-b62ff12e-b263-11e7-8758-c2b349c71ffa.png)

To distinguish literal objects from resource URIs, two columns were used. The literals table contained all literal values and the resources table contained all resource URIs in the graph. 

- **Advantages:** This schema was very efficient in space as multiple occurrences of the same resource URI or literal value were only stored once. 
- **Drawbacks:** However, every find operation required multiple joins between the statement table and the literals table or the resources table.

### 3.3 BerkeleyDB format
In this approach, each statement i.e., SPO tuple, was stored three times: once indexed by subject, once by predicate and once by object.
- **Advantages:** Access becomes significantly faster than graphs stored in relational
databases.
- **Disadvantages:** Obviously, this schema is a trade-off between space and response-time.

### 3.4 Hybrid approach
Drawing on experience from the denormalized schema in which resource URIs and simple literal values are stored directly in the statement table. 
A separate literals table is only used to store literal values whose length exceeds a threshold, such as blobs. 
Similarly, a separate resources table is used to store long URIs. 
- **Advantages:** By storing values directly in the statement table it is possible to perform many find operations without a join. 
- **Disadvantages:** Uses more database space because the same value (literal or URI) is stored repeatedly.
- **Storage optimisations:** 
  - First, common prefixes in URIs, such as namespaces, are compressed. 
  - They are stored in a separate table and cached in memory.
  - Expanding the prefixes will be done in memory and will not require a database join.
  - Long values are stored only once.


## 4. Optimizations for common statement patterns
Applications typically have access patterns in which certain subjects and/or properties are accessed together. For example, a graph of data about persons might have many occurrences of objects with properties name, address, phone, gender that are referenced together. Using knowledge of these access patterns to influence the underlying database storage structures can provide a performance benefit.
### 4.1 Property-class table:
We could potentially cluster properties that are commonly accessed together and store it in a separate table as shown below:


![image](https://user-images.githubusercontent.com/22542670/31597024-ba147c24-b263-11e7-8118-f051c17423cf.png)

**Advantages:** 
- Note that property tables offer a small storage savings because the property URI is not stored in the table, and for clustered property tables, the subject is only stored once. 
- For some properties, the datatype of the object value will be fixed and known. It may be specified as a property range constraint. Property tables can leverage this knowledge by, when possible, making the underlying database column for the property value match the property type1. This may enable the database to better optimise the storage and searching of the column.


## 5. Performance evaluation
A synthetic database of 10,000 reified RDF statements was generated and stored in two different formats. In the first case, the reified statement was stored in an optimised form as a property-class table. In the second case, the reified statement was stored unoptimised as RDF triples, i.e., each reified statement was stored as four RDF statements. Consequently, the first table contained 10,000 rows while the second table contained 40,000 rows.
Each test was run four times with different random number seeds and three different test sizes were run of 200, 1000, 5000 retrievals. 
For a small number of retrievals, the optimised format shows a large improvement between the first and fourth run. We attribute this to caching effects that decrease with larger numbers of retrievals. The speed-up for large numbers of retrievals exceeds our expectations. This may be due to database caching effects. Since the optimised table is smaller, it is possible to cache a larger percentage of the entire table which reduces the number of relatively slow disk seek operations.

![image](https://user-images.githubusercontent.com/22542670/31597550-9aee6f82-b266-11e7-852f-1a6b819a1222.png)

# References: 
[https://www.cs.uic.edu/~ifc/SWDB/proceedings.pdf](https://www.cs.uic.edu/~ifc/SWDB/proceedings.pdf)
