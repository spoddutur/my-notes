# How Solr parses our Query

QueryParser is mainly responsible to:
1. **Tokenize user's query** which is a String and generate [TokenStream](https://github.com/apache/lucene-solr/blob/master/lucene/core/src/java/org/apache/lucene/analysis/TokenStream.java).
2. **Tranform TokenStream** to either filter, edit or add new tokens by applying analysers like shingles, synonyms, auto phrasing, taxonomies etc as per user's configuration on the field. 
3. **Generate [Lucene's Query](https://github.com/apache/lucene-solr/blob/master/lucene/core/src/java/org/apache/lucene/search/Query.java)** object out of TokenStream.

### All of this is fine, but, **Does Solr generate TERM-CENTRIC Query or FIELD-CENTRIC QUERY?**
For a user query like **`q=red+apple&qf=title,description`**, where user is looking for documents talking about **`red apples`**, which of the following two options does Solr parse it?

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
http://mail-archives.apache.org/mod_mbox/lucene-solr-user/201703.mbox/%3cCALG6HL8W_cPeXCYnVKs2eSpDsTtcZ8_RbcYqWr+ZPoXwU5APPQ@mail.gmail.com%3e 

### 1.4 Fix: 
- Give importance to terms i.e., **Go Term-Centric from Field-centric**
- **`For this, a new parser was added to take per-term maximum to bias results towards documents containing more of user's search terms.`**
- **`TADAAA - Came DisjunctionMaxQueryParser!!`**

## 2. DisjunctionMaxQueryParser

### 2.1 Objective:
Documents containing most user's search terms should get higher score.

### 2.2 Parsing Style:
DisMax takes per-term maximum and adds them together as shown below: 
```markdown
(title:red | description:red) OR (title:apple | description:apple)

where
OR operator denotes disjunction and
| operator denotes max operator
```

### 2.3 Analysis:
![image](https://user-images.githubusercontent.com/22542670/41509173-97516c68-726d-11e8-841a-c04874715560.png)

### 2.3 Pros:
Here the highest scored result will have BOTH search terms. So essentially a document that has both red and apple will come to the top. This strategy is coined as **term centric**.

### 2.4 Cons:
If u notice in the above workflow, the user query is first tokenised by whitespace and then passed down to field analysers. Splitting user query before applying analysers will break some other expected behaviours like multi-word synonyms, shingles and n-gram analysers at query-time. This is because these analysers can't see across whitespace boundaries. For example:
- Consider ```q=united states of america``` query with SynonymFilter having ```usa,america, united states of america``` entry.
- Now, if we tokenize i.e., split by whitespace before applying SynonymFilter, then we'll see 4 tokens ```united```, ```states```, ```of```, ```america```. This way, SynonymFilter will match these to any synonyms.
- If we apply SynonymFilter before tokenizer, then it'll see ```united states of america``` mention and appropriately add its synonyms ```usa``` and ```america``` to user query.

### 2.5 Fix:
Based on his query needs, user should be given control on whether tokenisation should happen before or after applying field analysers. For this purpose, as part of [SOLR-9185 change in Solr 6.5](https://lucene.apache.org/solr/guide/6_6/the-extended-dismax-query-parser.html#TheExtendedDisMaxQueryParser-ThesowParameter),  a parameter called ```sow a.k.a split-on-whitespace``` is introduced.

## 3. SOW - Split on Whitespace param for DisMax

### 3.1 sow:
This parameter decides whether to split each of the query terms by whitespace or not before applying field analysers/filters. 
- **sow=true:** When true, parser will split terms in user query by space before sending to analysis.
In this case, query parsing will be same as pre-Solr 6.5 discussed above.
- **sow=false:** When false, parser will first send the terms for analysis before generating tokens. sow=false will follow following algorithm:
    1.	For each field passed into qf (title, description above), build field-centric queries based on each field’s analysis & settings.
    2.	Each of the generated field-centric queries are put together attempting to build a single term-centric query.
Query parsing with sow=false can flip in surprising ways between term-centric to field-centric. To illustrate this better, let’s take examples for each case.

### 3.2 Case1: fields mentioned in qf having different settings
1. **Query:** ```q=the+red+apple&qf=title,description&sow=false```
2. **```title``` field setting:** has StopWord filter with “the” as stop-word
3. **```description``` field settings:** None
4. **Parsed query:** ```(title:red | title:apple) (description:the | description:red | description:apple)```
5. **Analysis:** Each clause is picking best match per field. **`field-centric!!`**

### 3.3 Case2: all fields mentioned in qf share same settings
1. **Query:** ```q=usa foreign policies&qf=title,description&sow=false```
2. **```title``` and ```description``` field settings:**
    1. has Synonym analyser with “usa, unites states, america” entry
	2. autoGeneratePhraseQueries=true
3. **Parsed query:**
markdown```((title:usa title:”united states” title:america) | (description:usa description:”united states” description:america)) OR
    (title:foreign | description:foreign) OR
    (title:policies | description:policies)```
4. **Analysis:** Each dismax clause is picking the best match per term. **`term-centric!!`**

## Appendix:

https://github.com/apache/lucene-solr/blob/master/solr/core/src/java/org/apache/solr/search/QueryParsing.java - parseOP() - default operator is OR
