# JAAL JSON Schema documentation

This file documents each part of the JAAL JSON Schema in detail.
The schema files are at `spec/shemas/*.json`. The schema files itself have
comments in the "description" fields, but as they are oneliners, further
information is stored in this document:

- The what: the purpose of each schema.
- The why: design choices behind each schema.

## Common properties for JAAL schemas

### "id" field

Most schemas have an "id" field (a string). It has two purposes:

1. The beginning of the id indicates the type of a subschema
   (edge, graph, keyvalue, matrix, node, style). As the JAAL document is
   one large object, this is the way to easily recognise the data structures
   when reading it.

2. Each data structure must have a unique id, because data structures can
   be referred by their id in JAAL. Cases:

   - Graph has nodes, and multiple edges may point to the same node.
   - The animation or model answer contains an operation which modifies an
    existing data structure.

## animation.json

Student's solution steps for the exercise.

Properties:

## definitions.json

Other initial data than the initial data structures.

Properties:

- styles: Array of style definitions. Here are the actual styles defined, and
  it is preferred that the recording otherwise refers to the IDs of these
  styles.

- score: Grading data of the exercise.
  - modelSteps: Number of steps in the model answer.
  - studentSteps: This is the same than length of the array jaal.animation.
  - correctSteps: Number of student's consecutive correct steps from the
    beginning, counted by automatic grader of the exercise.
  - undoCount: Number of Undo actions.

  The computational model regarding these steps is the following:
  - modelSteps are decided by the exercise.
  - correctSteps are decided by the exercise. They are counted after the
    student has clicked the Grade button.
  - studentSteps increase every time the student does something, and it is
    handled by the JAAL recorder.
  - UndoCount increases every time the student clicks the Undo button.
    It is handled by the JAAL recorder.

- modelAnswer: the model solution of the exercise. This is stored as either an array of events, or as an array of array of events. In the 2-D array of events case, the intended behaviour is that one step is a major step that matches what the student should be doing, with optional extra minor narration steps. 

## edge.json

A graph edge in a JAAL recording.

Properties:

- node: A graph edge is defined by two nodes. If the graph is directed, the'
  first node is the source and the second node is the target.

  Currently JAAL does not support hyperedges; each edge has exactly two node
  references. However, "node" is an array for forward compatibility, if
  hyperedges are to be added in the future.

- tag: Text displayed in the middle of the edge. E.g. name or weight of the edge.
  In finite state machines, the tag should be used for indicating the condition
  related to a state transfer (including epsilon automata and Turing machines).

- tailElem: Data element at the tail (start) node of the edge. This is a way
  to indicate the names of pointer variables when describing the logical
  structure of a linked list or a tree.

- headElem: This exist for symmetry to tailElem. Furthermore, it can be used
  for annotations where an edge depicts a relation that has designated roles
  for both ends of the relation.

## event.json

An event in a JAAL recording: either user's action or an step in model answer.

Properties:
- type: The type of the event.
  - `click` is a click of a data structure or an user interface widget in the
    simulation. The click events have a timestamp: integer value depicting
    *milliseconds* from the time-date when the exercise is loaded. This is most
    convenient for data analysis.
  - `undo` is the click of the Undo button. This is a separate event type
    to easily support finding difficult parts of the exercise where the user
    has clicked the Undo button.
  - `grade` is the click of the Grade button.
  - `operation` is a data structure operation. This is used at least in the
     model answer.
  - `narration` means that the narrative text in the model answer changes.
- time: the time in milliseconds since the start of the simulation. Must be an integer.
- object: the name of a JAAL data structure that is referenced with the current event. Must start with `edge`, `graph`, `keyvalue`, `matrix`, or `node`. 
- gradable: a boolean to indicate whether the current event responds to a gradable step. 
- image: a string representation of an SVG image. 

## graph.json

A graph data structure for representing lists, trees, and graphs.

The graph is the most generic structure. Trees are graphs without loops.
Lists are linear trees: each node has at most one children.

Multigraphs are possible in JAAL 1.1, but not intended behaviour. Hypergraphs are explicitly forbidden as an edge must have exactly two nodes it is connected to. 

Some applications:
- lists (linked list)
- trees
  - rooted
    - search tree
      - binary (AVL, red-black)
      - generic
      - kd-tree      
    - trie
    - parse tree
    - single-source shortest paths tree
  - rootless
    - spanning tree
- graphs
  - basic graph algorithm demonstration
  - finite state machines
    (deterministic finite automata, Turing machine)

## initialState.json

Initial state for the exercise.

Properties:
- dataStructures: an array of data structures. Only Graph, Matrix and Node data
  structures are allowed here. A single node is allowed in the initial state
  to represent an already initialized variable which has important role in
  the simulation, such as a temporary value for swaps in the Insertion sort
  algorithm.


## jaal.json

JSON-based algorithm animation language. This is the the top-level schema which
references to other schemas. It defines the main structure of the language.

## keyvalue.json

A key which points to a value.

Many search structures have key-value pairs: consider trees and hash tables.
Therefore we consider that a key-value is a typical, logical unit of data and
we want to be able to highlight it easily.

Consider B-trees which have multiple key-value pairs. It is meant that a
B-tree node has an array of key-value pairs, not two arrays separately for keys
and values; the logical connection inside each key-value pair is stronger than
between keys.1

## matrix.json

A one- or two-dimensional array with fixed number of rows and columns.

Applications: sorting algorithms, auxiliary data structures, parsing tables
in compilers, and linear algebra.

This schema is called "matrix" for clarity to discern it from the array type
of JSON Schema.

## metadata.json

Metadata about a JAAL recording.

This a typical header that most file formats have.

- jaalVersion: this allows a JAAL-reading software easily recognise a JAAL
  file. The version number allows future development of the language.

- jaalGenerator: software which created the file. This should contain name
  and version number for debugging purposes.

- browser: Name and version of the web browser which is running JAAL data
  generator software. This is for debugging as well.

- exercise.name: Name of the visual algorithm simulation (VAS) exercise,
  such as "Insertion Sort" or "Red-black Tree Insert".

- exercise.collection: This further identifies the exercise. Currently the
  only exercise collection seems to be the OpenDSA.

- exercise.runningLocation: URL of the configured and running exercise in
  a learning management system. It is optional, but may be used to identify
  the exact location of the software + exercise combination that produced the
  JAAL data.

## node.json

Generic data structure for an array cell, a list node, a tree node, and a
graph node.

- legend: A text to be displayed aside the node. Examples:

  - OpenDSA tree traversal exercises mark a sequential number
    along each tree node that the user has clicked. The numbers indicate the
    traversal order.

  - Other annotations which are not the input data but illustrate how the
    algorithm proceeds.

- key: Primary data inside the node. If the type is string, it may also be an
       identifier of a graph, matrix, or node defined elsewhere.

  In case of an array, each key of a node is simply a string. Furthermore,
  each cell of an array (matrix) is defined as a unique node, because we want
  to be able to identify clicks of certain nodes, or highlight some of them
  with a certain visual style.

  The key can be a key-value pair, if we want to represent a binary search tree
  with both the [search] keys and the values [data under each key].

  Furthermore, some search trees, such as B-tree, have multiple key-value
  pairs inside one node. See also keyvalue.json.

- style: Graphical style of the node: a name of a style definition.

  Applications:
  - Red-black tree
  - Quicksort: current pivot, pivots in the stack, current range to be
    processed, etc.
  - Finite automata: whether a state accepts or rejects the input
  - Graph algorithms: visitedness of a node

  All initial styles must be defined in the definitions section of JAAL,
  because it is likely that many data structures have similar style.

## style.json

A graphical style definition in a JAAL recording.

JAAL is designed with high compatibility with web browsers. Therefore it is
meant that graphical style definitions in JAAL are similar to Cascading Style
Sheets (CSS) for web pages.
