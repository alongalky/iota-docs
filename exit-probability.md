# Algorithm for calculating exit distribution
The following is an algorithm for calculating the exit probabilities of all tips in the tangle. Exit probabilities are necessary in order to determine the confirmation confidence of transactions in the tangle. In addition, it could be used as an alternative (but equivalent) tip selection algorithm.

## Definitions
Let G=(V, E) be a DAG, and s∈V a vertex. For each v∈V, We define P(v) to be the probability that a weighted random walk starting at s will pass through v.

Let u,vV such that u approves v, we define T(v, u) to be the probability of walking from u to v. Note that this is independent of where we started the walk. We call this the _transition probability_.

### Transition probability
To know the transition probability we first have to calculate the [cumulative weight](cumulative.md). If w(v) is the cumulative weight of v, and A(v) is the set of direct approvers of V, then:

T(v,u) = e<sup>αw(u)</sup> / [Σ<sub>u∈A(v)</sub> e<sup>αw(u)</sup>]

*Implementation note*: this formula will cause numerical problems, since e<sup>n</sup>=∞ if n is large. The solution is to factor out the maximum weight in the sum, making all values <=1. Let's call W the largest weight:

W=max<sub>u∈A(v)</sub>{w(v)} 

Then the formula for T(v,u) becomes:

T(v,u) = e<sup>α(w(u)-W)</sup> / [Σ<sub>u∈A(v)</sub> e<sup>α(w(u)-W)</sup>]

Which can be computed without problems. In the case of the unweighted walk (α=0), the formula is simplified to:

T(v,u) = 1 / |A(v)|

## Outline
We first calculate the _future set of s_ F(s) by a graph traversal, after which we only deal with the induced subgraph G[F(s)]. This the set of all vertices which reference s.

The next step is to sort the subgraph in reverse topological order. This sorting would place s at the beginning, and the tips in the end.

Finally, we iterate over the sorted vertices, and update the direct approvers of each vertex by performing adding the probability of passing through the vertex times the probability of transitioning to the approver.

## Algorithm
Input: G'=(V', E'), and a starting vertex s∈V. This graph represents the main tangle.

1. Compute F(s) by a graph traversal (BFS / DFS), and let G(V,E) be the [induced subgraph](https://en.wikipedia.org/wiki/Induced_subgraph) G[F(s)]. Note that V=F(s)
1. Initialize P(v) ← 0, for all v∈V
1. P(s) ← 1
1. SortedVertices ← reverse [topological sorting](https://en.wikipedia.org/wiki/Topological_sorting) of G
1. For each v in SortedVertices:
    1. For each u with an edge from u to v:
          1. P(u) ← P(u) + (P(v) * T(v, u))
1. For any tip v, P(v) is its exit probability


## Proof of correctness (Sketch)
It can be shown inductively that each step in the iteration calculates the current vertex's P value correctly, assuming the P values of all previous vertices are correct.

The idea is that to pass through a vertex you have to walk to one of its approvees, and the walk onto it. The probability of this path is the chance to reach this particular approvee times the chance to transition from it to the vertex. Once all approvees are summed over, the P value is correct.

## Time complexity
1. Computing F(s) and sorting topologically are both O(|V|+|E|). However, in the special case of a tangle, each vertex has exactly two outgoing edges, which implies |E|<=2|V|. This means the runtime is actually O(|V|).
1. The loop also runs in O(|V|). Each u is updated a maximum of 2 times, once for each approvee.

All in all, this algorithm runs in linear time with respect to |V|.

## Space complexity
This is the interesting part. Computing the subgraph and sorting topologically both are O(|V|+|E|) in memory, and in the tangle case O(|V|).

Storing the P values is O(|V|). The overall space complexity is therefore O(|V|).
