# NGraph

A simple (Network) Graph library in C++


## Introduction

NGraph is a simple C++ class library for the analysis of small to medium-sized network graphs (e.g. social networks, web graphs, computer networks). The library provides functionality for creating and accessing graphs in a convenient way, with a short learning curve, and without complicated interfaces. Ngraph was designed for educational purposes.
Although a mathematical Graph can consist of any enumerated type (cities, colors, people, etc.) it is convenient to think of the nodes as non-negative integers {0, 1, 2, ... } with edges being a pair of nodes.

This version of NGraph implements a directed graph (which can also be used as the basis of a undirected graph by storing only edges (i,j) where i < j) without multiple edges or loops (edges from a vertex to itself). These conventions are not as severly limiting as they sound, as there are many simple workarounds to accomodate them.

Finally, it should be noted that NGraph is not a replacement or a substitute for more elaborate graph packages, like the Boost Graph Library (BGL), or the LEDA graph library. These packages are much more sophisticated and complex. NGraph, on the other hand, is a tiny library (several hundred lines of code) meant to demonstrate how one can harness C++ components (like sets and maps from Standard Template Library) to create useful data structures with just a few lines of code.


## Examples

To create a Graph, we call the insert_edge() function to build its edges:

```c++
using namespace NGraph;

Graph A;
A.insert_edge(3,4);
A.insert_edge(4,5);
...
A.insert_edge(0,1);
```

Note that because this is not a multi-graph (i.e. one that allows repeated edges) inserting that same edge twice has no effect.
We can see find out a graph's order and size by calling the num_vertices() and num_edges(). For example,

```c++
std::cout << "Graph has " << A.num_vertices() << " vertices and "
    << A.num_edges() << " edges.\n";
```

One can remove edges by calling remove_edge(), e.g.

```c++
A.remove_edge(4,5);
```

If the edge is not present in the graph, this operation has no effect. We can repeatedly remove edges from a graph until it is empty (i.e. its order is zero). We can also check to see if a given edge is present by calling contains_edge(), which returns a boolean value true if the edge exsits, and false otherwise. Likewise, we can remove vertices with the remove_vertex() function and it will also automatically remove all edges from the graph containing that vertex as well.
Once a graph is built, we may use it in expressions, for example:

```c++
Graph::vertex a = 4;    // a more meaningful type than "unsigned int" 
Graph::vertex b = 6;    //   
A.degree(a);            // return the degree at vertex a
A.in_degree(a);         // return the number of vertices that point to this one

Graph::vertex_set S = A.out_neighbors(a);   // return vertices reached through this one

A.subgraph(S);          // return the graph whose edge include all vertices in S

A + B;                  // return the union of two graphs

cout << A ;             // print out graph A onto standard output stream
```


## Iterating through Graphs

The above functionality is fine for simple operations with graphs. In theory, one could get by with these primitives. However, in practice, more sophisticated graph algorithms require to be able to traverse a graph quickly, and be able to access in-coming and out-going edges with each vertex efficiently.
A straightforward graph format is to maintain a list of the graph's edges as an (i,j) pair (referred to as coordinate list.) While universal scheme, it does not provide an efficient structure for finding vertex neighbors --one would need to scan the whole list to compute this.

Instead, modified graph structures are often implemented to better optimize such operations. One common method is an adjacency list, which is a list of neighbors for each vertex. Another approach would be to use some sort of hashing function, or clustering mechanism to optmize storage and/or accessing cost. Whatever the method used, we would like to able to use the Graph object in the same way. That is, we wouldn't want our application code to be completely dependent on the implementation details of the Graph library. In fact, we would like to keep this issue aside, so we can try several alternative strategies and potentially choose the best one for our case.

We need then, some methodology to walk (or traverse) through the graph in a way that does make any asummptions about its underlying structure. The Standard Template Library (STL) faced this very same problem and solve it quite elegantly with the concept of iterators. These pointer-like entities allow one to traverse a container without making many assumptions about the underlying implementation. A.begin() refers to the first node, and A.end() refers to the tail end of the container, while the ++ operator allows one to jump from one element to the next. We give an example below.

```c++
//  for each node in the graph...
//
for ( Graph::const_iterator p = A.begin(); p != A.end(); p++)
{
    int do = out_degree(p);
    int di = in_degree(p);

    // identify its neighbors
    //
    Graph::vertex_set Si = in_neighbors(p);
    Graph::vertex_set So = out_neighbors(p);

    // print out each out-going edge
    //
    for (Graph::vertex_set::const_iterator t = Si.begin(); t !=Si.end(); t++)
    {
        cout << *t << "\n";
    }
    cout << "\n";
}
```


## Another example: computing clustering coefficients

The cluster coefficient of a vertex gives an indication of how well a group of neighbors are connected. In a social network graph where the nodes are people and edges represent friendships, this number determines the probability that any two of your friends will know each other. Mathematically, it is the ratio of the number of edges formed by the subgraph of the neighboring divided by the total number of possible edges in its complete graph (i.e. where everyone know each other). This gives a real number somewhere between 0.0 and 1.0, with the higher the number, the more "friendly" the vertcies seem to be. We can compute the cluserting coefficient of node "a" as follows. First, we find out the neighboring verticies (the group of friends), then form a subgraph of this group with the graph. Lastly, the size of a complete graph is N*(N-1)/2, where N is the order, so we can write

```c++
double cluster_coeff(const Graph &A, const Graph::vertex &a)
{
   Graph::vertex_set &group_of_friends = A.out_neighbors(a);
   double  N = group_of_friends.size(); 

    return   A.subgraph(group_of_friends).size() / (N * (N-1));
}
```

(Note that we made the variable "N" a double so that the division expression would be performed as a floating point calculation, rather than integer one. Also, as we visit each node we count each edge twice, so we need to cut the numerator in half. This cancels with halving the denominator, so there is no "2" in computation.)


## Try other examples

There are other examples in the source distribtution, along with some documented code, but this short tutorial should get you started. The best way to learn the capabilities of Ngraph is to play around with it!
A collection of tools and applications based on NGraph can be found at the Network Graph Toolkit page. 

This software was developed at the National Institute of Standards and Technology (NIST) by employees of the Federal Government in the course of their official duties. Pursuant to title 17 Section 105 of the United States Code this software is not subject to copyright protection and is in the public domain. NIST assumes no responsibility whatsoever for its use by other parties, and makes no guarantees, expressed or implied, about its quality, reliability, or any other characteristic.

We would appreciate acknowledgement if the software is incorporated in redistributable libraries or applications.
