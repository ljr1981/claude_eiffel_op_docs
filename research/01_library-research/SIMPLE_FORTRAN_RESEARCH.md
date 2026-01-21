# simple_fortran Research Report

**Date**: 2024-12-26
**Feasibility**: HIGH
**Recommendation**: BUILD (niche but valuable)

## Executive Summary

Fortran remains the gold standard for numerical computing. ISO_C_BINDING (Fortran 2003+) provides standardized C interoperability. Access to BLAS, LAPACK, and decades of optimized numerical libraries makes Fortran integration valuable for scientific computing.

## Step 1: Problem Definition

**Goal**: Enable Eiffel to call Fortran numerical libraries for scientific computing.

**Use Cases**:
- Linear algebra (BLAS, LAPACK)
- Numerical simulations
- Legacy Fortran codebases
- High-performance computing

## Step 2: Technical Research

### ISO_C_BINDING

Fortran 2003 introduced standardized C interoperability:

```fortran
module math_lib
    use iso_c_binding
    implicit none
contains

    function add(a, b) result(c) bind(C, name="fortran_add")
        real(c_double), intent(in), value :: a, b
        real(c_double) :: c
        c = a + b
    end function add

    subroutine matrix_multiply(A, B, C, m, n, k) bind(C, name="fortran_matmul")
        integer(c_int), intent(in), value :: m, n, k
        real(c_double), intent(in) :: A(m, k), B(k, n)
        real(c_double), intent(out) :: C(m, n)
        C = matmul(A, B)
    end subroutine matrix_multiply

end module math_lib
```

Build:
```bash
gfortran -shared -o mathlib.dll math_lib.f90
# or Intel Fortran:
ifort /dll math_lib.f90
```

### C Type Mapping

| Fortran | C | Eiffel |
|---------|---|--------|
| `integer(c_int)` | `int` | `INTEGER_32` |
| `integer(c_long)` | `long` | `INTEGER_64` |
| `real(c_float)` | `float` | `REAL_32` |
| `real(c_double)` | `double` | `REAL_64` |
| `logical(c_bool)` | `_Bool` | `BOOLEAN` |
| `character(c_char)` | `char` | `CHARACTER_8` |

### BLAS/LAPACK Access

```fortran
! Wrapper for DGEMM (matrix multiplication)
subroutine my_dgemm(A, B, C, m, n, k) bind(C, name="my_dgemm")
    use iso_c_binding
    integer(c_int), intent(in), value :: m, n, k
    real(c_double), intent(in) :: A(m, k), B(k, n)
    real(c_double), intent(out) :: C(m, n)

    call dgemm('N', 'N', m, n, k, 1.0d0, A, m, B, k, 0.0d0, C, m)
end subroutine
```

### Eiffel Integration

```eiffel
class SIMPLE_FORTRAN

feature -- Matrix Operations

    matrix_multiply (a, b: ARRAY2[REAL_64]): ARRAY2[REAL_64]
        require
            compatible: a.width = b.height
        local
            m, n, k: INTEGER
            a_ptr, b_ptr, c_ptr: POINTER
        do
            m := a.height
            n := b.width
            k := a.width

            create Result.make_filled (0.0, m, n)

            a_ptr := a.area.base_address
            b_ptr := b.area.base_address
            c_ptr := Result.area.base_address

            c_fortran_matmul (a_ptr, b_ptr, c_ptr, m, n, k)
        ensure
            result_size: Result.height = a.height and Result.width = b.width
        end

feature {NONE} -- C Externals

    c_fortran_matmul (a, b, c: POINTER; m, n, k: INTEGER)
        external "C inline use %"fortran_lib.h%""
        alias "fortran_matmul($a, $b, $c, $m, $n, $k);"
        end

end
```

## Step 3: Ecosystem Compatibility

| Requirement | Solution |
|-------------|----------|
| SCOOP compatibility | Fortran is inherently thread-safe for pure functions |
| Void safety | Fortran doesn't use pointers much |
| DBC | Contracts on Eiffel side |
| Windows support | gfortran, ifort both work |

## Step 4: Value Proposition

Access to optimized libraries:
- **BLAS**: Basic Linear Algebra Subprograms
- **LAPACK**: Linear Algebra PACKage
- **FFTW**: Fast Fourier Transform
- **ScaLAPACK**: Distributed memory LAPACK
- **Intel MKL**: Intel Math Kernel Library

## Step 5: Design Decisions

1. **Focus**: Numerical operations, not general Fortran
2. **Approach**: Wrap BLAS/LAPACK via simple interface
3. **Arrays**: Map Eiffel ARRAY2 to Fortran column-major

### Proposed Classes

| Class | Purpose |
|-------|---------|
| SIMPLE_FORTRAN | Facade - matrix ops |
| FORTRAN_MATRIX | Column-major matrix wrapper |
| FORTRAN_BLAS | BLAS Level 1/2/3 wrappers |
| FORTRAN_LAPACK | LAPACK equation solvers |

## Step 6: Conclusion

**Feasibility**: HIGH

**Recommendation**: BUILD (niche value)

**Rationale**:
- ISO_C_BINDING is standardized and works well
- Access to 50+ years of optimized numerical code
- BLAS/LAPACK are industry standard
- Valuable for scientific/numerical Eiffel applications

**Estimated Effort**: 1-2 phases

**Target Users**: Scientists, engineers, numerical computing

---

*Research completed as part of cross-language interoperability investigation.*
