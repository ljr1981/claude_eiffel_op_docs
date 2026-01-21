# simple_graph Research Notes

## Step 1: Specifications

### Formal Graph Definition
- **Graph G(V, E)**: Finite set of vertices V with set of edges E
- **Directed graph (digraph)**: Edges are ordered pairs (u, v) representing u → v
- **Undirected graph**: Edges are unordered pairs {u, v}
- **Weighted graph**: Edges have associated numeric weights

### Representation Standards
- **Adjacency Matrix**: 2D boolean/numeric matrix adjMat[n][n]
  - Space: O(V²)
  - Edge lookup: O(1)
  - Iterate neighbors: O(V)
  - Best for dense graphs

- **Adjacency List**: Array of lists, each vertex has list of neighbors
  - Space: O(V + E)
  - Edge lookup: O(degree) or O(log degree) with sorted lists
  - Iterate neighbors: O(degree)
  - Best for sparse graphs (most real-world graphs)

### Core Algorithms (IEEE/ACM standard)
- **Traversal**: BFS, DFS
- **Shortest Path**: Dijkstra, Bellman-Ford, Floyd-Warshall, A*
- **Minimum Spanning Tree**: Prim, Kruskal
- **Connectivity**: Strongly/Weakly connected components, Tarjan's algorithm
- **Topological Sort**: For DAGs

Sources:
- [Graph (abstract data type) - Wikipedia](https://en.wikipedia.org/wiki/Graph_(abstract_data_type))
- [Adjacency list - Wikipedia](https://en.wikipedia.org/wiki/Adjacency_list)
- [Graph Algorithms - GeeksforGeeks](https://www.geeksforgeeks.org/dsa/graph-data-structure-and-algorithms/)

---

## Step 2: Tech-Stack Library Analysis

### Python - NetworkX
**Strengths:**
- Pure Python, easy to use
- Comprehensive algorithm collection
- Flexible - any hashable object as node, arbitrary edge data
- Great documentation and community support (4k+ GitHub stars)

**Weaknesses:**
- 10x slower than other libraries
- Single-source shortest path: 67s vs 6.8s (NetworKit)
- PageRank: 10+ minutes vs 1 minute (igraph)
- Not production-ready for large graphs
- In-memory only, no persistence

### Rust - Petgraph
**Strengths:**
- Multiple graph types: Graph, StableGraph, GraphMap, MatrixGraph, CSR
- Each optimized for different trade-offs
- Fast and memory-efficient
- Comprehensive algorithms included
- Extensible trait system

**API Patterns:**
```rust
let mut graph = Graph::<&str, f32>::new();
let a = graph.add_node("A");
let b = graph.add_node("B");
graph.add_edge(a, b, 1.0);
```

### Python - Rustworkx (Qiskit)
- Rust backend wrapping petgraph
- 3x to 100x faster than NetworkX
- Similar API to NetworkX but not drop-in replacement

### Key API Patterns Across Libraries
1. **Node handles/indices** instead of direct object references
2. **Generic node/edge data** - any type can be stored
3. **Fluent/builder patterns** for graph construction
4. **Iterator-based** neighbor traversal
5. **Separate graph types** for directed/undirected

Sources:
- [petgraph - GitHub](https://github.com/petgraph/petgraph)
- [rustworkx - GitHub](https://github.com/Qiskit/rustworkx)
- [Benchmark of graph packages](https://www.timlrx.com/blog/benchmark-of-popular-graph-network-packages)

---

## Step 3: Eiffel Ecosystem

### Gobo Structure Library
- Portable data structures library
- Lists, stacks, tables with multiple implementations
- MIT License
- No dedicated graph implementation found

### ETH Experimental Graph Library (Olivier Jeger)
- Adjacency list implementation
- Both directed and non-directed graphs
- Mathematical models using EiffelSpec
- Linear time neighbor iteration O(k)
- Edge lookup O(log k) with sorted lists
- **Not integrated into EiffelBase** (recommended but not done)

### EiffelBase
- Fundamental structures and algorithms
- "Linnaean" taxonomy of computing structures
- No native graph support found

### Gap Analysis
- **No production-ready graph library** in Eiffel ecosystem
- ETH project was experimental, not maintained
- Opportunity for simple_graph to fill this gap

Sources:
- [Gobo Eiffel Structure Library](http://www.gobosoft.com/eiffel/gobo/structure/)
- [Experimental Graph Library](https://www.eiffel.org/blog/maverick/experimental_graph_library)
- [EiffelBase](https://www.eiffel.org/doc/solutions/EiffelBase)

---

## Step 4: Developer Pain Points

### Performance
- NetworkX 10x slower than alternatives
- Pure Python implementation bottleneck
- Large graphs (billions of edges) unusable

### Data Management
- Data import from multiple sources is messy
- In-memory only - must reload for every change
- No persistence solution built-in

### API Issues
- Documentation extensive but confusing
- API changes between versions break code
- Node identity confusion (objects vs IDs)
- Layout issues with object nodes

### Visualization
- Not a drawing package
- Large graph visualization fails
- Need separate library for visualization

### Production Readiness
- Good for exploration, not production
- No ad-hoc querying
- Missing database features

### What Developers Want
1. **Simple API** for common cases
2. **Performance** that scales
3. **Stable node/edge identifiers**
4. **Clear documentation** with examples
5. **Built-in algorithms** that work out of the box
6. **Type safety** for node/edge data

Sources:
- [Biggest challenges with NetworkX](https://networkx.guide/biggest-challenges/)
- [Hacker News discussion](https://news.ycombinator.com/item?id=33717100)

---

## Step 5: Innovation Opportunities

### simple_graph Differentiators

1. **Contract-Based Safety**
   - Preconditions prevent invalid operations (negative weights for Dijkstra)
   - Postconditions guarantee algorithm correctness
   - Invariants maintain graph consistency

2. **Integer Node IDs**
   - Avoid NetworkX's object identity confusion
   - Stable references that survive graph mutations
   - O(1) lookup via array indexing

3. **Generic Node Data**
   - Store any Eiffel object as node payload
   - Type-safe access with generics `SIMPLE_GRAPH [G]`

4. **Fluent API for Construction**
   ```eiffel
   graph.add_node ("A").add_node ("B").add_edge (a, b, 1.0)
   ```

5. **Algorithm Results as Objects**
   - Path results include cost, path list, visited nodes
   - Easy to inspect and debug

6. **SCOOP-Ready Design**
   - Thread-safe for concurrent access
   - Separate processors for parallel algorithms (future)

7. **Progressive Disclosure**
   - Simple: `graph.add_node`, `graph.add_edge`, `graph.dijkstra`
   - Advanced: Custom edge comparators, algorithm callbacks

---

## Step 6: Design Strategy

### Core Design Principles
- **Simple**: One class for most use cases
- **Safe**: Contracts catch errors at runtime
- **Generic**: Works with any node data type
- **Efficient**: Adjacency list for sparse graphs (most common)

### API Surface

#### Main Class: SIMPLE_GRAPH [G]
```eiffel
class SIMPLE_GRAPH [G]

create
    make,           -- Undirected graph
    make_directed   -- Directed graph

feature -- Access
    node_count: INTEGER
    edge_count: INTEGER
    is_directed: BOOLEAN
    is_empty: BOOLEAN
    node_data (id: INTEGER): G
    has_node (id: INTEGER): BOOLEAN
    has_edge (from_id, to_id: INTEGER): BOOLEAN
    neighbors (id: INTEGER): LIST [INTEGER]
    edge_weight (from_id, to_id: INTEGER): REAL_64

feature -- Modification
    add_node (data: G): INTEGER  -- Returns node ID
    remove_node (id: INTEGER)
    add_edge (from_id, to_id: INTEGER)
    add_weighted_edge (from_id, to_id: INTEGER; weight: REAL_64)
    remove_edge (from_id, to_id: INTEGER)
    set_node_data (id: INTEGER; data: G)

feature -- Algorithms
    bfs (start: INTEGER): LIST [INTEGER]
    dfs (start: INTEGER): LIST [INTEGER]
    dijkstra (start, target: INTEGER): LIST [INTEGER]
    shortest_path_cost (start, target: INTEGER): REAL_64
    is_connected: BOOLEAN
    connected_components: LIST [LIST [INTEGER]]
    topological_sort: LIST [INTEGER]  -- For DAGs
    has_cycle: BOOLEAN
```

### Contract Strategy

**Preconditions:**
```eiffel
add_edge (from_id, to_id: INTEGER)
    require
        from_exists: has_node (from_id)
        to_exists: has_node (to_id)
        no_self_loop: from_id /= to_id  -- or allow if needed
```

**Postconditions:**
```eiffel
add_node (data: G): INTEGER
    ensure
        node_added: has_node (Result)
        count_increased: node_count = old node_count + 1
        data_stored: node_data (Result) = data
```

### Integration Plan
- Add to SERVICE_API: `new_graph`, `new_directed_graph`, `new_string_graph`
- No dependencies on other simple_* libraries (only base)
- Future: Visualization support via simple_web SVG generation

### Test Strategy
- Node/edge CRUD operations
- BFS/DFS traversal correctness
- Dijkstra shortest path (weighted, unweighted)
- Connected components
- Cycle detection
- Edge cases: empty graph, single node, disconnected

---

## Step 7: Implementation Assessment

### Current simple_graph Status

**What's Implemented:**
- Basic SIMPLE_GRAPH [G] class
- Adjacency list representation
- Directed/undirected support
- add_node, add_edge, has_node, has_edge
- BFS, DFS traversal
- Dijkstra shortest path
- Basic tests (28 passing in simple_mq reference)

**What's Missing (Based on Research):**
1. **remove_node, remove_edge** - Mutation support
2. **edge_weight accessor** - Query edge weights
3. **connected_components** - Graph analysis
4. **topological_sort** - DAG support
5. **has_cycle** - Cycle detection
6. **is_connected** - Connectivity check
7. **Node data update** - set_node_data
8. **A* algorithm** - Heuristic search
9. **MST algorithms** - Prim/Kruskal

**Contract Gaps:**
- Need stronger preconditions on algorithms
- Missing postconditions on traversals
- No invariants defined

### Recommendations

1. **Add missing algorithms** from research findings
2. **Strengthen contracts** based on pain point analysis
3. **Add edge removal** for graph mutation
4. **Consider MST** for common use case
5. **Document performance** characteristics

---

## Checklist

- [x] Formal specifications reviewed (Step 1)
- [x] Top 5 libraries in other languages studied (Step 2)
- [x] Eiffel ecosystem researched (Step 3)
- [x] Developer pain points documented (Step 4)
- [x] Innovation opportunities identified (Step 5)
- [x] Design strategy synthesized (Step 6)
- [x] Implementation assessment completed (Step 7)
- [ ] Missing features implemented
- [ ] Contracts strengthened
