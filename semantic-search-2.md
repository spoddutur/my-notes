# Information Retrieval via SemanticSearch on LinkedData

This blog is continuation of [part1](https://spoddutur.github.io/my-notes/semantic-search-1)..
So far, we have covered:
1. How traditional keyword search works and how it falls behind in capturing user intent
2. We also saw the challenges in capturing the intent of the searcher

In this blog, am going to talk about addressing one of the challenges i.e., Vocabulary Problem. 

## 3. Solution1 - How to capture user intent
Let's see how can we capture user intent by addressing Vocabulary problem. Note that, this is the most natural and successful technique for this problem. 

### 3.1 Concern with Vocabulary problem:
We should capture the `Semantic (instead of the keywords)` of user query.

### 3.2 Addressing Vocabulary Problem via Query Expansion:
There are 2 steps involved here:
1. The indexers and the users need to use the related and synonym words instead of using the same words.
2. Expand the original query with suitable words that best capture the actual user intent 

### 3.3 How to expand the original query with suitable words? 
1. User inputs query in natural language
2. Use tools like StanfordParser to identify the noun phrases and other grammar in the query:
3. Related synonym sets of various words in the query are also obtained using Word Net API.
4. Add these words to the original query and form the new quer
5. The queries formed will be more refined and are sent to Search API which fetches the results related to the user query.
Following diagram depicts the same:
![image](https://user-images.githubusercontent.com/22542670/31303807-92da66c0-ab31-11e7-9078-6fb8d5626db8.png)

### 3.4 Example run

- Step 1 - **User Query:** `name of football clubs in EEFA`
- Step 2 - **Parsed words for this user query using Stanford Parser:**   
![image](https://user-images.githubusercontent.com/22542670/31304026-1f1d62e6-ab36-11e7-9245-bb212d4dae81.png)
- Step 3 - **Word Net Synonym words:** `list, soccer`
- Step4 - **Expanded Query:** `Name or list the football Soccer clubs in EEFA`

### 3.5 Advantages of semantic search over traditional keyword search:
- Tradional keyword search will not be able to understand the difference between: `USA Players in Catalan basket team Vs Catalan Palyers in USA teams`. Such cases are not a problem for semantic search.
- **Better Recall** by using class hierarchies and rules. For example, a query for water sports in Spain would return results in scuba diving, windsurf and other subclasses, in Cádiz, Málaga, Almería and other Spanish locations
- **Better precision** by reducing polysemic ambiguities using instance labels and classifications of concepts and documents

### 3.6 Some sample queries to realise the potential of Semantic search:
- Query-1 List the team names in EEFA
- Query-2 Events of olympics
- Query-2 Persons who won medal in olympics
- Query-4 Persons who won medal in chess
- Query-5 What are the upcoming sports events in Europe?
- Query-6 What is sports concepts?
- Query-7 200 meter players
- Query-8 Martial arts sports 

### 3.7 Disadvantages:
- These approaches are typically based on a single knowledge source such as WordNet or Wikipedia 
- Bound to the specific structure of the knowledge source assumed to be known a-priori.
- Highly restricted by the scope of the knowledge source (WordNet or wikipedia in this case).
- For a query like “bass”, this method may not detect the different senses for ambiguous keywords like: _dbpedia-owl:Fish, dbpedia-owl:Instrument, dbpedia-owl:instrument, dbpedia-owl:voice, dbpedia-owl:partner, dbpedia-owl:note, dbpedia-owl:Musical, dbpedia-owl:lowest etc_

Next, Iam going to talk about a more generic approach to expand the query which addresses some of the disadvantages mentioned above.. (continued in [part3](https://spoddutur.github.io/my-notes/semantic-search-3))

### References:
- [https://www.redlinels.com/hypernyms-and-hyponyms/](https://www.redlinels.com/hypernyms-and-hyponyms/)
- [https://www.thoughtco.com/what-is-a-meronym-1691308](https://www.thoughtco.com/what-is-a-meronym-1691308)
- [https://www.thoughtco.com/what-is-a-meronym-1691308](https://www.thoughtco.com/what-is-a-meronym-1691308)
- [http://thescipub.com/PDF/jcssp.2015.361.371.pdf](http://thescipub.com/PDF/jcssp.2015.361.371.pdf)
- [http://ceur-ws.org/Vol-992/paper3.pdf](http://ceur-ws.org/Vol-992/paper3.pdf)

