# Algorithm for calculating cumulative weight
The following is an algorithm for calculating the cumulative weight of all transactions in a subgraph in the tangle, comprised of all ancestors of some vertex `s`. This calculation is needed to perform the weighted random walk phase of the tip selection algorithm in Iota.

This algorithm is very similar to the currently implemented IRI rating function, but it uses set unions instead of sums in order to achieve the correct weight function.

## Definitions
Let G=(V, E) be a DAG. For each v∈V, We define A<sub>v</sub> to be the set of ancestors of v (those u∈V for which there is a path from u to v). By convention, v is also included in A<sub>v</sub>.

We define H(v)=|A(v)| to be the cumulative weight of v.

## Algorithm
Input: G=(V, E), and a starting vertex s∈V.

1. Compute A<sub>s</sub> by a graph traversal (BFS / DFS)
1. Initialize A<sub>v</sub> ← {v} for all v∈V\\{s}
1. Sort the vertices in A<sub>s</sub> in topological order
1. For each v∈A<sub>s</sub> (in topological order):
    1. For each u with an edge from v to u:
          1. A<sub>u</sub> ← A<sub>u</sub> ∪ A<sub>v</sub>
    1. CumulativeWeight[v] ← |A<sub>v</sub>|
    1. (Optimization) Free the memory taken by A<sub>v</sub>, as it will not be needed further

## Proof of correctness (Sketch)
It can be shown inductively that each step in the iteration calculates the current vertice's cumulative weight correctly, assuming the cumulative weight of all previous vertices is correct.

## Time complexity:
1. Computing A<sub>s</sub> and sorting topologically are both O(|A<sub>s</sub>|+|E(A<sub>s</sub>)|), where E(A<sub>s</sub>) are the edges in the sub DAG comprised of the vertics of A<sub>s</sub>.
1. The iteration is also O(|A<sub>s</sub>|+|E(A<sub>s</sub>)|), assuming the set unions are done in O(1).

All in all, this algorithm runs in linear time.

## Space complexity
This is the interesting part. Computing the subtree and sorting topologically both are O(|A<sub>s</sub>|+|E(A<sub>s</sub>)|) in memory, but the ancestors sets A<sub>v</sub> can get quite large. A naive worst case analysis is that each vertex has O(|A<sub>s</sub>|) ancestors, and so the memory used is O(|A<sub>s</sub>|<sup>2</sup>), which may be prohibitive.

This bound can be lowered by a factor: if v<sub>i</sub> is the i-th vertex in the topological ordering, it can have at most i ancestors: |A<sub>v<sub>i</sub></sub>|<= i. This means the total memory used is bound by the arithmetic series (1,2,3,4,...), and this puts the bound at O(|A<sub>s</sub>|<sup>2</sup>/2).

We can get a slightly better factor, by noticing that at the i-th step |A<sub>v</sub>|<=i for all v. This is because only vertices that have an index lower than i in the topological ordering may be included in the ancestor set. This means in the first step the bound is |A<sub>s</sub>|, in the second it's 2|A<sub>s</sub>|, and in the i-th step it's i|A<sub>s</sub>|. However, because of the optimization step, at the i-th step we only store |A<sub>s</sub>|-i sets. This puts the bound at i(|A<sub>s</sub>|-i). This is maximized at i=|A<sub>s</sub>|/2, where it reaches O(|A<sub>s</sub>|<sup>2</sup>/4).

We believe that actual bound is significantly lower, since we didn't take into account the tangle property that each vertex has at most 2 direct children. We present the challenge of finding it.

## Further discussion
1. We are looking to show that this algorithm is not memory-prohibitive for practical use in the tangle. To achieve that, we need to estimate the size of A<sub>s</sub> for a characteristic selected start vertex, which we have not yet done. This should be considered both theoretically and by looking at real tangle behavior.

1. We believe that space complexity upper bound is actually lower than O(|A<sub>s</sub>|^2), due to the tangle structure, but we have yet to find a proof for this claim.

1. For the case of selecting multiple starting points s<sub>1</sub>, s<sub>2</sub>, s<sub>3</sub>,... the algorithm can be modified in the following way: calculate the ancestors for each of the starting positions, and run the rest of the algorithm on the DAG made of A<sub>s<sub>1</sub></sub>∪A<sub>s<sub>2</sub></sub>∪A<sub>s<sub>3</sub></sub>∪...

1. Implementation in the IRI will not be hard, but will require proper testing to make sure the memory consumption does not overflow.