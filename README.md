ndga
====

A prototypical Curry library that employs non-determinism in certain graph algorithms.

Installation
------------

To use this library, the [KiCS2 compiler](http://www-ps.informatik.uni-kiel.de/kics2/) is required.
Assuming that this repository has been cloned
to `~/ndga/`, a simple interactive way to use it is the following.

~~~{.sh}
bash> cd ndga
bash> kics2
kics2> :set v0
kics2> :set +interactive
kics2> :l Basics.lcurry
~~~

The module `Basics` exports all exported functions from the modules `Graph`, `MapArithmetics`,
`VertexSet`, `SearchTree` and `SetFunctions` and thus allows to use all of them in the interactive
mode of KiCS2.
Additional modules can be loaded like

~~~{.sh}
kics2> :add Matching.lcurry
~~~

Documentation
-------------

You can browse the files online as literate Curry files:

* [Graph](./Graph.lcurry)
  contains graphs and graph operations, including path search.
* [Matching](./Matching.lcurry)
  contains all functions necessary for the computation of maximum matchings.
* [Flow](./Flow.lcurry)
  contains all functions necessary for the computation of maximal flows.
* [VertexSet](./VertexSet.lcurry)
  provides an auxiliary data structure for vertex sets and some operations on these sets.
* [MapArithmetics](./MapArithmetics.lcurry)
  another auxiliary module that provides arithmetic operations on finite maps.
* [Basics](./Basics.lcurry)
  a convenience module that exports a list of other modules only and thus simplifies testing.

Alternatively, you can view them in a slightly more formatted way as markdown files:

* [Graph](./Graph.md)
* [Matching](./Matching.md)
* [Flow](./Flow.md)
* [VertexSet](./VertexSet.md)
* [MapArithmetics](./MapArithmetics.md)
* [Basics](./Basics.md)