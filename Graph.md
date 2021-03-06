This module provides a simple representation of graphs in an adjacency
list model and several operations on graphs. Additionally, it contains a
simple path search algorithm and some small example graphs.

``` 
module Graph 
  ( Edge, Vertex, Path, Graph, showGraph, graphEdges, emptyGraph, mkGraph, mkGraphFromAdjacency,
    graphToAdjacencyList, successors, vertices, noSuccessors, graphSize, addEdge, addBiEdge, 
    removeEdge, removeBiEdge, xorEdge, xorBiEdge, xorEdges, xorBiEdges, isEdge, isPathIn, 
    isEmptyGraph, transpose, graphUnion, graphIntersection, symmetrise, mkSymmetricGraph,
    edgesOnPath, reachable, path, pathToSingle, inSameComponent, twoPathsCycle, predecessors,
    graph1, graph2, graph3, graph4, graph5, graph6, graph7 )
  where
```

``` 
import FiniteMap ( FM, eltsFM, mapFM, addListToFM, lookupWithDefaultFM, keysFM, filterFM, fmToList,
                   addToFM_C, plusFM_C, updFM, foldFM, intersectFM_C )
import List      ( sortBy, nub )
```

``` 
import VertexSet ( Vertex, VertexSet, vertexListToSet, empty, inSet, insert, singleton )
```

Edges are pairs of vertices.

``` 
type Edge = (Vertex, Vertex)
```

A path is a list of vertices, where the in the list is the order of
traversal.

``` 
type Path = [Vertex]
```

Graphs and construction functions.
==================================

A graph is a finite map with vertex keys and lists of vertices as
values. The latter are increasingly sorted with respect to the natural
order on vertices. Each list of vertices represents the adjacency list
of the key the value belongs to.

``` 
data Graph = Graph (FM Vertex [Vertex])
```

Selector function that returns the underlying finite map of a graph.

``` 
graphFM :: Graph -> FM Vertex [Vertex]
graphFM (Graph fm) = fm
```

This function pretty prints a graph in a table-like form, where each
line contains a value /v/ that is the given vertex followed by its
adjacency list.

``` 
showGraph :: Graph -> String
showGraph = unlines . zipWith (\i line -> unwords [show i, ":", show line]) [0..] . eltsFM . graphFM
```

This function returns the list of edges of a graph.

``` 
graphEdges :: Graph -> [Edge]
graphEdges = concat . eltsFM . mapFM (zip . repeat) . graphFM
```

This function creates an empty graph with a specified number of
vertices.

``` 
emptyGraph :: Int -> Graph
emptyGraph n = mkGraph (replicate n [])
```

Checks whether a given graph is empty.

``` 
isEmptyGraph :: Graph -> Bool
isEmptyGraph = foldFM (const((&&) . null)) True . graphFM
```

A smart constructor for graphs which sorts the adjacency lists.

``` 
mkGraph :: [[Vertex]] -> Graph
mkGraph = mkGraphFromAdjacency . zip [0..] . map (nub . sortBy (<=))
```

Takes a list of vertex-adjacency-list pairs and turns it into a graph.
As mentioned above, adjacency lists are assumed to be increasingly
sorted.

``` 
mkGraphFromAdjacency :: [(Vertex, [Vertex])] -> Graph
mkGraphFromAdjacency = Graph . addListToFM empty
```

Turns a graph into a list of vertex-adjacency-list pairs.

``` 
graphToAdjacencyList :: Graph -> [(Vertex, [Vertex])]
graphToAdjacencyList = fmToList . graphFM
```

Single step graph operations.
=============================

Computes the successors of a vertex in a graph. When a vertex is not
contained in a graph, this function returns the empty list.

``` 
successors :: Graph -> Vertex -> [Vertex]
successors (Graph g) = lookupWithDefaultFM g []
```

Computes the predecessors of a vertex in a graph. This function is
declarative, but rather inefficient, since computes (parts of) the
transposed graph.

``` 
predecessors :: Graph -> Vertex -> [Vertex]
predecessors = successors . transpose
```

Returns the vertices of a graph.

``` 
vertices :: Graph -> [Vertex]
vertices = keysFM . graphFM
```

Returns those vertices of a graph that do not have successors. For
symmetric graphs this is the same as computing isolated vertices.

``` 
noSuccessors :: Graph -> VertexSet
noSuccessors = vertexListToSet . keysFM . filterFM (const null) . graphFM
```

This function returns the size of a graph.

``` 
graphSize :: Graph -> Int
graphSize = length . vertices
```

Inserts an edge into a graph.

``` 
addEdge :: Vertex -> Vertex -> Graph -> Graph
addEdge s t (Graph fm) = Graph (addToFM_C union fm s [t])
```

Inserts an undirected edge into a graph, i.e. given vertices `v, w` it
inserts the edges `(v, w)` and `(w, v)`.

``` 
addBiEdge :: Vertex -> Vertex -> Graph -> Graph
addBiEdge s t g = addEdge s t (addEdge t s g)
```

Removes an edge from a graph.

``` 
removeEdge :: Vertex -> Vertex -> Graph -> Graph
removeEdge s t (Graph fm) = Graph (updFM fm s (delete t))
```

Removes an undirected edge from a graph by removing both directions.

``` 
removeBiEdge :: Vertex -> Vertex -> Graph -> Graph
removeBiEdge s t g = removeEdge s t (removeEdge t s g)
```

Given a curried edge this function removes this edge from the graph if
it is already contained in the graph and inserts it into the graph
otherwise.

``` 
xorEdge :: Vertex -> Vertex -> Graph -> Graph
xorEdge x y (Graph gm) = Graph (updFM gm x (symDiff [y]))
```

Performs the `xorEdge` operation with a list of edges.

``` 
xorEdges :: Graph -> [Edge] -> Graph
xorEdges = foldr (uncurry xorEdge)
```

Similar to `xorEdge`, but in both directions.

``` 
xorBiEdge :: Vertex -> Vertex -> Graph -> Graph
xorBiEdge x y g = xorEdge y x (xorEdge x y g)
```

Performs the `xorBiEdge` operation with a list of edges.

``` 
xorBiEdges :: Graph -> [Edge] -> Graph
xorBiEdges = foldr (uncurry xorBiEdge)
```

Checks whether there is an edge between two vertices in the graph in the
given order.

``` 
isEdge :: Graph -> Vertex -> Vertex -> Bool
isEdge graph s t = t `elem` successors graph s
```

Checks whether a path is located in a graph.

``` 
isPathIn :: Path -> Graph -> Bool
isPathIn ps gr = all (uncurry (isEdge gr)) (edgesOnPath ps)
```

Additional graph operations
===========================

This function transposes a graph, i.e. flips all the edges.

``` 
transpose :: Graph -> Graph
transpose g = mkGraphFromAdjacency [(v, [w | w <- vs, isEdge g w v]) | v <- vs]
  where vs = vertices g
```

Computes the union of two graphs by combining vertices and edges. A
vertex/edge is present in the result, iff it is present in one of the
arguments.

``` 
graphUnion :: Graph -> Graph -> Graph
graphUnion (Graph g) (Graph h) = Graph (plusFM_C union g h)
```

Computes the intersection of two graphs by intersecting the vertices and
edges. A vertex/edge is present in the result, iff it is present in both
arguments.

``` 
graphIntersection :: Graph -> Graph -> Graph
graphIntersection (Graph g) (Graph h) = Graph (intersectFM_C intersection g h)
```

This function computes the symmetric closure of a graph by uniting it
with its transposed self.

``` 
symmetrise :: Graph -> Graph
symmetrise g = g `graphUnion` transpose g
```

Create a symmetric graph from a list of vertex lists.

``` 
mkSymmetricGraph :: [[Vertex]] -> Graph
mkSymmetricGraph = symmetrise . mkGraph
```

Computes the edges along a path.

``` 
edgesOnPath :: Path -> [Edge]
edgesOnPath p = zip p (tail p)
```

This function checks whether there is a path from a vertex to a vertex
set in the graph. If there is such a path, it returns `Success`
otherwise it fails and returns no value at all.

``` 
reachable :: Graph -> Vertex -> VertexSet -> Success
reachable g from ts = find empty from where

    find vis s | s `inSet` ts                        = success
               | isEdge g s i && not (i `inSet` vis) = find (insert s vis) i
               where i free
```

This function non-deterministically computes a path from a vertex into a
vertex set. If no such path exists, the function fails and returns no
values, otherwise it returns a path.

``` 
path :: Graph -> Vertex -> VertexSet -> Path
path g from ts = find empty from where

  find vis s | s `inSet` ts                         = [s]
             | isEdge g s i && not (i `inSet` vis)  = s : find (insert s vis) i
             where i free
```

This function calls `path` with a singleton target set.

``` 
pathToSingle :: Graph -> Vertex -> Vertex -> Path
pathToSingle g from t = path g from (vertexListToSet [t])
```

This function checks whether two vertices are contained in the same
strongly connected component.

``` 
inSameComponent :: Graph -> Vertex -> Vertex -> Success
inSameComponent g s t | reachable g s (singleton t) = reachable g t (singleton s)
```

With `onCycle` one can obtain a cycle that contains the given vertices,
in case such a cycle exists. However, this function finds only those
cycles that consist exactly of two paths, namely a path from the first
vertex to the second followed by a path from the second vertex to the
first. This is important when the start vertex is the same as the target
vertex, since in this case exactly the trivial cycle is found and not
all cycles containing the vertex.

``` 
twoPathsCycle :: Graph -> Vertex -> Vertex -> [Vertex]
twoPathsCycle g s t = pathToSingle g s t ++ tail (pathToSingle g t s)
```

Example graphs
==============

This is the graph *G1* from the paper.

``` 
graph1 :: Graph
graph1 = mkSymmetricGraph [[1,3], [2,4], [], [4], [5], []]
```

This is the graph *G2* from the paper.

``` 
graph2 :: Graph
graph2 = mkSymmetricGraph [[1,3], [2], [5], [6], [5,7], [8], [7], [8], []]
```

A small unsymmetric graph containing a cycle and a transitive step ((0,
2), (2, 1) and (0, 1)).

``` 
graph3 :: Graph
graph3 = mkGraphFromAdjacency [(0, [1,2]), (1, [3]), (2, [1]), (3, [2])]
```

``` 
graph3s :: Graph
graph3s = symmetrise graph3
```

``` 
graph4 :: Graph
graph4 = mkSymmetricGraph [[1,2], [0,3], [0,3,4], [1,2,5], [2], [3,4]]
```

``` 
graph5 :: Graph
graph5 = mkSymmetricGraph [[1,2], [3], [3,4], [5], [], []]
```

A cycle with three vertices.

``` 
graph6 :: Graph
graph6 = mkSymmetricGraph [[1], [2], [0]]
```

This graph is "severely" not bipartite and will break typical
implementations on specific vertex orderings.

``` 
graph7 :: Graph
graph7 = mkSymmetricGraph [[1], [2, 6], [3, 6], [4, 5, 7], [5, 6], [6], [], []]
```

Auxiliaries
-----------

Auxiliary functions for union, difference, intersection and symmetric
difference. All are based upon a merging technique which requires all
arguments to be increasingly sorted with respect to their natural order.
These functions are not exported.

``` 
union :: [a] -> [a] -> [a]
union []         ys@(_:_)             = ys
union xs         []                   = xs
union l@(x : xs) m@(y : ys) | x < y     = x : union xs m
                            | x == y    = x : union xs ys
                            | otherwise = y : union l ys
```

``` 
difference :: [a] -> [a] -> [a]
difference []         (_ : _)                = []
difference xs         []                     = xs
difference l@(x : xs) m@(y : ys) | x < y     = x : difference xs m
                                 | x == y    =     difference xs ys
                                 | otherwise =     difference l ys
```

``` 
intersection :: [a] -> [a] -> [a]
intersection []         (_ : _)                = []
intersection _          []                     = []
intersection l@(x : xs) m@(y : ys) | x < y     =     intersection xs m
                                   | x == y    = x : intersection xs ys
                                   | otherwise =     intersection l ys
```

``` 
delete :: a -> [a] -> [a]
delete x ys = difference ys [x]
```

``` 
symDiff :: [a] -> [a] -> [a]
symDiff []         ys@(_:_)               = ys
symDiff xs         []                     = xs
symDiff l@(x : xs) m@(y : ys) | x < y     = x : symDiff xs m
                              | x == y    = symDiff xs ys
                              | otherwise = y : symDiff l ys
```
