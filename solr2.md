# How Solr Query Parser Advanced

This article is an attempt to give a preview of how solr query parser progressed from just simple Conjunction/Disjunction parsing today's graph representations of token streams. For this, let’s go little back into history. 
We'll use below query to understand the different variants of query parsings as it evolved:
```markdown
q=red apple&qf=title,description
```

## Original Lucene Query Parser
The original Lucene query parser would parse our above query using simple conjunction(i.e., AND)/Disjunction(i.e., OR) like this:
#### Simplest original parsing:
```markdown
(title:red OR description:apple) OR (text:red OR description:apple)
```
#### Cons: 
- A document that matches term `red` in two fields will end up getting the same score as a document that matches BOTH red and apple. 
- This is because `two mentions of term red` and `one red and one apple mention` both will get 2 hits for `(title:red OR description:apple) OR (text:red OR description:apple)` query.
- What users typically expect is documents with `red apple’s`, not just `red things` nor just `apple things`! 
http://mail-archives.apache.org/mod_mbox/lucene-solr-user/201703.mbox/%3cCALG6HL8W_cPeXCYnVKs2eSpDsTtcZ8_RbcYqWr+ZPoXwU5APPQ@mail.gmail.com%3e 

So DisjunctionMaxQuery a.k.a DisMax came about to bias towards results that had more of the user's search terms. DisMax takes per-term maximum and adds them together as shown below: 
(title:red | description:red) OR (title:apple | description:apple)
‘|’ operator shown in the above query takes max. Here the highest scored result will have BOTH search terms. So essentially a document that has both red and apple will come to the top. This strategy is coined as "term centric”
