# Context-Aware Semantic Association Ranking
A second generation “Semantic Web” is being realised as a scalable ontology-driven information system where the heterogenous 
data content is linked meaning with semantic metadata. It is not uncommon that the quest for entity associations surfaces too
many relationships between them thereby, making it equally important to rank them in order to find interesting and meaningful
relationships before presenting them to end user.

To rank the relevance of semantic associations, it is necessary to capture the context within which they are going to be 
interpreted and used. This is a different problem compared to ranking documents in traditional search-engines, where ranking is done 
based on number of references to them (TF-IDF). 

In this blog, we look at formulating a custom ranking function which incorporates user-defined semantics (e.g., context) and universal semantics (e.g., associations conveying more information) via weight-assignments.

## 1. Weight Assignments 
A path connecting two entities can contain many entities in between. Here, we'll look at how can we rank a path 
The semantic associations i.e., the path connecting two entities can have multiple entities/properties in between. 
To rank these paths, a rank function is defined which is constituted by a lot of intermediate weights assigned to edges connecting them.
The weights are particularly categorized into two types:
1. **Universal Weights**
2. **User-Defined Weights**

Let's look at these two weights in detail:
## 2. Universal Weights
Certain weights will influence a path rank regardless of the query or context of interest. We call them Universal Weights.
Let's discuss the one such kind of universal weight that contribute to the overall path rank:

### 2.1 Subsumption Weight
**Intuition:** Assign more weight to more `specific` semantic associations because they convey more meaning than general associations.

Following figure depicts a class `Organization` and its subclasses. `Organization` being the highest class in hierarchy is the most general while `Political Organization` is a more specific organization. 

![image](https://user-images.githubusercontent.com/22542670/31599986-5db9e2a2-b272-11e7-985b-c49ffc6e31c4.png)

**Computing subsumption weight of a path P:**
For this, we've to first compute component weight based on calss hierarchy where a component is any entity or property contained in path P.

1. Compute component weight of the _ith_ component in Path P is defined as follows:
<img style="border-width:2px" src="https://user-images.githubusercontent.com/22542670/31600020-77314f40-b272-11e7-9f21-4db4d2525d75.png" width="300"/>

According to the above formula, Democratic Political Organisation(c3) will have a component weight of 1 and Political Organisation(c2) will have a component weight of 0.6 as shown below:

<img src="https://user-images.githubusercontent.com/22542670/31600022-7a172072-b272-11e7-939f-fc02073d31b0.png" width="300"/>

2. Given component weights, the subsumption weight of a path P is computed as shown below: 
![image](https://user-images.githubusercontent.com/22542670/31600025-7f0ac958-b272-11e7-8d34-452bf669c8ab.png)


where |c| is the total number of components in the path P excluding starting and ending entities.
Thus , <img src="https://user-images.githubusercontent.com/22542670/31651781-9b97d62e-b33a-11e7-9380-4e380aea917a.png" width="20"/>
i.e., the Subsumption Weight of a path P is the product of all the component weights within P, normalised by the number of components in the path (to avoid bias in path length). 

**Example:**


![image](https://user-images.githubusercontent.com/22542670/31600030-842fb7fe-b272-11e7-9a26-65baf7974bd9.png)

- Subsumption weight of Path e1 -> e2 -> e5  = 1/3 * (1/2 * 1/2 * 1/1) = 0.083
- Subsumption weight of Path e1 -> e3 -> e5  = 1/3 * (1/2 * 2/2 * 1/1) = 0.167
- Subsumption weight of Path e1 -> e4 -> e5  = 1/3 * (1/1 * 2/2 * 1/1) = 0.334

Thus, Subsumption Weight will assign higher weights to paths which have more specific meaning.

## 3.User-Defined Weights 
Unlike Universal Path Weights, User-Defined weights are more specific to the query (or context).

### Path Length Weight:
- **Intuition:** Let path length influence rank of a path.
- **Computing rank of a path based on path length:**
If user wants to rank shortest paths high, then as shown in the figure below, (a) is used and if user wants to favor long paths then (b) is used. 

![image](https://user-images.githubusercontent.com/22542670/31605738-a363d044-b284-11e7-8b2b-2b4963f6f609.png)

Here, |c| is number of components in the path P (excluding first and last nodes).

- **Example:** 

Path length weight <img src="https://user-images.githubusercontent.com/22542670/31651778-95ffd36a-b33a-11e7-8486-3c2f44e787cc.png" width="20"/> in the below example for longer path will be 1/9 and that of direct path will be 1/1. Shorter path will get higher weight as expected. Alternatively, if we have to favor longer paths, then <img src="https://user-images.githubusercontent.com/22542670/31651778-95ffd36a-b33a-11e7-8486-3c2f44e787cc.png" width="20"/>
 of longer path = 1- (1/9 )= 0.889 and <img src="https://user-images.githubusercontent.com/22542670/31651778-95ffd36a-b33a-11e7-8486-3c2f44e787cc.png" width="20"/> for direct path = 1-(1/1) = 0.

![image](https://user-images.githubusercontent.com/22542670/31605741-aa53bb9e-b284-11e7-854f-af49ffe802b5.png)

### Trust Weight:
- **Intuition:** Rank trusted sources higher (e.g., Reuters could be regarded as a more trusted source on international news than some of the other news organizations)

- **Computing rank of a path based on source trust weight:**

![image](https://user-images.githubusercontent.com/22542670/31648494-aac7d0ea-b32b-11e7-8312-764c08bc5735.png)
 Trust weight of an overall path P is defined as product of trust weights of all properties in P (where ![image](https://user-images.githubusercontent.com/22542670/31648468-84a7b678-b32b-11e7-8711-fc694dd1400f.png))
 is the trust weight of _i_th property in the path P
 
![image](https://user-images.githubusercontent.com/22542670/31648413-2da49bb6-b32b-11e7-84c3-336544f02bf9.png)

## 4. Final Ranking criterion:
We will now define the overall path rank, using all the different weights discussed above.

![image](https://user-images.githubusercontent.com/22542670/31648403-1f0e9192-b32b-11e7-9c7d-67df7dc91874.png)
where all the _Ki_'s add up to 1.0 and are intended to allow fine-tuning of the different ranking criteria (e.g., trust can be given more weight than path length). 

## 5. Conclusion:
We have seen how we can plugin different ranking criteria's to score the semantic associations based on user's interest namely:
- Subsumption Weight <img width="20" src="https://user-images.githubusercontent.com/22542670/31651781-9b97d62e-b33a-11e7-9380-4e380aea917a.png"/> - How much meaning a semantic association conveys depending on the places of its components in the ontology
- Path Length Weight <img width="20" src="https://user-images.githubusercontent.com/22542670/31651778-95ffd36a-b33a-11e7-8486-3c2f44e787cc.png"/> - Allows preference of either immediate or distant relationships
- Trust Weight <img width="20" src="https://user-images.githubusercontent.com/22542670/31648494-aac7d0ea-b32b-11e7-8312-764c08bc5735.png"/> - determining how reliable a relationship is according to its provenance

One can tweak and add more ranking criteria's and come up with their own formula. This blog is to give an idea on how to go about devising ranking strategy. There are indeed a lot of other advanced ways to rank the results depending on your domain and usecase. Hope it helps!!
