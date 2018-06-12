# How Solr Query Parser Advanced

This article is an attempt to give a preview of how solr query parser progressed from just simple Conjunction/Disjunction parsing today's graph representations of token streams. For this, let’s go little back into history. 

### Query used to evaluate different version of solr query parser
We'll use below query to understand the different variants of query parsings as it advanceded:
```markdown
q=red apple&qf=title,description
```

## 1. Original Lucene Query Parser

#### 1.1 Parsing Style:
The original Lucene query parser would parse our above query using simple conjunction(i.e., AND)/Disjunction(i.e., OR) like this:
```markdown
(title:red OR title:apple) OR (description:red OR description:apple)

**Analysis**
(title:red OR title:apple) -> looks for red and apple terms in title field
(description:red OR description:apple) -> looks for red and apple terms in description field
Its more **field-centric** i.e., looks for users search terms per field.
```
#### Cons: 
- **User's Expectation?** users typically expect documents with `red apple’s` to get higher score than documents having just `red things` or just `apple things`! 
- **Reality?** A document that has 2 mentions of term `red` in two fields will end up getting the same score as a document that talks about `red and apple`. 
- **Why is that so??** This is because **`two mentions of term red`** will get 2 hits. Likewise **`One mention of red and one mention of apple`** will also get 2 hits for **`(title:red OR description:apple) OR (text:red OR description:apple)`** query.
- What users typically expect is documents with `red apple’s`, not just `red things` nor just `apple things`! 
http://mail-archives.apache.org/mod_mbox/lucene-solr-user/201703.mbox/%3cCALG6HL8W_cPeXCYnVKs2eSpDsTtcZ8_RbcYqWr+ZPoXwU5APPQ@mail.gmail.com%3e 

#### Fix: 
- Change the parser to take per-term maximum to bias results towards documents containing more of user's search terms.
- **TADA - Came DisjunctionMaxQueryParser!!**

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

