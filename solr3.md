# Relevancy Tuning with Field Boosts

### Problem statement:
A naive lucene user might conveniently apply field boosts like **`qf=title^10 tags^7 description^1`** and assume that search results will be sorted with ```title``` matched documents given 10 times boost, ```tags``` matched documents are given 7 times boost and description comes next.

### Discussion: Improper Field Boosts can cause unexpected surprising results
In this article, am going to discuss on how inspite of boosting `title` field higher than `tags` field, the final results might show 100’s of good `tags` matches followed by good `title` matches. Following example illustrates this case: 

With **`title^10 tags^7`** field boosts, a search on **`Chemotherapy & Cancer`** can return documents in following ranked order :
1. Cancer Treatment Options and Technologies
2. Cancer therapy advisor
3. Chemotherapy: What it is, what to expect, side effects, and outlook
4. Chemotherapy technology advancements

#### Why aren't 3rd and 4rth documents which seem more relevant ranked higher over 2nd?

### Reason behind this is:
Lucene’s tf-idf scores are field-based where-in each field is its own universe.
Let’s attempt to understand this field-based scores better. 

### Documents Data: Listed below are four document’s titles and tags.
```
Doc1: 
title: Cancer Treatment Options and Technologies
tags: cancer, chemotherapy, radiation

Doc2:
title: Cancer therapy advisor
tags: cancer, chemotherapy, radiation

Doc3: 
title: Chemotherapy: What it is, what to expect, side effects, and outlook
tags: chemotherapy

Doc4:
title: Chemotherapy technology advancements
tags: chemotherapy
```

### Our SearchQuery:
```
q=Chemotherapy Cancer
qf=title^10 tags^7
defType=dismax
```

### Parsed Query in Lucene's Syntax:
- **parserd_query:** 
```+((DisjunctionMaxQuery((title:Chemotherapy | tags:Chemotherapy)) DisjunctionMaxQuery((title:Cancer | tags:Cancer)))```
- **parserd_query_tostring:**
```+((title:Chemotherapy | tags:Chemotherapy) (title:Cancer | tags:Cancer))```

### Analysis:
1. The parsed query generated above is **term-centric query** i.e., searches for each user query terms in documents to bias the results having most query terms. (You can find details about field-centric vs term-centric in my article [here](https://spoddutur.github.io/my-notes/solr3)).
2. For each term i.e., ```Chemotherapy``` and ```Cancer```, dismax computes per-field tf-idf scores and picks max out of them. So the Dismax Query for our terms will look like this:
  1. Dismax Query for term Chemotherapy:  ```(title:Chemotherapy | tags:Chemotherapy)```
  2. Dismax Query for term Cancer:  ```(title:Cancer | tags:Cancer)```
3. Let's analyse Dismax query for Chemotherapy: ```(title:Chemotherapy | tags:Chemotherapy)```
  1. ```"|"``` operator denotes max operator of dismax
  2. Here, lucene does two things:
    1. Computes td-idf score for the term ```Chemotherapy``` in ```title``` and ```tags``` fields respectively
    2. Picks max-score among the two as shown below:
```
      7.0710677 
      	= max of:
      	  7.0710677 = title:Chemotherapy, product of:(10.0=boost and 0.70710677=tf-idf)
      	  4.9497474 = tags:Chemotherapy, product of:(7.0=boost and 0.70710677=tf-idf)
```
4. Similarly Dismax Query for term Cancer is computed  ```(title:Cancer | tags:Cancer)```
5. Now the final step of combining both Dismax queries: ```+((title:Chemotherapy | tags:Chemotherapy) (title:Cancer | tags:Cancer))```. Here, we perform **SUM of (Chemotherapy DISMAX Score) and (Cancer DISMAX Score)**

#### So far we've seen how scoring is done for dismax query. Now, with this knowledge, let's further deep dive to understand as to why could there be 100's of good tag field matches before good title field matches.

For this, we should understand the nature of dismax when applied on diverse fields. If dismax is applied on GRE and TOEFL score fields, results will always be sorted by GRE score and not by TOEFL because we can pretty much guarantee that: max(GRE(student), TOEFL(student)) == GRE(student). So, there'll be 100's of good GRE score students before good TOEFL score students. So, the diversity of fields used in query could lead to one field's score dominating search results.

The same thing can happen with ```tags``` and ```title``` fields where tags scores will be by default higher over title scores. This is because, for text-based fields, field-length plays an important role in tf-idf score. Shorter the text, higher the score.
- **tags** field is very terse and pointed capturing aboutness of the document. 
- **title** field is more like short text.
So naturally, ```tags``` field matches can be scored higher over ```title``` field.



