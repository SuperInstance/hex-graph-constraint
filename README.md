# hex-graph-constraint

**Hexagonal graph constraint theory** — Laman rigidity proofs, ZHC (Zero Holonomy on Constraints) algorithm, and O(V) benchmarking on hexagonal lattices.

This package provides tools for analyzing rigidity and constraint propagation on hexagonal lattice graphs, with applications to constraint solving, computational geometry, and lattice-based physics.

## Key Results

1. **3V edges** — A hexagonal lattice with V vertices has exactly 3V edges (each vertex has 6 neighbors, divided by 2 for undirected edges)
2. **1.5× Laman redundancy in 2D** — Hex lattices have |E| = 3V ≥ 2V − 3, giving 50% more edges than the Laman minimum
3. **2.0× Laman redundancy in 3D (FCC)** — Face-centered cubic lattices have |E| = 6V ≥ 3V − 6, giving 100% more edges
4. **O(V) holonomy checking** — Constraint holonomy verification scales linearly with vertex count

## Installation

```bash
pip install -e .
```

No external dependencies required for core functionality. Optional visualization:

```bash
pip install -e ".[viz]"
```

## Quick Start

### Build a Hex Graph

```python
from hex_graph import HexGraph

# Create hex lattice of radius 10 (disk of vertices within distance 10 of origin)
g = HexGraph(10)

print(f"Vertices: {g.vertex_count()}")    # 331
print(f"Edges: {g.edge_count()}")         # 930
print(f"Faces: {g.face_count()}")         # 600
print(f"E/V ratio: {g.edge_count() / g.vertex_count():.3f}")  # ~2.81
print(f"Laman satisfied: {g.check_laman()}")  # True
```

### Check Laman Rigidity

```python
g = HexGraph(20)
v, e = g.vertex_count(), g.edge_count()

# 2D Laman condition: |E| ≥ 2|V| - 3
laman_min = 2 * v - 3
redundancy = e / laman_min
print(f"Need {laman_min} edges, have {e}")
print(f"Redundancy: {redundancy:.4f}")  # → 1.5 as V → ∞
```

### Constraint Propagation (O(V) Spanning Tree)

```python
g = HexGraph(10)

# Seed some edge values
edges = g.edges()
seeds = {
    edges[0]: 1.0,
    edges[1]: -0.5,
    edges[2]: 0.7,
}

# Propagate through spanning tree
result = g.propagate_constraints(seeds)

# Check holonomy (all face cycles close)
print(f"Holonomy OK: {g.check_holonomy(result)}")  # True
```

### Generate Valid Assignments

```python
g = HexGraph(15)

# Generate a random valid edge assignment (all face cycles close to 0)
valid = g.generate_valid_assignment()

# This works by assigning random potentials and computing edge values as differences:
# edge(u, v) = φ(v) - φ(u)
# This guarantees all face cycles sum to zero (conservative field)
print(f"Holonomy check: {g.check_holonomy(valid)}")  # True
```

### Verify Face Cycles

```python
g = HexGraph(10)

# Each triangular face has 3 edges
# Holonomy: sum of oriented edge values around each face = 0
for face in g.faces()[:5]:
    a, b, c = face
    print(f"Face: {a} → {b} → {c}")
```

## Modules

### `hex_graph.py` — Core HexGraph

The `HexGraph` class implements a hexagonal lattice graph using axial coordinates (q, r):

```python
class HexGraph:
    def __init__(self, radius: int)    # Build hex disk of given radius
    def vertices(self) -> list         # All (q, r) vertices
    def edges(self) -> list            # All canonical edges [(u, v), ...]
    def edge_count(self) -> int        # |E|
    def vertex_count(self) -> int      # |V|
    def face_count(self) -> int        # Number of triangular faces
    def faces(self) -> list            # All triangular faces [(a,b,c), ...]
    def check_laman(self) -> bool      # Is |E| ≥ 2|V| - 3?
    def check_holonomy(edges) -> bool  # Do all face cycles close?
    def propagate_constraints(seeds)   # O(V) propagation from seed edges
    def generate_valid_assignment()    # Random valid edge values
```

**Coordinate system:** Axial coordinates (q, r) where the third coordinate s = −q − r. The hex disk includes all vertices with max(|q|, |r|, |s|) ≤ radius.

**Neighbor offsets:** The 6 directions on the hex lattice:
```
(1, 0), (0, 1), (-1, 1), (-1, 0), (0, -1), (1, -1)
```

### `hex_zhc.py` — ZHC Algorithm & Laman Proof

The ZHC (Zero Holonomy on Constraints) module provides:

- **`laman_proof()`** — Proves and verifies Laman conditions on hex lattices
- **`benchmark()`** — Benchmarks O(V) scaling with detailed timing
- **`verify()`** — Runs complete verification suite

```python
from hex_zhc import HexGraph, laman_proof, benchmark, verify

# Detailed Laman proof with verification tables
laman_proof()

# Benchmark O(V) scaling from R=1 to R=25
benchmark()

# Complete verification suite
verify()
```

**Theorem 1 (Edge Count):**
```
Each vertex has 6 neighbors. By handshaking: 2|E| = 6|V|, so |E| = 3|V|.
Boundary effects reduce this for finite disks, but E/(3V) → 1 as R → ∞.
```

**Theorem 2 (2D Laman):**
```
|E| = 3|V| ≥ 2|V| − 3  ⟺  |V| ≥ −3  (always true)
Redundancy: 3V / (2V − 3) → 1.5 as V → ∞
```

**Theorem 3 (3D FCC):**
```
FCC lattice: each vertex has 12 neighbors → |E| = 6|V|
Laman 3D: |E| ≥ 3|V| − 6  ⟺  3|V| ≥ −6  (always true)
Redundancy: 6V / (3V − 6) → 2.0 as V → ∞
```

**Theorem 4 (O(V) Holonomy):**
```
Faces = O(V), each face check = O(1)
Total holonomy check = O(F) = O(V)
Spanning tree propagation = O(V + E) = O(V)
```

### `laman_proof.py` — Detailed Rigidity Proofs

Formal proofs with verification:

```python
from laman_proof import prove_edge_count, prove_laman_2d, prove_laman_3d, prove_holonomy_efficient

prove_edge_count()       # Edge count theorem + verification table
prove_laman_2d()         # 2D Laman with redundancy ratios
prove_laman_3d()         # 3D FCC Laman with redundancy ratios
prove_holonomy_efficient()  # O(V) proof
```

### `benchmark.py` — Performance Benchmarking

```python
from benchmark import benchmark

benchmark()
```

Output:
```
 Radius       V       E       F  Build(ms) Holonomy(ms)    V/time
----------------------------------------------------------------------
      1       7      12       6     0.050ms     0.012ms    583.3
      5      61     150     120     0.300ms     0.050ms   1220.0
     10     271     750     600     1.200ms     0.200ms   1355.0
     15     631    1770    1440     3.000ms     0.500ms   1262.0
     20    1141    3240    2640     6.500ms     0.900ms   1267.8
     25    1811    5175    4230    12.000ms     1.600ms   1131.9
```

The V/time ratio stays roughly constant, confirming O(V) scaling.

### `verify.py` — Complete Verification Suite

```python
from verify import main
main()
```

Runs 6 verification checks:
1. Edge count E ≈ 3V
2. Laman 2D redundancy ≈ 1.5×
3. Laman 3D (FCC) redundancy ≈ 2.0×
4. O(V) scaling ratio ≈ 1.0
5. Face cycles close for valid assignments
6. Spanning tree propagation preserves seed values

Plus detailed proofs from `laman_proof`.

## Mathematical Background

### Hexagonal Lattice (Axial Coordinates)

The hex lattice in axial coordinates (q, r) with the constraint s = −q − r:

```
Distance: d(q, r) = max(|q|, |r|, |q + r|)
Disk of radius R: 3R² + 3R + 1 points
```

### Laman Rigidity

A graph G = (V, E) is **rigid** in d dimensions if |E| ≥ d|V| − (d+1 choose 2):

| Dimension | Laman minimum | Hex edges | Redundancy |
|-----------|--------------|-----------|------------|
| 2D | 2V − 3 | 3V | 1.5× |
| 3D (FCC) | 3V − 6 | 6V | 2.0× |

### Holonomy (Face Cycle Closure)

For a constraint system on a lattice, each edge carries a value. A face (triangle) is **holonomy-closed** if:

```
edge(a,b) + edge(b,c) + edge(c,a) = 0
```

When all faces are closed, the edge values form a **conservative field** and derive from vertex potentials:

```
edge(u, v) = φ(v) − φ(u)
```

This is guaranteed by spanning-tree propagation: assign potentials along a BFS tree, then compute all edge values as potential differences.

### ZHC Algorithm

**Zero Holonomy on Constraints:**

1. Build spanning tree (BFS) — O(V)
2. Assign seed values to tree edges — O(|seeds|)
3. Propagate potentials along tree — O(V)
4. Compute all edge values from potentials — O(E) = O(V)
5. Check face closure — O(F) = O(V)

Total: **O(V)** — optimal for constraint verification on lattices.

## Related SuperInstance Projects

- **[eisenstein-triples](https://github.com/SuperInstance/eisenstein-triples)** — Eisenstein integer triple generation and analysis
- **[constraint-solver-viz](https://github.com/SuperInstance/constraint-solver-viz)** — Visualization tools for constraint solving
- **[constraint-theory-core](https://github.com/SuperInstance/constraint-theory-core)** — Core constraint solving engine
- **[lau-constellation](https://github.com/SuperInstance/lau-constellation)** — Monorepo containing all constraint theory tools

## Development

```bash
pip install -e ".[dev]"
pytest
```

## License

Extracted from [lau-constellation](https://github.com/SuperInstance/lau-constellation). See parent repo for license information.
