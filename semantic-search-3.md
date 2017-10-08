# Information Retrieval via SemanticSearch on LinkedData

This blog is continuation of [part2](https://spoddutur.github.io/my-notes/semantic-search-2).. So far, we have covered:
1. How traditional keyword search works and how it falls behind in capturing user intent
2. We also saw the challenges in capturing the intent of the searcher and
3. Solution1 - How to capture user intent

In this blog, we are going to discuss a more generic solution for capturing user intent

## 4. Solution2 - A more generic approach for capturing user intent
- **Common point between Soltuion1 and Solution2:** They both expand user query with semantic synonyms. 
- **Difference**: In Solution1, for a given set of words in user query, we relied on WordNet vocabulary to add synonyms and generate expanded query. In Solution2, we don't rely on any such dictionary to get semantic synonyms. Let's see how its done below.
- **Assumption:** Assuming that data is stored as [linked data](http://linkeddata.org/)

## 4.1 Generic approach to get Semantic Synonyms
**Generic Query Expansion via Mapping keywords to LinkedData Resources:**
As the title suggests, in this generic method, to expand query we map user keywords to linkeddata resources and get its corresponding semantic synonyms from owl:class and rdf:property labels of the Dataset.

**Goal:** Given a keyword w, find representative resources in Linked Data.

**Input Dataset:** The dataset we operate on consists of a set of resources, R, stored as Linked Data,  where every resource R has a set of properties used to denote specific relationships between resources.

**Labelling Properties:** We have chosen a set of labelling properties, i.e. properties whose values are expected to be literals which might be worthwhile in identifying distinct concepts. Ex: _rdfs:Label, foaf:name, dc:title, skos:prefLabel, skos:altLabel, fb:type.object.name_.

**Method:** In order to find representative concepts, we construct from w (i.e., given keyword) an expanded set of keywords, Ew that improve the chance of finding the most fitting concept in the target vocabulary according to its labelling (under the Labelling Properties).

## 4.1.1 Algorithm to get semantic synonyms:
For every keyword w in user query, if a resource R exists for w in Dataset, then to get its semantic synonyms, explore the neighbours N of R such that:
- N is of type owl:class or rdf:property
- N is related to entity R via predefined Labelling relations.

The below picture illustrates how this computation happens for user query "Honda". Pick the neighbours who are related to Honda through one of the labelling relations and they should be of type owl:class or rdfs:property. 

Resulted semantic synonyms: Automotive, organisation, vehicle and engine. 
(Note: This is just a curated example to get an idea)

![image](https://user-images.githubusercontent.com/22542670/31314127-204e2e26-ac16-11e7-8d4f-eef86c9a5fe8.png)

### 4.1.2 Examples:
UserQuery and its corresponding semantic synonyms found by this approach:
- *honda*: dbpedia-owl:automotive, dbpedia-owl:organisation, dbpedia-owl:vehicle and dbpedia-owl:engine
- *spacecraft*: dbpedia-owl:spacecraft, dbpedia-owl:satellite, dbpedia-owl:missions and dbpedia-owl: Rocket
- *wife*:	dbpedia-owl:spouse, dbpedia-owl:person, dbpedia-owl:family and dbpedia-owl:sex
- *bass*:	dbpedia-owl:fish, dbpedia-owl:instrument, dbpedia-owl:note and dbpedia-owl:musical

### 4.1.3 Advantages:
- **Better Recall** by using class hierarchies and rules. For example, a query for water sports in Spain would return results in scuba diving, windsurf and other subclasses, in Cádiz, Málaga, Almería and other Spanish locations
- **Better precision** by reducing polysemic ambiguities using instance labels and classifications of concepts and documents

## 5. Conclusion: 
This generic approach uses semantic similarity to expand query. These expanded sets are more general than ‘synsets’ (sets of synonyms within dictionary-oriented terms) in terms of both, including a huge potential range of named entities and in the flexibility of the semantic relationships covered.

**Hybrid approach:** In general, a multi-strategy approach is recommended where this generic approach is used only after the lexical expansion with WordNet failed to give desired result.

## References:
- [http://dbpedia.org/page/Honda](http://dbpedia.org/page/Honda)
- [https://www.redlinels.com/hypernyms-and-hyponyms/](https://www.redlinels.com/hypernyms-and-hyponyms/)
- [https://www.thoughtco.com/what-is-a-meronym-1691308](https://www.thoughtco.com/what-is-a-meronym-1691308)
- [https://www.thoughtco.com/what-is-a-meronym-1691308](https://www.thoughtco.com/what-is-a-meronym-1691308)
- [http://thescipub.com/PDF/jcssp.2015.361.371.pdf](http://thescipub.com/PDF/jcssp.2015.361.371.pdf)
- [http://ceur-ws.org/Vol-992/paper3.pdf](http://ceur-ws.org/Vol-992/paper3.pdf)
