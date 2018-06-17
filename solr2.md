# How Solr parses our Query

QueryParser is mainly responsible to:
1. **Tokenize user's query** which is a String and generate [TokenStream](https://github.com/apache/lucene-solr/blob/master/lucene/core/src/java/org/apache/lucene/analysis/TokenStream.java).
2. **Tranform TokenStream** to either filter, edit or add new tokens by applying analysers like shingles, synonyms, auto phrasing, taxonomies etc as per user's configuration on the field. 
3. **Generate [Lucene's Query](https://github.com/apache/lucene-solr/blob/master/lucene/core/src/java/org/apache/lucene/search/Query.java)** object out of TokenStream.

This article is an attempt to give a preview of how solr's QueryParser progressed from just simple parsing using mainly conjunction/disjunction operators to handling today's advanced graph-representation of TokenStreams i.e., those produced by analysers like SynonymGraphFilter etc. For this, let’s go little back into history and analyse following query as to how different versions of QueryParsers parsed it.
```markdown
q=red apple&qf=title,description
```

## 1. Original Lucene Query Parser

#### 1.1 Parsing Style:
The original Lucene query parser would parse our above query using simple Disjunction(i.e., OR) operator like this:
```markdown
(title:red OR title:apple) OR (description:red OR description:apple)
```

#### 1.2 Analysis - This style is more FIELD-CENTRIC!!
![image](https://user-images.githubusercontent.com/22542670/41508488-34fb01aa-7263-11e8-99d0-87e37cd70b6b.png)

#### Cons: 
- **User's Expectation:** Users typically expect documents with `red apple’s` to get higher score than documents having just `red things` or just `apple things`! 
- **Reality:** A document that has 2 mentions of term `red` in two fields will end up getting the same score as a document that talks about `red and apple`. 
- **Why is that so??** This is because **`two mentions of term red`** will get 2 hits. Likewise **`One mention of red and one mention of apple`** will also get 2 hits for **`(title:red OR description:apple) OR (text:red OR description:apple)`** query.
- What users typically expect is documents with `red apple’s`, not just `red things` nor just `apple things`! 
http://mail-archives.apache.org/mod_mbox/lucene-solr-user/201703.mbox/%3cCALG6HL8W_cPeXCYnVKs2eSpDsTtcZ8_RbcYqWr+ZPoXwU5APPQ@mail.gmail.com%3e 

#### Fix: 
- Change the parser to take per-term maximum to bias results towards documents containing more of user's search terms.
- **TADAAA - Came DisjunctionMaxQueryParser!!**

## 2. DisjunctionMaxQueryParser

#### 2.1 Objective:
Documents containing most user's search terms should get higher score.

#### 2.2 Parsing Style:
DisMax takes per-term maximum and adds them together as shown below: 
```markdown
+((title:red | description:red) (title:apple | description:apple))

+ => denotes disjunction or AND operator
| -> denotes max operator
(title:red | description:red) -> takes max tf-idf score for term `red` in title and description fields
(title:apple | description:apple) -> takes max tf-idf score for term `apple` in title and description fields
Its more **term-centric** i.e., looks for documents containing all users search terms.
```
#### 2.3 Pros:
Here the highest scored result will have BOTH search terms. So essentially a document that has both red and apple will come to the top. This strategy is coined as **term centri**.

#### 2.4 Cons:


## Appendix:

https://github.com/apache/lucene-solr/blob/master/solr/core/src/java/org/apache/solr/search/QueryParsing.java - parseOP() - default operator is OR
