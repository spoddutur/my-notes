# Information Retrieval via SemanticSearch on LinkedData

Semantic Search understands the intent of the searcher and the contextual meaning of terms to generate more relevant results thereby, improves accuracy of the search.
In this blog, I'll cover following topics:
1. Compare the traditional keyword search with semantic search
2. Challenges in semantic search
3. Solution1 - How to capture user intent
4. Solution2 - A more generic approach for capturing user intent
5. Conclusion

## Traditional Method for Document Retrieval:
Searching is done in four stages in classical search engines:
1. *Document indexing:*  Simply index documents :)
2. *Term weighting:* Importance of the terms used within the document are calculated with the help of term frequency.
3. *Similarity coefficients:* Documents and queries are represented by vectors of term weight.
4. *Retrieval:* Retrieval is done by cosine similarity. 

**Obvious disadvantages:**
- Search keywords must be precise
- Document with similar context but different term won’t be retrieved

Challenges in Semantic Search:
Term mismatch is the most concerning problem for effective information retrieval. In that, there are multiple kinds of problems namely:
1. **Vocabulary problem:** 
- The words on which the documents are indexed (vs) the words in user query are not same
2. **Synonymy:**
- Same words different meanings (Ex: “apple” as company [vs] fruit)
- Synonymy may result in a failure to retrieve relevant documents
- Decreases Recall
3. **Polysemy:**
- Different words with same meaning (Ex: “television” and “tv”)
- Polysemy may cause retrieval of erroneous or irrelevant documents 
- Decreases Precision of retrieval.
4. **Hypernymy and Hyponymy: **
- We can place a hypernym and its hyponyms in a hierarchy
- The more general hypernym above the hierarchy and the more specific hyponyms below as shown in the picture below
Example: https://user-images.githubusercontent.com/22542670/31303800-491fabb2-ab31-11e7-9ffb-d91b0d55a1eb.png

5. **Meronymy and Holonym:**
- A meronym is a word that denotes a part or member of something. 
- The opposite of a meronym is a holonym. 
- For example, finger is meronym of hand and hand is the homonym of finger 

Next, Iam going to talk about addressing Vocabulary problem. (continued in [part2](https://spoddutur.github.io/blog/semantic-search-2))

### References:
- https://www.redlinels.com/hypernyms-and-hyponyms/
- https://www.thoughtco.com/what-is-a-meronym-1691308
- https://www.thoughtco.com/what-is-a-meronym-1691308
- http://thescipub.com/PDF/jcssp.2015.361.371.pdf
- http://ceur-ws.org/Vol-992/paper3.pdf
