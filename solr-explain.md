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
**MYTH: tf-idf scoring depends only on tf and idf.**

### 2. Quick peek into Lucene's Default TF-IDF scoring:
In reality, there are lot of other factors that this scoring depends on as shown below:

![image](https://user-images.githubusercontent.com/22542670/40882365-2a65298a-66fd-11e8-990b-68b132567a2e.png)

As we can see in the figure shown above, there are other factors apart from tf and idf in this scoring formula. Let’s discuss them:

### 2.1 Scoring Factors
1. **coord(q,d)** - captures how many terms of query 'q' are found in document 'd'.  
2. **tf(t in d)** - returns the number of times a query term 't' appears in document 'd'.
3. **idf(term)** - returns the the number of documents in which the query term 't' appears
4. **queryNorm(q)** - is a normalizing factor used to make scores between queries comparable. This factor does not affect document ranking (since all ranked documents are multiplied by the same factor)
5. **t.getBoost** - search time boost specified by user for the query term t. 
6. **norm(t in d)** - encapsulates two things: fieldBoost and lengthNorm.
  - **fieldBoost** - boost specified to the field associated with term 't' in schema.xml and 
  - **lengthNorm** - effects score in accordance with the number of tokens of this field in the document. Shorter fields contribute more to the score and viceversa. For example, consider two documents with names "Alex John Smith" vs "Alex Smith". If you search for "Smith", document with "Alex Smith" will get higher lengthNorm.

### 2.1 "Field Based" perception
**Note that Lucene is field based.** Hence each query term is applied to a single field and so does the scoring factors. 
To understand this better, let's consider an example usecase of index on persons with `name` and `friend_names` fields. Now, consider the following two sample queries:

```Number of persons with name Alex VS number of persons having Alex as friend```

- **Search for people named Alex**: Documents containing the term `Alex` only in `name` field should get boosted for this query. 
- **search for people who have Alex as friend** Documents containing the term `Alex` only in `freinds` field should boost this query.
This is where scoring on a per-field-basis is very important. Without `field-based` perception, scoring factors like `tf` and `idf` will be same for both the queries.

### 2.2 Revisit Scoring factors with Field-Based perception
Now, let's revisit above listed scoring factors again with this new perception where each of the factors are evaluated on the matching field as shown below:
1. **norm(t in d)** actually means norm(field(t) in doc d) where field(t) is the field associated with term t.
2. **tf(t in d)** similarly means tf(field(t) in doc d
3. **idf(term)** also means idf(field(term))!!
4. **lengthNorm** i.e., Document length normalization is by the length of the certain field,


