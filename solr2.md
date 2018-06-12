# How Solr Query Parser Advanced

This article is an attempt to give a preview of how solr query parser progressed from just simple Conjunction/Disjunction parsing today's graph representations of token streams. For this, let’s go little back into history. 

### Query used to evaluate different version of solr query parser
We'll use below query to understand the different variants of query parsings as it advanceded:
```markdown
q=red apple&qf=title,description
```

## Original Lucene Query Parser
The original Lucene query parser would parse our above query using simple conjunction(i.e., AND)/Disjunction(i.e., OR) like this:
```markdown
(title:red OR description:apple) OR (text:red OR description:apple)
```
#### Cons: 
- **User's Expectation?** users typically expect documents with `red apple’s` to get higher score than documents having just `red things` or just `apple things`! 
- **Reality?** A document that has 2 mentions of term `red` in two fields will end up getting the same score as a document that talks about `red and apple`. 
- **Why is that so??** This is because **`two mentions of term red`** will get 2 hits. Likewise **`One mention of red and one mention of apple`** will also get 2 hits for **`(title:red OR description:apple) OR (text:red OR description:apple)`** query.
- What users typically expect is documents with `red apple’s`, not just `red things` nor just `apple things`! 
http://mail-archives.apache.org/mod_mbox/lucene-solr-user/201703.mbox/%3cCALG6HL8W_cPeXCYnVKs2eSpDsTtcZ8_RbcYqWr+ZPoXwU5APPQ@mail.gmail.com%3e 

So DisjunctionMaxQuery a.k.a DisMax came about to bias towards results that had more of the user's search terms. DisMax takes per-term maximum and adds them together as shown below: 
(title:red | description:red) OR (title:apple | description:apple)
‘|’ operator shown in the above query takes max. Here the highest scored result will have BOTH search terms. So essentially a document that has both red and apple will come to the top. This strategy is coined as "term centric”
