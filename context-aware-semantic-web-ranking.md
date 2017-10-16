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
 - 1. Universal and 
-  2. User-Defined

## 2. Universal Weights
Certain weights will influence a path rank regardless of the query or context of interest. We call them Universal Weights.
Let's discuss the different kinds of Universal Weights that contribute to the overall path rank:

### 2.1 Subsumption Weight
- **Intuition:** Assign more weights to more `specific` semantic associations because they convey more meaning than general associations.
Following figure depicts a class `Organization` and its subclasses. `Organization` being the highest class in hierarchy is the most general while `Political Organization` is a more specific organization. 

![image](https://user-images.githubusercontent.com/22542670/31599986-5db9e2a2-b272-11e7-985b-c49ffc6e31c4.png)

- **Computing subsumption weight of a path P:**
For this, we've to first compute component weight based on calss hierarchy. Component(c) is essentially any entity or property contained in path P.
Component weight of the _ith_ component in Path P is defined as follows:

![image](https://user-images.githubusercontent.com/22542670/31600020-77314f40-b272-11e7-9f21-4db4d2525d75.png)

According to the above formula, Democratic Political Organisation(c3) will have a component weight of 1 and Political Organisation(c2) will have a component weight of 0.6 as shown below:

![image](https://user-images.githubusercontent.com/22542670/31600022-7a172072-b272-11e7-939f-fc02073d31b0.png)

Given component weights, the subsumption weight of a path P is computed as shown below: 
![image](https://user-images.githubusercontent.com/22542670/31600025-7f0ac958-b272-11e7-8d34-452bf669c8ab.png)


where |c| is the total number of components in the path P excluding starting and ending entities.
Thus the Subsumption Weight of a path P is the product of all the component weights within P, normalised by the number of components in the path (to avoid bias in path length). 

- **Example:**
![image](https://user-images.githubusercontent.com/22542670/31600030-842fb7fe-b272-11e7-9a26-65baf7974bd9.png)

- Subsumption weight of Path e1 -> e2 -> e5  = 1/3 * (1/2 * 1/2 * 1/1) = 0.083
- Subsumption weight of Path e1 -> e3 -> e5  = 1/3 * (1/2 * 2/2 * 1/1) = 0.167
- Subsumption weight of Path e1 -> e4 -> e5  = 1/3 * (1/1 * 2/2 * 1/1) = 0.334

Thus, Subsumption Weight will assign higher weights to paths which have more specific meaning.
