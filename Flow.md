This module is used for the computation of maximal flows in a network.

``` 
module Flow 
  ( Network, mkNetwork, graph, source, sink, capacity, Path ( .. ), augmentBy, augmenting,
    maximalFlowWith, maximalFlowDFS, maximalFlowBFS, network1, network2, network3 ) 
  where

import FiniteMap      ( FM, intersectFM )
import Integer        ( minlist )
import SearchTree     ( Strategy, bfsStrategy, dfsStrategy )
import SetFunctions   ( set1With, isEmpty, chooseValue )
```

``` 
import Graph          ( Vertex, Edge, Graph, isEmptyGraph, transpose, graphIntersection, mkGraph )
import MapArithmetics ( swap, (.+.), (.-.), (*>), (!), del, showEdgeMap, fromList ) 
import VertexSet      ( empty, insert, inSet )
```

Networks and paths
==================

An edge map is simply a finite map with edges for keys.

``` 
type EdgeMap a = FM Edge a
```

For simplicity we represent a network as a structure consisting of an
asymmetric graph, the source, the sink and the capacity edge map.

``` 
data Network a = Network Graph Vertex Vertex (EdgeMap a)
```

This is a smart constructor that checks the graph for being asymmetric
and if it is not, it fails and no network is created.

``` 
mkNetwork :: Graph -> Vertex -> Vertex -> EdgeMap a -> Network a
mkNetwork g s t c | isEmptyGraph (g `graphIntersection` transpose g) = Network g s t c
```

Returns the underlying graph of a network.

``` 
graph :: Network a -> Graph
graph (Network g _ _ _) = g
```

Returns the source of a network.

``` 
source :: Network a -> Vertex
source (Network _ so _ _) = so
```

Returns the sink of a network.

``` 
sink :: Network a -> Vertex
sink (Network _ _ si _) = si
```

Returns the capacity edge map of a network.

``` 
capacity :: Network a -> EdgeMap a
capacity (Network _ _ _ c) = c
```

A path with values along its edges is either the final vertex or a path
from a vertex following an edge with a given weight to a given path.

``` 
data Path a = Final Vertex | From Vertex a (Path a)
```

Flow improvement and maximal flows
==================================

This function non-deterministically computes an augmenting path with
respect to a given residual capacity.

``` 
augmenting :: Vertex -> Vertex -> EdgeMap Int -> Path Int
augmenting s t cap = go empty s where

    go vis from | from == t                         = Final from
                | cfi > 0 && not (from `inSet` vis) = From from cfi (go (insert from vis) i)
               where i free
                     cfi = cap ! (from, i)
```

Collects the valued edges along a valued path.

``` 
toEdges :: Path a -> [(Edge, a)]
toEdges (Final _)         = []
toEdges (From x val path) = go x val path where
```

``` 
   go prevV prevA (Final v)    = [((prevV, v), prevA)]
   go prevV prevA (From v a p) = ((prevV, v), prevA) : go v a p
```

This function takes an augmenting path, the original capacity, the
residual capacity, the current flow and updates the residual capacity
and the current flow according to the Ford-Fulkerson theorem.

``` 
augmentBy :: Path Int -> EdgeMap Int -> EdgeMap Int -> EdgeMap Int -> (EdgeMap Int, EdgeMap Int)
augmentBy path original residual flow = ((residual .-. upd) .+. swap upd, flow .+. upd) where

   weightedEdges = toEdges path
   updateMap     = fromList (map (\(e, _) -> (e, 1)) weightedEdges)

   eps   = minlist (map snd weightedEdges)
   upd   = eps *> intersectFM original (updateMap .-. swap updateMap)
```

The maximum flow function parametrised over a search strategy. It
repeatedly searches for an augmenting path and returns the current flow,
if no such path exists. This function introduces additional
non-determinism in the choice of the augmenting path.

``` 
maximalFlowWith :: Strategy (Path Int) -> Network Int -> EdgeMap Int
maximalFlowWith str (Network _ s t original) = go (original, empty) where

  go (cf, flow) 
     | isEmpty ps = flow
     | otherwise  = go (augmentBy path original cf flow)
     where ps = set1With str findAugmenting cf
           path = chooseValue ps

  findAugmenting = augmenting s t
```

This function searches for maximal flows using the depth-first strategy.

``` 
maximalFlowDFS :: Network Int -> EdgeMap Int
maximalFlowDFS = maximalFlowWith dfsStrategy
```

This function searches for maximal flows using the breadth-first
strategy.

``` 
maximalFlowBFS :: Network Int -> EdgeMap Int
maximalFlowBFS = maximalFlowWith bfsStrategy
```

Example networks
================

This is the example network from the paper.

``` 
network1 :: Network Int
network1 = mkNetwork g 0 7 cap where
  g   = mkGraph [[1, 2, 3], [4, 5], [5], [5, 6], [2, 7], [4, 6], [7], []]
  cap = fromList . map regroup $
           [(0, 1, 7), (0, 2, 8), (0, 3, 9),
            (1, 4, 3), (1, 5, 4),
            (2, 5, 6),
            (3, 5, 2), (3, 6, 5),
            (4, 2, 8), (4, 7, 10),
            (5, 4, 4), (5, 6, 7),
            (6, 7, 10)]
```

This is the example network from the [Wikipedia
page](http://en.wikipedia.org/wiki/Maximal_flow).

``` 
network2 :: Network Int
network2 = mkNetwork g 0 5 cap where
  g   = mkGraph [[1, 3], [2, 3], [4, 5], [4], [5], []]
  cap = fromList [((0,1), 3), ((0,3), 3), 
                 ((1,2), 3), ((1,3), 2), 
                 ((2,4), 4), ((2,5), 2), 
                 ((3,4), 2), 
                 ((4,5), 3)]
```

This network is a simple example of how the depth-first strategy can be
not polynomially complex, depending on a poor choice of a vertex
ordering.

``` 
network3 :: Network Int
network3 = mkNetwork g 0 3 cap where
   g   = mkGraph [[1, 2], [2,3], [3], []]
   cap = fromList . map regroup $ [(0, 1, h), (0, 2, h), (1, 2, l), (1, 3, h), (2, 3, h)]
   h   = 1048576
   l   = 1
```

Auxiliary function that allows to avoid brackets above.

``` 
regroup :: (a, b, c) -> ((a, b), c)
regroup (x, y, z) = ((x, y), z)
```
