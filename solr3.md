# Relevancy Tuning with Field Boosts

### Problem statement:
A naive lucene user might conveniently apply field boosts like **`qf=title^10 tags^7 description^1`** and assume that 
- Search results will be sorted with ```title``` matched documents taking highest precedence because of 10 times boost
- ```tags``` matched documents will come next due to 7 times boost and 
- ```description``` matched documents will get least precedence.

**Wrong Assumption*:* One of the main reasons for above mentioned misconcetion is the assumption that, for such query, documents will get cumulative score of _**(title-match-score)^7 + (tags-match-score)^7 + (description-match-score)^1**_. 

### Discussion: Improper Field Boosts can cause unexpected surprising results
In this article, am going to discuss about two things:
1. **Myth1:** How above mentioned scoring is wrong and 
2. **Myth2:** Inspite of boosting `title` field higher than `tags` field, the final results might show 100’s of good `tags` matches followed by good `title` matches. Following example illustrates this case: 

With **`title^10 tags^7`** field boosts, a search on **`Handlooms`** can return documents in following ranked order :
1. National awardees of Indian handicrafts to present their art at Heimtextil India & Ambiente India 
3. See the best of Artificial Intelligence, 5D tech and more from IIT-D
2. Amazon India launches Weavesmart on its marketplace 
4. Weaving out of trouble: Handloom industry looks at Budget 2018 to solve woes

#### Why aren't 3rd and 4rth documents which seem more relevant ranked higher over 1st and 2nd?

## Understand how scoring works
To understand the myths mentioned above, its really important to get an understanding on how scoring works.
### Lucene's Field-Based Scoring:
Lucene’s tf-idf scores are field-based where-in each field is its own universe.
Let’s attempt to understand this field-based scores better with a sample query shown below. 

### Our SearchQuery:
```
q=Chemotherapy Cancer
qf=title^10 tags^7
defType=dismax
```

### Parsed Query in Lucene's Syntax:
- **parsed_query:** 
markdown```
   +(DisjunctionMaxQuery((title:Chemotherapy | tags:Chemotherapy))
     DisjunctionMaxQuery((title:Cancer | tags:Cancer))```
     
- **parsed_query_tostring:**
markdown```
    +((title:Chemotherapy | tags:Chemotherapy)
     (title:Cancer | tags:Cancer))```

### Analysis - How dismax scoring happens:
1. The parsed query generated above is **term-centric query** i.e., searches for each of the user-query terms in the documents to bias the results having most query terms. (You can find details about field-centric vs term-centric in my article [here](https://spoddutur.github.io/my-notes/solr3)).
2. For each term i.e., ```Chemotherapy``` and ```Cancer```, dismax computes per-field tf-idf scores and picks max out of them _**So, Dismax is essentially winner-takes-all behaviour with highest scoring field being the winner here**_.
3. Let's analyse Dismax query for term _Chemotherapy_:
  - Query: ```(title:Chemotherapy | tags:Chemotherapy)```
  - ```"|"``` operator denotes max operator of dismax
  - Here, lucene does two things:
      1. Compute td-idf score for the term ```Chemotherapy``` in ```title``` and ```tags``` fields respectively
      2. Boosts title score 10 times
      3. Boosts tags score 7 times
      4. Now it picks the max among the two field's scores as shown below:
```
      7.0710677 
      	= max of:
      	  7.0710677 = title:Chemotherapy, product of:(10.0=boost and 0.70710677=tf-idf)
      	  4.9497474 = tags:Chemotherapy, product of:(7.0=boost and 0.70710677=tf-idf)
```
4. Similarly Dismax Query for term Cancer is computed  ```(title:Cancer | tags:Cancer)```
5. Now the final step of combining both Dismax queries: 
```
+((title:Chemotherapy | tags:Chemotherapy) (title:Cancer | tags:Cancer))
```
6. Here, we perform **SUM of (Chemotherapy DISMAX Score) and (Cancer DISMAX Score)**

### Either-Or Situation to pick Winner With DisMax:
- As shown above, Dismax picks max scoring field per term.
```
max(chemotherapy-match-score-in-title^10, 
    chemotherapy-match-score-in-tags^7)
              + 
max(cancer-match-score-in-title^10,
      cancer-match-score-in-tags^7)
```
Hopefully, this clears why the common misconception of `(title-match-score)^10 + (tag-matching-score)^7` scoring for our query **`title^10 tags^7`** is incorrect.

#### So far we've seen how scoring is done for dismax query. This clears one of the myths mentioned in the beginning. Now, with this knowledge, let's further deep dive to understand the second myth which is:

## Myth2
### Why could there be 100's of good tag field matches before good title field matches.

### Nature of DisMax on Diverse Fields
For this, we should understand the nature of dismax when applied on diverse fields. If dismax is applied on GRE and TOEFL score fields, results will always be sorted by GRE score and not by TOEFL because dismax picks max-score per field. In this case, we can pretty much guarantee that: 
```max(GRE(student), TOEFL(student)) == GRE(student)```
So, there'll be 100's of good GRE score students before good TOEFL score students. This is how the diversity of fields used in query could lead to one field's scores dominating search results.

The same thing can happen with ```tags``` and ```title``` fields where tags scores will be by default higher over title scores. This is because, for text-based fields, field-length plays an important role in tf-idf score. Shorter the text, higher the score.
- **tags** field is very terse and pointed capturing aboutness of the document. 
- **title** field is more like short text.

So it is possible that inspite of lower boost, ```tags``` field matches might still get be scored higher over ```title``` field. Let's see with an example
### Documents Data: Listed below are four document’s titles and tags.
```
Doc1: 
title: National awardees of Indian handicrafts to present their art at Heimtextil India & Ambiente India 
tags: handicrafts, handlooms

Doc2:
title: Amazon India launches Weavesmart on its marketplace 
tags: e-commerce, handlooms

Doc3: 
title: See the best of Artificial Intelligence, 5D tech and more from IIT-D
tags: AI, handlooms, prosthetics, energy saving
(Doc3 talks about new advancements in handlooms, prosthetics, energy consumption etc by IIT-D. So, it got the tag handloom)

Doc4:
title: Weaving out of trouble: Handloom industry looks at Budget 2018 to solve woes
tags: handloom, budget, economy
```


