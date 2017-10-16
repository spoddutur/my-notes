# Second Generation Semantic Web Vision: Relationship at the core of it
At the heart of second generation Semantic Web is Semantic Associations a.k.a Relationships. Relationships lend meaning to information, makes them understandable and actionable, and provides new and possibly unexpected insights. A fundamental shift in focus happened from `documents` and `entities within documents` to `discovering and reasoning about relationships`. 

The perspective of data as KNOWLEDGE was a critical movement in the world of BigData. Let's go little back and peek into the history:

### The emergence of a “Web of Data”:
The `Web of Data` has undergone rampant growth with the publication of large datasets – often as Linked Data - by public institutions around the world.  Varied range of players such as crowd sourcing projects like dbpedia, web companies like WWW, schema.org, open-data publishers like data.gov, data.worldbank.org have contributed in publishing open data. This development paved way for mainly two things: 
- **Semantic Web technologies** and 
- **Ontology:** Standard vocabularies to describe semantics a.k.a relationships of the data that helped in exposing, sharing and connecting data.

### 1.1 Semantic Web Technologies:
Accessing and consuming data on the Web should not require end users to be aware of where to find the data (which datasets) and how it is represented. This has thrust Semantic technologies into limelight.

**Semantic web vision:** Capture semantic association between entities (or resources) and allow user to formulate NLQ (Natural Language Queries) to find semantic association(s).

**Capturing Semantic associations between entities using RDF:**
To support NLQ, one should capture semantic associations between entities. Semantic associations essentially indicates how one entity (or resource) is related to other entity. Resource Description Framework (RDF) data model provides a framework for describing relationships among resources in terms of uniquely identified properties (attributes) and values.

**RDF Overview:** An overview of RDF is provided [here](https://spoddutur.github.io/my-notes/rdf-overview)

### 1.2 Ontology - Standard Vocabulary to describe relationships of the data
- **Problem:** One of the current challenges for accessing and consuming data on the Web is dealing with heterogeneity, i.e. data can be distributed and duplicated in different data stores, and described by different custom schema’s (or vocbulary). 
- **Solution:** Standardise these custom schema’s.
- **Ontology - Provides standard vocabulary per domain:** A formal naming and definition was given for the types, properties, and interrelationships of the entities for a specific domain. Web companies like www, schema.org and linked open vocabulary took initiatives and published standard reusable linked vocabulary ecosystem. The initiative was not only to collect vocabulary metadata but also to promote good practice and improve overall ecosystem quality. Below picture illustrates example ontology of Organisation domain:
![image](https://user-images.githubusercontent.com/22542670/31596678-aed42424-b261-11e7-9059-abb5a6bb8784.png)

## 3. Conclusion:
I hope this article gives a good perspective on why its importatnt to look beyond identifying entities in your data corpus. Mining relationships among the entities in your data brings forth a lot of new potential particularly in building semantic web out of data. RDF's and Ontology have evolved a long way into becoming industry standard to capture entities and their relationships.

However, there  are some flaws to keep in mind while mining ontology in your data (as listed below):
- Ontology learning systems are typically able to automate much of the ontology building (and sometimes maintenance) process, this comes at the expense of a loss of accuracy due to the replacement of human experts with algorithms which could be error-prone.
- Current ontology learning systems also throw away a substantial amount of information encoded within the textual content they are processing. For example, any entities not discovered as nodes during the ontology mining process have no known relationships to any other entities, regardless of whether those relationships were actually represented within the analyzed content (and just overlooked) during the ontology mining process.
- Furthermore, since most terms and phrases can take on alternate, nuanced meanings within different contexts, these nuanced meanings are often lost when representing terms and phrases as single nodes independent of the context in which they are used.
- Finally, a substantial amount of meaning is encoded in the nuanced linguistic representations present in a corpus of free-text content (terms, character sequences, term ordering, placement of words within phrases and paragraphs, and so on). Existing ontology creation approaches fail to adequately support the representation and scoring of relationships between these nuanced and complex interrelationships.

## 4. References
[https://www.cs.uic.edu/~ifc/SWDB/proceedings.pdf](https://www.cs.uic.edu/~ifc/SWDB/proceedings.pdf)
[https://ercim-news.ercim.eu/en96/special/linked-open-vocabularies](https://ercim-news.ercim.eu/en96/special/linked-open-vocabularies 
)

