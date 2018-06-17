
# Infamous Misconceptions on Solr
I have learned some interesting misconceptions listed below on Solr relevancy tuning while trying to improve search results. 
In this article, I'll be focusing mainly on:
1. Query Parsing Myths
2. TF-IDF Similarity Scoring Myths

## Myth1: On Query Parsing
Consider search query such as `q=red+apple&qf=title,description`. Usual perception for the above query is that it will return matching documents which contains the terms red and apple in title and description fields respectively. But in reality, which of the following two options will this actually get parsed to:

![image](https://user-images.githubusercontent.com/22542670/41504841-8fb771aa-7218-11e8-9b06-a83a6dceca70.png)

Obviously, each of these query interpretations mean different and will generate different search results. If you also don’t have the clarity on which of the above two queries solr generates, then go ahead and read the rest of the article.

## Myth2: On Scoring with FieldBoosts
Another common misconception that I noticed is a general belief that queries with field boosts like `qf=description^1 title^5 ` will assign cumulative score of `(title-match-score)^5 + (description-match-score)^1` to the document. **But in reality, this is not how documents get scored.**

## Myth3: On TF-IDF Similarity scoring
In Lucene, TF (Term Frequency) and IDF (Inverse Document Frequency) form the core of default tf-idf similarity scoring. Following lists three common myths that I observed on tf-idf:
- **Myth 3.1:** TF is very commonly misinterpreted as a strong indicator of document relevancy. This is a wrong interpretation in some cases.
- **Myth 3.2:** Documents having higher mentions of search terms will always get higher score. This also might not be true always.
- **Myth 3.3:** Documents containing the same TF and IDF values will always get same score. Again, this is another common fallacy.

Let’s take a look at each of myths and understand them better.
