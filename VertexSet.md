This module is an auxiliary module containing an implementation of sets
of vertices.

``` 
module VertexSet 
 ( Vertex, VertexSet, empty, insert, insertAll, inSet, remove, vertexListToSet, vertexSetToList ) 
 where
```

``` 
import FiniteMap ( FM, emptyFM, addToFM, elemFM, delFromFM, keysFM )
```

Vertices are simply integers for simplicity.

``` 
type Vertex = Int
```

Vertex sets are represented using finite maps with unit values. Unit
values do not require any combination of values in case a key is already
contained in the map.

``` 
type VertexSet = FM Vertex ()
```

The empty finite map.

``` 
empty :: FM a b
empty = emptyFM (<)
```

Inserts a vertex in a vertex set.

``` 
insert :: Vertex -> VertexSet -> VertexSet
insert i m = addToFM m i ()
```

Inserts a list of vertices into a vertex set.

``` 
insertAll :: [Vertex] -> VertexSet -> VertexSet
insertAll xs m = foldr insert m xs
```

Checks whether a given vertex is contained in the vertex set.

``` 
inSet :: Vertex -> VertexSet -> Bool
inSet = elemFM
```

Deletes a vertex from a vertex set.

``` 
remove :: Vertex -> VertexSet -> VertexSet
remove = flip delFromFM
```

Turns a list of vertices into a set of vertices. Multiple occurrences of
elements are removed automatically.

``` 
vertexListToSet :: [Vertex] -> VertexSet
vertexListToSet xs = insertAll xs empty
```

Turns a set of vertices into a list of vertices.

``` 
vertexSetToList :: VertexSet -> [Vertex]
vertexSetToList = keysFM
```
