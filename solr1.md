
I have learned some interesting misconceptions listed below on Solr relevancy tuning while trying to improve search results. If you also have similar thinking, then this article is definitely for you.

## Myth1: On Query Parsing
Consider search query such as `q=red+apple&qf=title,description`. Usual perception for the above query is that it will return matching documents which contains the terms red and apple in title and description fields respectively. But in reality, which of the following two options will this actually translate/parsed to:
```markdown
+((title:blue | title:apple) 
(description:blue | description:apple))


+((title:red|description:red) 
    (title:apple | description:apple))
```
If you observe, the first option is more field-centric where as the second option is more term-centric. Obviously, each of these query interpretations mean different and will generate different search results. If you also don’t have the clarity on which of the above two queries solr generates, then go ahead and read the remaining article. Its indeed important to understand how does our query get interpreted by lucene search-engine.
