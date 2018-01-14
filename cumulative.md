# Algorithm for calculating cumulative weight
The following is an algorithm for calculating the cumulative weight of all transactions in a subgraph in the tangle, comprised of all ancestors of some vertex `s`. This calculation is needed to perform the weighted random walk phase of the tip selection algorithm in Iota.

This algorithm is very similar to the currently implemented IRI rating function, but it uses [set unions](https://en.wikipedia.org/wiki/Union_(set_theory)) instead of sums in order to achieve the correct weight function.

## Definitions
Let G=(V, E) be a DAG. For each v∈V, We define F(v) to be the set of ancestors of v (those u∈V for which there is a path from u to v). In the tangle context, F(v) contains all the *future* transactions of v which approve it, as defined in the [Equilibria paper](https://arxiv.org/abs/1712.05385).

We define H(v)=1+|F(v)| to be the cumulative weight of v. In other words, the cumulative weight of a vertex is the number of ancestors it has plus one.

## Outline
We first calculate F(s) by a graph traversal, after which we only deal with the induced subgraph G[F(s)].

The next step is to sort the subgraph topologically. This sorting would place the tips first, and s at the very end. Each vertex is preceded by all its ancestors.

Finally, we iterate over the sorted vertices, and update the direct children of each vertex by performing a set union. In the tangle case, there are two such children. To each child's ancestor set we append the current vertex's parents, and the vertex itself. Following this method, by the time we reach a vertex, its ancestor list is complete and its cumulative weight may be calculated.

## Algorithm
Input: G'=(V', E'), and a starting vertex s∈V. This graph represents the main tangle.

1. Compute F(s) by a graph traversal (BFS / DFS), and let G(V,E) be the [induced subgraph](https://en.wikipedia.org/wiki/Induced_subgraph) G[F(s)]. Note that V=F(s)
1. Initialize F(v) ← { } for all v∈V
1. SortedVertices ← [topological sorting](https://en.wikipedia.org/wiki/Topological_sorting) of G
1. For each v in SortedVertices:
    1. For each u with an edge from v to u:
          1. F(u) ← F(u) ∪ F(v) ∪ {v}
    1. CumulativeWeight[v] ← |F(v)|
    1. (Optimization) Free the memory taken by F(v), as it no longer used

## Proof of correctness (Sketch)
It can be shown inductively that each step in the iteration calculates the current vertice's cumulative weight correctly, assuming the cumulative weight of all previous vertices is correct.

## Time complexity
1. Computing F(s) and sorting topologically are both O(|V|+|E|). However, in the special case of a tangle, each vertex has exactly two outgoing edges, which implies |E|<=2|V|. This means the runtime is actually O(|V|).
1. The loop also runs in O(|V|), assuming the [set unions are done in O(1)](https://en.wikipedia.org/wiki/Disjoint-set_data_structure#Time_complexity).

All in all, this algorithm runs in linear time with respect to |V|.

## Space complexity
This is the interesting part. Computing the subgraph and sorting topologically both are O(|V|+|E|) in memory, and in the tangle case O(|V|). However, the ancestor sets F(v) can get quite large.

Let us define S=Σ<sub>v∈V</sub> |F(v)|, the sum of the cardinalities of all ancestor sets. A naive worst case analysis implies an upper bound of S<=|V|<sup>2</sup>, since each vertex has at most |V| ancestors.

This bound can be lowered by a factor: if v<sub>i</sub> is the i-th vertex in the topological ordering, it can have at most i ancestors: |F(v<sub>i</sub>)|<=i. This means the total memory used is bound by the arithmetic sum 1+2+3+4+...+|V|, and this puts the bound at |V|(|V|+1)/2≈|V|<sup>2</sup>/2.

We can get a slightly better factor, by noticing that at the i-th step |F(v)|<=i for all v. This is because only vertices that have an index lower than i in the topological ordering may be included in the ancestor set. This means in the first step the total memory usage is bound by |V|, in the second by 2|V|, and in the i-th step by i|V|. However, because of the optimization step, at the i-th step we only store |V|-i ancestor sets. This puts the bound at i(|V|-i). This expression is maximized at i=|V|/2, where it reaches |V|<sup>2</sup>/4.

We believe that actual bound is significantly lower, since we didn't take into account the tangle property that each vertex has at most 2 direct children. We are open to ideas for improvement.

## Further discussion
1. We are looking to show that this algorithm is not memory-prohibitive for practical use in the tangle. To achieve that, we need to estimate the size of F(s) for a characteristic selected start vertex, which we have not yet done. This should be considered both theoretically and by looking at real tangle behavior.

1. We believe that space complexity upper bound is actually lower than O(|F(s)|^2), due to the tangle structure, but we have yet to find a proof for this claim.

1. For the case of selecting multiple starting points s<sub>1</sub>, s<sub>2</sub>, s<sub>3</sub>,... the algorithm can be modified in the following way: calculate the ancestors for each of the starting positions, and run the rest of the algorithm on the DAG made of F(s<sub>1</sub>)</sub>∪F(s<sub>2</sub>)∪F(s<sub>3</sub>)∪... Doing so will save a signficant amount of redundant calculation.

1. Implementation in the IRI will not be hard, but will require proper testing to make sure the memory consumption does not overflow.