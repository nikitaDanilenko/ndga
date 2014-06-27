ndga
====

A prototypical Curry library that employs non-determinism in certain graph algorithms.

To use this library, the [KiCS2 compiler](http://www-ps.informatik.uni-kiel.de/kics2/) is required.
Assuming that this repository has been cloned
to `~/ndga/`{.bash}, a simple interactive way to use it is the following.

> bash> cd ndga
> bash> kics2
> kics2> :set v0
> kics2> :set +interactive
> kics2> :l Basics.lcurry

The module `Basics` exports all exported functions from the modules `Graph`, `MapArithmetics`,
`VertexSet`, `SearchTree` and `SetFunctions` and thus allows to use all of them in the interactive
mode of KiCS2.
Additional modules can be loaded like

> kics2> :add Matching.lcurry