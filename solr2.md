# How Solr parses our Query

QueryParser is mainly responsible to:
1. **Tokenize user's query** which is a String and generate [TokenStream](https://github.com/apache/lucene-solr/blob/master/lucene/core/src/java/org/apache/lucene/analysis/TokenStream.java).
2. **Tranform TokenStream** to either filter, edit or add new tokens by applying analysers like shingles, synonyms, auto phrasing, taxonomies etc as per user's configuration on the field. 
3. **Generate [Lucene's Query](https://github.com/apache/lucene-solr/blob/master/lucene/core/src/java/org/apache/lucene/search/Query.java)** object out of TokenStream.

### All of this is fine, but, **Does Solr generate TERM-CENTRIC Query or FIELD-CENTRIC QUERY?**
For a user query like **`q=red+apple&qf=title,description`**, where user is looking for documents talking about **`red apples`**, which of the following two options does Solr QueryParser parse it to?

![image](https://user-images.githubusercontent.com/22542670/41504841-8fb771aa-7218-11e8-9b06-a83a6dceca70.png)

This article is an attempt to understand answer for above question. For this, let’s go little back into history and analyse  the same **`red apple`** query and see as to how different versions of QueryParsers will parse it.

## 1. Original Lucene Query Parser

### 1.1 Parsing Style:
The original Lucene query parser would parse our above query using simple Disjunction(i.e., OR) operator like this:
```markdown
(title:red OR title:apple) OR (description:red OR description:apple)
```

### 1.2 Analysis - This style is more FIELD-CENTRIC!!
![image](https://user-images.githubusercontent.com/22542670/41508567-a4c539a0-7264-11e8-9503-5a2933fa2f60.png)

### 1.3 Cons: 
**`Documents talking about either red things or apple things might get higher score over documents talking about both red-apple's`**
- **User's Expectation:** Users typically expect documents with ```red apple’s``` to get higher score than documents having just ```red things``` or just ```apple things```! 
- **Reality with this approach:** A document that has 2 mentions of term ```red``` in two fields will end up getting the same score as a document that talks about ```red and apple```. 
- **Why is that so??** This is because **`two mentions of term red`** will get 2 hits. Likewise **`One mention of red and one mention of apple`** will also get 2 hits in this case.
- What users typically expect is documents with ```red apple’s```, not just ```red things``` nor just ```apple things```! 

### 1.4 Fix: 
- Give importance to terms i.e., **Go Term-Centric from Field-centric**
- **`For this, a new parser was added to take per-term maximum to bias results towards documents containing more of user's search terms.`**
- **`TADAAA - Came DisjunctionMaxQueryParser!!`**

## 2. DisjunctionMaxQueryParser

### 2.1 Objective:
Documents containing most user's search terms should get higher score.

### 2.2 Parsing Style:
DisMax takes per-term maximum and waeves them together as shown below: 
```markdown
(title:red | description:red) OR (title:apple | description:apple)

where
OR operator denotes disjunction and
| operator denotes max operator
```

### 2.3 Analysis:
![image](https://user-images.githubusercontent.com/22542670/41509173-97516c68-726d-11e8-841a-c04874715560.png)

### 2.3 Pros:
Here the highest scored result will have BOTH search terms. So essentially a document that has both red and apple will come to the top. This is **term centric** strategy.

### 2.4 Cons:
If u notice in the above workflow, the user query is first tokenised by whitespace and then passed down to field analysers. Splitting user query before applying analysers will break some other expected behaviours like multi-word synonyms, shingles and n-gram analysers at query-time. This is because these analysers can't see across whitespace boundaries. For example:
- Consider **`q=united states of america`** query with SynonymFilter having **`usa,america, united states of america`** entry.
- **Tokenize before field analyzers**:  Now, if we tokenize i.e., split by whitespace before applying SynonymFilter, then **it will see 4 different tokens** ```united```, ```states```, ```of``` and ```america```. So, this way, SynonymFilter will not match any of these 4 tokens to any synonyms because it has synonym entry for ```united states of america``` as one single token.
- **Tokenize after field analyzers:** If we apply SynonymFilter before tokenizer, then it'll see ```united states of america``` mention and **appropriately add its synonyms** ```usa``` and ```america``` to user query.
- So, splitting query into tokens before applying analysers will break some expected behaviours

### 2.5 Fix:
Based on his query needs, user should be given control on whether tokenisation should happen before or after applying field analysers. For this purpose, as part of [SOLR-9185 change in Solr 6.5](https://lucene.apache.org/solr/guide/6_6/the-extended-dismax-query-parser.html#TheExtendedDisMaxQueryParser-ThesowParameter),  a parameter called ```sow a.k.a split-on-whitespace``` is introduced.

## 3. SOW - Split on Whitespace param for ExtendedDisMaxQueryParser

### 3.1 sow:
This parameter decides whether to split each of the query terms by whitespace or not before applying field analysers/filters.
- **sow=true:** When true, parser will tokenize/split the terms in user query by space and invokes text-analysis speparately for each individual token. In this case, query parsing will be term-centric as discussed above in Dismax parser section.
- **sow=false:** When false, parser will first pass the terms for analysis before splitting them into tokens. sow=false will follow below algorithm:
    1.	For each of the query fields (i.e., title and description in our case), build field-centric queries based on each field’s analyzers & settings.
    2.	Each of the generated field-centric queries are put together attempting to build a single term-centric query.
Query parsing with sow=false can flip in surprising ways between term-centric to field-centric. To illustrate this better, let’s take examples for each case.

### 3.2 Case1: query fields having different settings
1. **Query:** ```q=the red apple&qf=title,description&sow=false```
2. **```title``` field setting:** has StopWord filter with “the” as stop-word
3. **```description``` field setting:** None
4. **Parsed query:**  Solr parsed this and generated following query
```markdown
(title:red | title:apple) OR (description:the | description:red | description:apple)
```
5. **Analysis:** Each clause is picking best match per field. **`field-centric!!`**

### 3.3 Case2: query fields share same settings
1. **Query:** ```q=usa foreign policies&qf=title,description&sow=false```
2. **```title``` and ```description``` field settings:**
    1. has Synonym analyser with `“usa, unites states, america”` entry
    2. autoGeneratePhraseQueries=true
3. **Parsed query:** Solr parsed this and generated following query
```markdown
((title:usa title:”united states” title:america) | (description:usa description:”united states” description:america)) OR
    (title:foreign | description:foreign) OR
    (title:policies | description:policies)
```
4. **Analysis:** Each dismax clause is picking the best match per term. **`term-centric!!`**

### 3.4 Sow Analysis:
1. With sow, user got more control and flexibility to change query parser behaviour by enabling different query-time field analysers such as autoGeneratePhraseQueries, synonyms, stopwords etc.
2. We have also seen how the resulting parsed query could flip between term-centric and field-centric based on field settings.
3. In general, we’ve observed that ```sow=false``` setting along with ```autoGeneratePhraseQueries=true to handle synonyms ``` creates in general a saner term-centric query.
 
## 4. Conclusion:
1. Term-Centric: Biases documents containing most user's search terms will get higher score.
2. Field-Centric: Chooses documents with best matching field.
![image](https://user-images.githubusercontent.com/22542670/41510228-720897fe-727e-11e8-9361-d3d94ea889ec.png)

3. We've seen that Solr can flip between generating term-centric VS field-centric queries depending on our syntax, field settings, analysis chain and query fields.
4. **Recommendation: Keep It Simple**. In my experience, the term-centric syntax that comes with autoGeneratePhraseQueries and sow=false provides pretty good default search results. 
5. Note that Solr also promoted the usage of `sow=false` by changing its default value to false in [Solr 7.0](https://lucene.apache.org/solr/guide/7_0/major-changes-in-solr-7.html). Until then, `sow`'s default value was true.
5. One can always consider trial and errors with solr-explain i.e., by adding debugQuery=true
param to our query and see the details of how our queries are transformed into Lucene syntax.
