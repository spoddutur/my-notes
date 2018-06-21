# Relevancy Tuning with Field Boosts

### Problem statement: Myth
A naive lucene user might conveniently apply field boosts like **`qf=title^10 tags^7 description^1`** and assume that 
- Search results will be sorted with ```title``` matched documents taking highest precedence because of 10 times boost
- ```tags``` matched documents will come next due to 7 times boost and 
- ```description``` matched documents will get least precedence.

**Wrong Assumption**: One of the main reasons for above mentioned misconception is the assumption that, for such query, documents will get cumulative score of:
```(title-match-score)^10 + (tags-match-score)^7 + (description-match-score)^1```

### Improper Field Boosts can cause unexpected surprising results
In this article, am going to discuss about two facts related to field boosts and its impact on search results:
1. **Fact1:** Understand why `(title-match-score)^10 + (tags-match-score)^7 + (description-match-score)^1` scoring is wrong for `qf=title^10 tags^7 description^1` fieldboosts.
2. **Fact2:** Inspite of boosting `title` field higher than `tags` field, the final results might show 100’s of good `tags` matches followed by good `title` matches. Following example illustrates this case: 

With **`title^10 tags^7`** field boosts, a search on **`Handlooms`** can return documents in following ranked order :
1. National awardees of Indian handicrafts to present their art at Heimtextil India & Ambiente India 
3. See the best of Artificial Intelligence, 5D tech and more from IIT-D
2. Amazon India launches Weavesmart on its marketplace 
4. Weaving out of trouble: Handloom industry looks at Budget 2018 to solve woes

**Note:** Despite having `Handloom match in title field` which is boosted highest, `Doc4 is scored lower than other documents`. Why would `title` matched document get lower score over other documents inspite of title field having highest boost of 10? 

## Understand how scoring works
To understand the facts mentioned above, its really important to get an understanding on how scoring works.
### Lucene's Field-Based Scoring:
**Lucene’s tf-idf scores are field-based where-in each field is its own universe.**
Let’s attempt to understand this field-based scores better with a sample query shown below. 

### Our SearchQuery:
```
q=Chemotherapy Cancer
qf=title^10 tags^7
defType=dismax
```

### Wrong Scoring Expectation
```
(title-match-score)^10 + (tag-matching-score)^7 scoring for our query
```

### Scoring that happens in reality
```
max(chemotherapy-match-score-in-title^10, 
    chemotherapy-match-score-in-tags^7)
              + 
max(cancer-match-score-in-title^10,
      cancer-match-score-in-tags^7)
```

### Let's see why:
Now, let's understand how scoring is done in reality for this query starting from how this is parsed into lucene's syntax.
- **parsed_query:** 
```
   +(DisjunctionMaxQuery((title:Chemotherapy | tags:Chemotherapy))
     DisjunctionMaxQuery((title:Cancer | tags:Cancer))
```  
- **parsed_query_tostring:**
```
    +((title:Chemotherapy | tags:Chemotherapy)
     (title:Cancer | tags:Cancer))
```

### Analysis - How dismax scoring happens:
1. The parsed query generated above is called **term-centric query** i.e., searches for each of the user-query terms in the documents to bias the results having most query terms. (You can find details about field-centric vs term-centric in my article [here](https://spoddutur.github.io/my-notes/solr3)).
2. There are two terms in our search query: `Chemotherapy` and `Cancer`
3. For each term i.e., ```Chemotherapy``` and ```Cancer```, dismax computes per-field tf-idf scores and picks max out of them _**So, Dismax is essentially winner-takes-all behaviour with highest scoring field being the winner here**_.
4. Let's analyse Dismax query for term _Chemotherapy_:
   1. Query: ```(title:Chemotherapy | tags:Chemotherapy)```
   2. ```"|"``` operator denotes max operator of dismax
   3. Here, lucene does four things:
      1. Compute td-idf score for the term ```Chemotherapy``` in ```title``` and ```tags``` fields respectively
      2. Boosts title score 10 times
      3. Boosts tags score 7 times
      4. Now it picks the max among the two field's scores as shown below:
```
      tf-idf score for chemotherapy
         = 7.0710677 
      	= max of:
      	  7.0710677 = title:Chemotherapy, product of:(10.0=boost and 0.70710677=tf-idf)
      	  4.9497474 = tags:Chemotherapy, product of:(7.0=boost and 0.70710677=tf-idf)
```
5. Similarly Dismax Query for the term Cancer is computed  ```(title:Cancer | tags:Cancer)```
6. Now the final step of combining both Dismax queries: 
```
+((title:Chemotherapy | tags:Chemotherapy) (title:Cancer | tags:Cancer))
```
7. Here, we perform **SUM of (Chemotherapy DISMAX Score) and (Cancer DISMAX Score)**

### Either-Or Situation to pick Winner With DisMax:
1. We've see that Dismax picks max scoring field per term. 
2. Its clear winner-takes-all behaviour (i.e., only the winning field contributes to dismax scores).
3. This is because, Dismax was designed to solve **"Finding Needle in HayStack"**. Its not good in finding **"Hay in HayStack"**.
2. With this, we can see clearly:
   1. Why the expected scoring mentioned above is wrong and 
   2. Why in reality, the scoring is done the way it is:
```
max(chemotherapy-match-score-in-title^10, 
    chemotherapy-match-score-in-tags^7)
              + 
max(cancer-match-score-in-title^10,
      cancer-match-score-in-tags^7)
```

### TIE param - Wait, there's an option to still make the scoring match your expectation
Yes, there's an alternative to still meet the expectation mentioned above:
Dismax query supports [tie](https://lucene.apache.org/solr/guide/6_6/the-dismax-query-parser.html#TheDisMaxQueryParser-Thetie_TieBreaker_Parameter) param. With tie, dismax scoring happens like this:
```
dismax total score = max(field scores) + tie * sum(other field scores)
```
- Essentially, the tie parameter allows one to control how the lower scoring fields affects score for a given word.
- Without tie, only the winner dictates scoring results.
- A value of "0.0", which is the default value, makes the query a pure **disjunction max query** where only the maximum scoring sub query contributes to the final score. 
- A value of "1.0" makes the query a pure **disjunction sum query** where it doesn’t matter what the maximum scoring sub query is, because the final score will be the sum of the subquery scores. 
- **Recommended value**: Typically a low value, such as 0.1, is useful.

#### So far we've seen how scoring is done for dismax query. This clears one of the facts mentioned in the beginning. Now, with this knowledge, let's further deep dive to understand the second fact.

## Fact2
Inspite of boosting `title` field higher than `tags` field, the final results might show 100’s of good `tags` matches followed by good `title` matches.
#### Why could search results show 100's of good tag field matches before good title field matches??

### Diversity of Fields with Dismax
For this, we should understand the nature of dismax when applied on diverse fields. If dismax is applied on GRE and TOEFL score fields, results will always be sorted by GRE score and not by TOEFL because dismax picks max-score per field. In this case, we can pretty much guarantee that: 
```max(GRE(student), TOEFL(student)) == GRE(student)```

So, search results can list 100's of good GRE score students before listing good TOEFL score students. This is how the diversity of fields used in query could lead to one field's scores dominating search results.

The same thing can happen with ```tags``` and ```title``` fields where tags scores can be by default higher over title scores. Let's take a quick peek into the tf-idf scoring factors for both the fields:
- **tf**: both tags and title will most likely have term-frequency=1 (as there wont be any repetition of terms)
- **idf**: tags is likely to be more shorter and denser than title field. So, tags field is more likely to have better idf score over title field
- **field-length-normalization**: This is another important scoring-factors for text-based fields. Shorter the text, higher the score. **tags** field is very terse and pointed capturing aboutness of the document where as **title** field is more like short free-flow text. So, even here tags is the winner.

(**Note**: If you are interested in knowing other scoring-factors of TF_IDF, please refer to my article [here](https://spoddutur.github.io/my-notes/solr-explain))

With the scoring factors discussed above favoring tags field over title, it is possible that inspite of lower boost, ```tags``` field matches might still get scored higher over ```title``` field. Let's take an example to demo the same:
### Documents Data: Listed below are four document’s titles and tags in sorted order of their scoring.
```
Doc1: 
title: National awardees of Indian handicrafts to present their art at Heimtextil India & Ambiente India 
tags: handicrafts, handlooms

Doc2:
title: Amazon India launches Weavesmart on its marketplace 
tags: e-commerce, handlooms

Doc3: 
title: See the best of Artificial Intelligence, 5D tech and more from IIT-D
tags: AI, handlooms, energy
(Doc3 talks about new advancements in handlooms, energy consumption etc by IIT-D. So, it got the tag handloom)

Doc4:
title: Weaving out of trouble: Handloom industry looks at Budget 2018 to solve woes
tags: handloom, budget, economy, employment
```
- **SearchQuery:** Handlooms 
- **Sort order:** Doc1 got the highest score and Doc4 got the lowest

#### Analysis:
1. If we actually observe, Doc4 is the only document with Handlooms term in the title. 
2. Doc1, Doc2 and Doc3 have handlooms in tag
3. **tf=1** for all the docs
4. **idf** depends how many documents have Handlooms term in tags field and title field respectively. So, it varies based on the corpus. Let's assume, for this example, that in our corpus we have equal idf's for handloom term i.e., 30 documents have handloom tag and  30 documents have handloom mention in title, then:
    1. **Handloom match in title:** will have an idf of 1/30 as handloom appeared in title field for 30 documents.
    2. **Handloom match in tags:** will have lower idf of 1/30 as handloom appeared in tags field for 30 documents..
5. **field-length-normalization** is higher for tags field because its terser than title.
6. So, with above assumptions, tf & idf are same for both fields. field-length-normalization is the deciding winner and its  favoring tags field because its terse. shorte the field length higher the field-length-normalization is.
7. Therefore, documents with lesser tags will get higher scores (Doc1, Doc2 and Doc3).
8. Doc4 has Handloom mention in title. But title field is longer than tags. So, the field-length-normalization factor will be lower causing title-match score to be lesser than tags-match score. This is why its ranked lowest.

### Conclusion
- We've seen dismax is designed to find **"Needle in HayStack"** and not **"Hay in Haystack"**. So, only the winner will dictate scoring results.
- This is why, if the field having highest boost is not the winner, then it will have zero contribution in dismax scoring results. This might lead to surprising unexpected results.
- However, one can change dismax behaviour using tie param.
- With tie, `total score = max(field scores) + tie * sum(other field scores)`
- tie=1.0 parameter to the DisMax scoring, changes the total relevancy score of any given record to be the sum of contributing field scores.
- We've also illustrated how diversity of the fields impact dismax search results. We illustrated this with an example of how dismax search results can list 100's of good GRE score students before listing any good TOEFL score students. In other words, in this case, search results will be dominated with GRE students over TOEFL students.
- Hopefully, this article gave a better understand on how to play with field boosts next time when you are performing relevancy tuning of your search results. These are no hardset rules to tune relevancy because as we have seen, it depends on a lot of characteristics of your data corpus. These are more like guidelines to be aware of to make better choices and avoid surprises in your search results. Good luck with your relevancy Tuning!!
