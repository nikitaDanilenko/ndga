This is an auxiliary module that provides arithmetic operations on
finite maps.

``` 
module MapArithmetics ( swap, (.+.), (.-.), (*>), (!), del, showEdgeMap, fromList ) where
```

``` 
import FiniteMap ( FM, listToFM, keyOrder, fmToListPreOrder, plusFM_C, mapFM, 
                   lookupWithDefaultFM, keysFM, intersectFM, addListToFM, emptyFM )

import List      ( sum, intercalate )
```

This function associates the value at key *(v, w)* with the one at *(w,
v)* and vice versa.

``` 
swap :: FM (a, a) b -> FM (a, a) b
swap c = listToFM (keyOrder c) (map (\((v, w), a) -> ((w, v), a)) (fmToListPreOrder c))
```

Addition of integer valued maps. This operation is performed pointwise
and thus based upon the union of finite maps.

``` 
(.+.) :: FM a Int -> FM a Int -> FM a Int
(.+.) = plusFM_C (+)
```

Subtraction of integer valued maps. The second argument is negated and
then added to the first one.

``` 
(.-.) :: FM a Int -> FM a Int -> FM a Int
a .-. b = a .+. mapFM (const negate) b
```

Multiplies an integer map with a constant value.

``` 
(*>) :: Int -> FM a Int -> FM a Int
x *> c = mapFM (const (x *)) c
```

Queries an integer-valued finite map at a given key. If the key does not
have an association in the map, zero is returned.

``` 
(!) :: FM a Int -> a -> Int
m ! k = lookupWithDefaultFM m 0 k
```

Given an map on pairs of keys and a key this function computes the
difference of the values that exit this vertex and those that enter it.

``` 
del :: FM (a, a) Int -> a -> Int
del em x = sum [em ! k | k <- keys, fst k == x] - sum [em ! k | k <- keys, snd k == x]
  where keys = keysFM em
```

Pretty-prints the map association values.

``` 
showEdgeMap :: FM a b -> String
showEdgeMap = intercalate ", " . map (\(p, e) -> show p ++ "->" ++ show e) . fmToListPreOrder
```

Creates a map from a list of associations.

``` 
fromList :: [(a, b)] -> FM a b
fromList = addListToFM (emptyFM (<))
```
