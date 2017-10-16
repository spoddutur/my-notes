# Context-Aware Semantic Association Ranking
A second generation “Semantic Web” is being realised as a scalable ontology-driven information system where the heterogenous 
data content is linked meaning with semantic metadata. It is not uncommon that the quest for entity associations surfaces too
many relationships between them thereby, making it equally important to rank them in order to find interesting and meaningful
relationships before presenting them to end user.

To rank the relevance of semantic associations, it is necessary to capture the context within which they are going to be 
interpreted and used. This is a different problem compared to ranking documents in traditional search-engines, where ranking is done 
based on number of references to them (TF-IDF). 

In this blog, we look at two things:
- Capturing users interests semantically through an ontology-based context specification language
- Using a ranking function incorporating user-defined semantics (e.g., context) and universal semantics (e.g., associations conveying more information)

## 1. Weight Assignments 
A path connecting two entities can contain many entities in between. Here, we'll look at how can we rank a path 
The semantic associations i.e., the path connecting two entities can have multiple entities/properties in between. 
To rank these paths, a rank function is defined which is constituted by a lot of intermediate weights assigned to edges connecting them.
The weights are particularly categorized into two types:
- 1. **Universal Weights**
- 2. **User-Defined Weights**

Let's look at these two weights in detail:
## 2. Universal Weights
Certain weights will influence a path rank regardless of the query or context of interest. We call them Universal Weights.
Let's discuss the different kinds of Universal Weights that contribute to the overall path rank:

### 2.1 Subsumption Weight
**Intuition:** Assign more weights to more `specific` semantic associations because they convey more meaning than general associations.

Following figure depicts a class `Organization` and its subclasses. `Organization` being the highest class in hierarchy is the most general while `Political Organization` is a more specific organization. 

![image](https://user-images.githubusercontent.com/22542670/31599986-5db9e2a2-b272-11e7-985b-c49ffc6e31c4.png)

**Computing subsumption weight of a path P:**
For this, we've to first compute component weight based on calss hierarchy. Component(c) is essentially any entity or property contained in path P.
Component weight of the _ith_ component in Path P is defined as follows:

![image](https://user-images.githubusercontent.com/22542670/31600020-77314f40-b272-11e7-9f21-4db4d2525d75.png)

According to the above formula, Democratic Political Organisation(c3) will have a component weight of 1 and Political Organisation(c2) will have a component weight of 0.6 as shown below:

![image](https://user-images.githubusercontent.com/22542670/31600022-7a172072-b272-11e7-939f-fc02073d31b0.png)

Given component weights, the subsumption weight of a path P is computed as shown below: 
![image](https://user-images.githubusercontent.com/22542670/31600025-7f0ac958-b272-11e7-8d34-452bf669c8ab.png)


where |c| is the total number of components in the path P excluding starting and ending entities.
Thus the Subsumption Weight of a path P is the product of all the component weights within P, normalised by the number of components in the path (to avoid bias in path length). 

**Example:**
![image](https://user-images.githubusercontent.com/22542670/31600030-842fb7fe-b272-11e7-9a26-65baf7974bd9.png)

- Subsumption weight of Path e1 -> e2 -> e5  = 1/3 * (1/2 * 1/2 * 1/1) = 0.083
- Subsumption weight of Path e1 -> e3 -> e5  = 1/3 * (1/2 * 2/2 * 1/1) = 0.167
- Subsumption weight of Path e1 -> e4 -> e5  = 1/3 * (1/1 * 2/2 * 1/1) = 0.334

Thus, Subsumption Weight will assign higher weights to paths which have more specific meaning.

## 3.User-Defined Weights 
Unlike Universal Path Weights, User-Defined weights are more specific to the query (or context).

### Path Length Weight:
Usually, user might be interested in the most direct path (i.e., shortest path) but there will be cases where one m
In some queries, a user may be interested in the most direct path (i.e., shortest paths). Yet in other cases a user may wish to find possibly hidden, indirect, or discrete paths (i.e., longer paths). . The latter may be more significant in domains where there may be deliberate attempts to hide relationships; for example, potential terrorist cells remain distant and avoid direct contact with one another in order to defer possible detection [10] or money laundering [1] involves deliberate innocuous looking transactions. Hence, the user should determine which Path Length influence, if any, should be used (largely domain dependent). We will now define the Path Length Weight. If a user wants to favor shorter paths, (5a) is used, where |c| is the number of components in the path P (excluding the first and last nodes). In contrast, if a user wants to favor longer paths (5b) is used.
