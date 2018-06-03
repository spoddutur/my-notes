## Solr Explain and Common Misconceptions on TF-IDF
When you learn about relevancy ranking in Lucene-based search engines like Solr and Elasticsearch, **TF-term frequency** and **IDF-inverse document frequency** are the first things we get to know. They form the core of their default scoring formula. However, there are some common misconceptions that people have on TF-IDF.

### Common Misconceptions on TF-IDF scoring
- Documents containing the same TF and IDF values will get same scores.
- Documents containing higher TF will always get scored higher over documents having lower TF.

#### Let’s try and understand them in this article.
1. Root cause for these misconceptions
2. Quick peek into Lucene's Default TF-IDF scoring
3. DEMO
For this, let’s take a quick peek into lucene’s default tf-idf scoring.

### 1. Root cause for these misconceptions
One of the primary root cause for the misconceptions mentioned above is the following myth on tf-idf scoring:

**tf-idf scoring depends only on tf and idf.**

### 2. Quick peek into Lucene's Default TF-IDF scoringFact:
In reality, there are lot of other factors that tf-idf scoring depends on as shown below:

<img src="https://user-images.githubusercontent.com/22542670/40882365-2a65298a-66fd-11e8-990b-68b132567a2e.png"> </img>

As we can see in the figure shown above, there are other factors apart from tf and idf in this scoring formula. Let’s discuss them:

### 2.1 Scoring Factors
1. **coord(q,d)** - captures how many terms of query 'q' are found in document 'd'.  
2. **tf(t in d)** - returns the number of times a query term 't' appears in document 'd'.
3. **idf(term)** - returns the the number of documents in which the query term 't' appears
4. **queryNorm(q)** - is a normalizing factor used to make scores between queries comparable. This factor does not affect document ranking (since all ranked documents are multiplied by the same factor)
5. **t.getBoost** - search time boost specified by user for the query term t. 
6. **norm(t in d)** - encapsulates two things: fieldBoost and lengthNorm.
7. **fieldBoost** - boost specified to the field associated with term 't' in schema.xml and 
8. **lengthNorm** - captures length of the field matched. Shorter fields will get higher score and vice versa.

### 2.1 Revisit Scoring factors with "Field Based" perception
**Note that Lucene is field based.** Hence each query term applies to a single field. Now, let's revisit above listed scoring factors again with this perception
1. **lengthNorm** i.e., Document length normalization is by the length of the certain field,
2. **norm(t in d)** actually means norm(field(t) in doc d) where field(t) is the field associated with term t.
3. **tf(t in d)** similarly means tf(field(t) in doc d
4. **idf(term)** also means idf(field(term))!!


