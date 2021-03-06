![Loom logo](https://raw.github.com/aysylu/loom/master/doc/loom_logo.png "Loom")

[![Build Status](https://travis-ci.org/aysylu/loom.png)](http://travis-ci.org/aysylu/loom)

**Caveat coder**: This library is not yet battle-tested.


## Video and Slides

Watch the [talk on Loom](http://youtu.be/Iev7zavblqg) and view [slides](http://www.slideshare.net/aysylu/aysylu-loom).

## Usage

The namespace `loom.graph` must be AOT compiled. You can include `:aot [loom.graph]` in your project.clj to do this. This is due to the issue described [on the Clojure mailing list here](https://groups.google.com/forum/?hl=en#!topic/clojure/ZeENLkHAQTU).

### Leiningen/Clojars [group-id/name version]

    [aysylu/loom "0.4.2"]

### Namespaces

    loom.graph - records & constructors
    loom.alg   - algorithms (see also loom.alg-generic)
    loom.gen   - graph generators
    loom.attr  - graph attributes
    loom.label - graph labels
    loom.io    - read, write, and view graphs in external formats

### Documentation

[API Reference](http://aysy.lu/loom/)

[Frequently Asked Questions](http://aysy.lu/loom/faq.html)

### Basics

Create a graph:

    ;; Initialize with any of: edges, adacency lists, nodes, other graphs
    (def g (graph [1 2] [2 3] {3 [4] 5 [6 7]} 7 8 9))
    (def dg (digraph g))
    (def wg (weighted-graph {:a {:b 10 :c 20} :c {:d 30} :e {:b 5 :d 5}}))
    (def wdg (weighted-digraph [:a :b 10] [:a :c 20] [:c :d 30] [:d :b 10]))
    (def rwg (gen-rand (weighted-graph) 10 20 :max-weight 100))
    (def fg (fly-graph :successors range :weight (constantly 77)))

If you have [GraphViz](http://www.graphviz.org) installed, and its binaries are in the path, you can view graphs with <code>loom.io/view</code>:

    (view wdg) ;opens image in default image viewer
    
Inspect:

    (nodes g)
    => #{1 2 3 4 5 6 7 8 9}
    
    (edges wdg)
    => ([:a :c] [:a :b] [:c :d] [:d :b])
    
    (successors g 3)
    => #{2 4}
    
    (predecessors wdg :b)
    => #{:a :d}
    
    (out-degree g 3)
    => 2
    
    (in-degree wdg :b)
    => 2
    
    (weight wg :a :c)
    => 20
    
    (map (juxt graph? directed? weighted?) [g wdg])
    => ([true false false] [true true true])
    
Add/remove items (graphs are immutable, of course, so these return new graphs):

    (add-nodes g "foobar" {:name "baz"} [1 2 3])
    
    (add-edges g [10 11] ["foobar" {:name "baz"}])
    
    (add-edges wg [:e :f 40] [:f :g 50]) ;weighted edges
    
    (remove-nodes g 1 2 3)

    (remove-edges g [1 2] [2 3])
    
    (subgraph g [5 6 7])

Traverse a graph:

    (bf-traverse g) ;lazy
    => (9 8 5 6 7 1 2 3 4)
    
    (bf-traverse g 1)
    => (1 2 3 4)
    
    (pre-traverse wdg) ;lazy
    => (:a :b :c :d)
    
    (post-traverse wdg) ;not lazy
    => (:b :d :c :a)
    
    (topsort wdg)
    => (:a :c :d :b)

Pathfinding:

    (bf-path g 1 4)
    => (1 2 3 4)
    
    (bf-path-bi g 1 4) ;bidirectional, parallel
    => (1 2 3 4)
    
    (dijkstra-path wg :a :d)
    => (:a :b :e :d)
    
    (dijkstra-path-dist wg :a :d)
    => [(:a :b :e :d) 20]

Other stuff:

    (connected-components g)
    => [[1 2 3 4] [5 6 7] [8] [9]]

    (bf-span wg :a)
    => {:c [:d], :b [:e], :a [:b :c]}

    (pre-span wg :a)
    => {:a [:b], :b [:e], :e [:d], :d [:c]}
    
    (dijkstra-span wg :a)
    => {:a {:b 10, :c 20}, :b {:e 15}, :e {:d 20}}
    
Attributes on nodes and edges:

    (def attr-graph (-> g
                    (add-attr 1 :label "node 1")
                    (add-attr 4 :label "node 4")
                    (add-attr-to-nodes :parity "even" [2 4])
                    (add-attr-to-edges :label "edge from node 5" [[5 6] [5 7]])))
    
    ; Return attribute value on node 1 with key :label               
    (attr attr-graph 1 :label)
    => "node 1"
    
    ; Return attribute value on node 2 with key :parity
    (attr attr-graph 2 :parity)
    => "even"
    
    ; Getting an attribute that doesn't exist returns nil
    (attr attr-graph 3 :label)
    => nil
    
    ; Return all attributes for node 4
    ; Two attributes found
    (attrs attr-graph 4)
    => {:parity "even", :label "node 4"}
    
    ; Return attribute value for edge between nodes 5 and 6 with key :label
    (attr attr-graph 5 6 :label)
    => "edge from node 5"
    
    ; Return all attributes for edge between nodes 5 and 7
    (attrs attr-graph 5 7)
    => {:label "edge from node 5"}
    
    ; Getting an attribute that doesn't exist returns nil
    (attrs attr-graph 3 4)
    => nil
    
    ; Remove the attribute of node 4 with key :label
    (def attr-graph (remove-attr attr-graph 4 :label))
    
    ; Return all attributes for node 4
    : One attribute found because the other has been removed
    (attrs attr-graph 4)
    => {:parity "even"}

## Dependencies

Nothing but Clojure. There is optional support for visualization via [GrapViz](http://graphviz.org).

## TODO

* Use deftype instead of defrecord
* Do more functional graph research
* Solidify performance guarantees
* Implement more algorithms
* Test & profile more with big, varied graphs
* Multigraphs, hypergraphs, adjacency matrix-based graphs?

## Contributors

Names in no particular order:

* [Justin Kramer](https://github.com/jkk/)
* [Aysylu Greenberg] (https://github.com/aysylu), [aysylu [dot] greenberg [at] gmail [dot] com](mailto:aysylu.greenberg@gmail.com), [@aysylu22](http://twitter.com/aysylu22)
* [Robert Lachlan](https://github.com/heffalump), [robertlachlan@gmail.com](mailto:robertlachlan@gmail.com)
* [Stephen Kockentiedt](https://github.com/s-k)

## License

Copyright (C) 2010-2013 Aysylu Greenberg & Justin Kramer (jkkramer@gmail.com)

Distributed under the [Eclipse Public License](http://opensource.org/licenses/eclipse-1.0.php), the same as Clojure.
