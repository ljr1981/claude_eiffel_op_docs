# simple_math Research Notes

## Step 1: Specifications

### BLAS (Basic Linear Algebra Subprograms)
The de facto standard for low-level linear algebra operations.

**Level 1 BLAS** - Vector operations O(n):
- Vector addition (AXPY): y = αx + y
- Dot product: x·y
- Vector norms: ||x||
- Scalar multiplication

**Level 2 BLAS** - Matrix-vector operations O(n²):
- Matrix-vector multiplication: y = Ax
- Triangular solve: Tx = b
- Rank-1 update: A = A + xy^T

**Level 3 BLAS** - Matrix-matrix operations O(n³):
- Matrix multiplication: C = AB
- Triangular matrix multiplication
- Rank-k update

### LAPACK (Linear Algebra PACKage)
Built on BLAS, provides higher-level operations:
- LU, QR, Cholesky, Schur decomposition
- Eigenvalue problems
- Singular Value Decomposition (SVD)
- Linear system solving
- Least squares

### IEEE 754 Floating Point
Standard for binary floating-point arithmetic:
- Single precision (32-bit): ~7 decimal digits
- Double precision (64-bit): ~15 decimal digits
- Special values: ±∞, NaN
- Rounding modes

### Statistics Standards
- Mean, median, mode
- Variance, standard deviation
- Covariance, correlation
- Percentiles, quartiles
- Regression (linear, polynomial)

Sources:
- [BLAS - Netlib](https://www.netlib.org/blas/)
- [LAPACK - Wikipedia](https://en.wikipedia.org/wiki/LAPACK)
- [BLAS - Wikipedia](https://en.wikipedia.org/wiki/Basic_Linear_Algebra_Subprograms)

---

## Step 2: Tech-Stack Library Analysis

### Python - NumPy/SciPy
**Strengths:**
- De facto standard for scientific Python
- Efficient n-dimensional arrays
- Broadcasting for element-wise operations
- Comprehensive linear algebra (numpy.linalg)
- Mature ecosystem

**API Patterns:**
```python
v = np.array([1, 2, 3])
m = np.zeros((3, 3))
m = np.eye(3)  # Identity
dot = np.dot(v1, v2)
det = np.linalg.det(m)
inv = np.linalg.inv(m)
```

**Pain Points:**
- Copy vs view confusion
- `*` is element-wise, not matrix multiply
- Deprecated `numpy.matrix` class
- Indexing mental model (row-major)

### Rust - nalgebra
**Strengths:**
- Pure Rust, no external dependencies
- Compile-time dimension checking
- no_std compatible
- 1D and 2D arrays only (focused)

**API Patterns:**
```rust
let v = Vector3::new(1.0, 2.0, 3.0);
let m = Matrix3::identity();
let det = m.determinant();
let inv = m.try_inverse();
```

### Rust - ndarray
**Strengths:**
- N-dimensional arrays (like NumPy)
- NumPy-like syntax
- ndarray-linalg for advanced operations
- ndarray-stats for statistics

**Key Difference:**
nalgebra = 1D/2D only (linear algebra focus)
ndarray = N-dimensional (general arrays)

### JavaScript - math.js
**Strengths:**
- Comprehensive math functions
- Matrix support with element-wise operations
- Symbolic computation
- BigNumber support

**Limitations:**
- Not high-performance
- General purpose, not specialized

### C++ - Armadillo
**Strengths:**
- MATLAB-like syntax
- High performance (BLAS/LAPACK backend)
- Template-based

Sources:
- [ndarray for NumPy users](https://docs.rs/ndarray/latest/ndarray/doc/ndarray_for_numpy_users/index.html)
- [nalgebra vs ndarray](https://users.rust-lang.org/t/ndarray-vs-nalgebra-which-is-best/88699)
- [math.js matrices](https://mathjs.org/docs/datatypes/matrices.html)

---

## Step 3: Eiffel Ecosystem

### Existing Solutions
**No dedicated math library found** in the Eiffel ecosystem:
- Gobo: Data structures, no math
- EiffelBase: Basic structures, no linear algebra
- EiffelMath: Not found

### Gap Analysis
Major opportunity - Eiffel lacks:
- Vector/matrix classes
- Linear algebra operations
- Statistics utilities
- Numerical methods

### Design Influence
Eiffel's strengths for math library:
- Contracts for numerical validity (non-singular matrix)
- Clear preconditions (dimension matching)
- Generic types for element types

Sources:
- [Comparison of linear algebra libraries](https://en.wikipedia.org/wiki/Comparison_of_linear_algebra_libraries)

---

## Step 4: Developer Pain Points

### NumPy Specific
1. **Copy vs View**: Arrays can share data unexpectedly
2. **Operator Confusion**: `*` is element-wise, `@` is matrix multiply
3. **Matrix Class Deprecated**: numpy.matrix discouraged since 1.15
4. **Indexing**: Row-major vs column-major mental model

### General Math Library Issues
1. **Dimension Mismatch**: Runtime errors for incompatible sizes
2. **Singular Matrix**: Division by zero in inverse
3. **Numerical Stability**: Floating-point precision issues
4. **Performance**: Naive implementations too slow
5. **Memory**: Large matrices exhaust memory

### What Developers Want
1. **Clear API**: Intuitive method names
2. **Dimension Safety**: Catch mismatches early
3. **Sensible Defaults**: Works for common cases
4. **Error Messages**: Explain what went wrong
5. **Documentation**: Examples for each operation

Sources:
- [NumPy beginners guide](https://numpy.org/doc/stable/user/absolute_beginners.html)
- [Hacker News NumPy API discussion](https://news.ycombinator.com/item?id=24923654)

---

## Step 5: Innovation Opportunities

### simple_math Differentiators

1. **Contract-Based Dimension Safety**
```eiffel
multiply (other: SIMPLE_MATRIX): SIMPLE_MATRIX
    require
        dimensions_compatible: cols = other.rows
    ensure
        result_rows: Result.rows = rows
        result_cols: Result.cols = other.cols
```

2. **No Copy/View Confusion**
- All operations return new objects
- Explicit `copy` when needed
- No hidden sharing

3. **Clear Operator Semantics**
- `multiply` for matrix multiplication
- `element_multiply` for element-wise
- No operator overloading confusion

4. **Fluent API**
```eiffel
result := matrix.transpose.multiply (other).inverse
```

5. **Built-in Validation**
- `is_square`, `is_symmetric`, `is_invertible`
- Check before operations that require properties

6. **Statistics Integration**
```eiffel
stats := create {SIMPLE_STATISTICS}.make
stats.add_all (data_array)
mean := stats.mean
std_dev := stats.standard_deviation
```

7. **SCOOP-Safe Design**
- Immutable results
- No shared mutable state

---

## Step 6: Design Strategy

### Core Design Principles
- **Simple**: Common operations easy, advanced possible
- **Safe**: Contracts catch dimension errors
- **Pure Eiffel**: No external C dependencies
- **Reasonable Performance**: O(n³) matrix operations acceptable

### API Surface

#### SIMPLE_VECTOR
```eiffel
class SIMPLE_VECTOR

create
    make,              -- Zero vector of dimension n
    make_from_array    -- From ARRAY [REAL_64]

feature -- Access
    dimension: INTEGER
    item (i: INTEGER): REAL_64
    is_zero: BOOLEAN
    magnitude: REAL_64

feature -- Operations
    add (other: SIMPLE_VECTOR): SIMPLE_VECTOR
    subtract (other: SIMPLE_VECTOR): SIMPLE_VECTOR
    scale (scalar: REAL_64): SIMPLE_VECTOR
    dot (other: SIMPLE_VECTOR): REAL_64
    cross (other: SIMPLE_VECTOR): SIMPLE_VECTOR  -- 3D only
    normalize: SIMPLE_VECTOR

feature -- Comparison
    is_equal (other: SIMPLE_VECTOR): BOOLEAN
    distance (other: SIMPLE_VECTOR): REAL_64
```

#### SIMPLE_MATRIX
```eiffel
class SIMPLE_MATRIX

create
    make,              -- Zero matrix rows x cols
    make_identity,     -- Identity matrix n x n
    make_from_array    -- From 2D array

feature -- Access
    rows, cols: INTEGER
    item (row, col: INTEGER): REAL_64
    row (i: INTEGER): SIMPLE_VECTOR
    column (j: INTEGER): SIMPLE_VECTOR
    is_square, is_symmetric, is_identity: BOOLEAN
    is_invertible: BOOLEAN

feature -- Operations
    add, subtract (other: SIMPLE_MATRIX): SIMPLE_MATRIX
    multiply (other: SIMPLE_MATRIX): SIMPLE_MATRIX
    scale (scalar: REAL_64): SIMPLE_MATRIX
    transpose: SIMPLE_MATRIX
    inverse: SIMPLE_MATRIX
    determinant: REAL_64
    trace: REAL_64

feature -- Decomposition
    lu_decomposition: TUPLE [l, u: SIMPLE_MATRIX]
```

#### SIMPLE_STATISTICS
```eiffel
class SIMPLE_STATISTICS

create
    make

feature -- Data
    add (value: REAL_64)
    add_all (values: ARRAY [REAL_64])
    count: INTEGER

feature -- Central Tendency
    mean: REAL_64
    median: REAL_64
    mode: REAL_64

feature -- Dispersion
    variance: REAL_64
    standard_deviation: REAL_64
    range: REAL_64

feature -- Position
    min, max: REAL_64
    percentile (p: REAL_64): REAL_64
    quartile (q: INTEGER): REAL_64  -- 1, 2, 3
```

### Contract Strategy

**Dimension Preconditions:**
```eiffel
dot (other: SIMPLE_VECTOR): REAL_64
    require
        same_dimension: dimension = other.dimension
```

**Invertibility:**
```eiffel
inverse: SIMPLE_MATRIX
    require
        is_square: is_square
        is_invertible: is_invertible
```

**Statistics Validity:**
```eiffel
mean: REAL_64
    require
        has_data: count > 0
```

### Integration Plan
- Add to SERVICE_API: `new_vector`, `new_matrix`, `new_statistics`
- Convenience: `new_vector_2d`, `new_vector_3d`, `new_identity_matrix`
- Singleton: `math` for constants (pi, e) and utilities

---

## Step 7: Implementation Assessment

### Current simple_math Status

**What's Implemented:**
- SIMPLE_VECTOR: dimension, item, add, subtract, dot, scale, normalize
- SIMPLE_MATRIX: rows, cols, item, add, multiply, transpose, determinant, trace, inverse, is_identity
- SIMPLE_STATISTICS: add, add_all, count, mean, median, variance, std_dev, min, max
- SIMPLE_MATH facade: pi, e, sqrt, abs, sin, cos, etc.

**What's Missing (Based on Research):**
1. **Vector**: cross product (3D), distance, angle_between
2. **Matrix**: LU decomposition, eigenvalues, SVD
3. **Matrix**: solve (Ax = b), is_symmetric, is_positive_definite
4. **Statistics**: mode, percentile, quartiles, correlation, covariance
5. **Numerical**: interpolation (already have lagrange), numerical integration

**Contract Gaps:**
- Need `is_invertible` check before `inverse`
- Need dimension contracts on all operations
- Missing postconditions on many features

### Recommendations

1. **Add missing vector operations** (cross, distance)
2. **Add is_invertible query** and use in precondition
3. **Add correlation/covariance** for statistics
4. **Strengthen contracts** throughout
5. **Add LU decomposition** for linear system solving

---

## Checklist

- [x] Formal specifications reviewed (BLAS, LAPACK, IEEE 754)
- [x] Top libraries studied (NumPy, nalgebra, ndarray, math.js)
- [x] Eiffel ecosystem researched (gap identified)
- [x] Developer pain points documented
- [x] Innovation opportunities identified
- [x] Design strategy synthesized
- [x] Implementation assessment completed
- [ ] Missing features implemented
- [ ] Contracts strengthened
