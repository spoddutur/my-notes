
# Infamous Misconceptions on Solr
I have learned some interesting misconceptions listed below on Solr relevancy tuning while trying to improve search results. 
In this article, I'll be focusing mainly on:
1. Query Parsing Myths
2. TF-IDF Similarity Scoring Myths

## Myth1: On Query Parsing
Consider search query such as `q=red+apple&qf=title,description`. Usual perception for the above query is that it will return matching documents which contains the terms red and apple in title and description fields respectively. But in reality, which of the following two options will this actually translate/parsed to:

#### Option1: 
```markdown
+((title:blue | title:apple) 
(description:blue | description:apple))
````

#### Option2:
```markdown
+((title:red|description:red) 
    (title:apple | description:apple))
```
#### Myth Question:
**For `q=red+apple&qf=title,description` user query, which of the above two options does solr translate it to? **
<br/> Obviously, each of these query interpretations mean different and will generate different search results. If you also don’t have the clarity on which of the above two queries solr generates, then go ahead and read the rest of the article.

