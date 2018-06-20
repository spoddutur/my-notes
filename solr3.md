### Problem statement:
A naive lucene user might conveniently apply field boosts like `**qf=title^10 tags^7 description^1**` and assume that search results will be sorted with title given the best i.e., 10 times importance, tags are given 7 times boost and description comes next.
