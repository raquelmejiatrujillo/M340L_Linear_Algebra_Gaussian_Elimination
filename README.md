M340L: Gaussian Elimination Coding Project
================
Table of Contents

- <a href="#computational-gaussian-elimination"
  id="toc-computational-gaussian-elimination"><strong>Computational:
  Gaussian Elimination</strong></a>
  - <a href="#function" id="toc-function">Function</a>
  - <a href="#test-case-11-x-10" id="toc-test-case-11-x-10">Test Case: 11 x
    10</a>
  - <a href="#other-test-cases" id="toc-other-test-cases">Other Test
    Cases</a>
- <a href="#theoretical-flops"
  id="toc-theoretical-flops"><strong>Theoretical: FLOPs</strong></a>
- <a href="#extension-jacobi-method"
  id="toc-extension-jacobi-method"><strong>Extension: Jacobi
  Method</strong></a>
  - <a href="#iterative-scheme" id="toc-iterative-scheme">Iterative
    Scheme</a>
  - <a href="#convergence" id="toc-convergence">Convergence</a>
  - <a href="#problematic-matrices"
    id="toc-problematic-matrices">Problematic Matrices</a>
  - <a href="#jacobi-method-vs-row-reduction"
    id="toc-jacobi-method-vs-row-reduction">Jacobi Method vs. Row
    Reduction</a>
- <a href="#references"
  id="toc-references"><strong>References</strong></a>

# **Computational: Gaussian Elimination**

## Function

The `SimpleEchelonElimination` function outlined below performs Gaussian
Elimination for an $m \times n$ matrix with partial pivoting to reduce
floating point error.

``` r
SimpleEchelonElimination = function(x){
  # Define matrix dimensions
  n = ncol(x)
  m = nrow(x)
  
  # Define starting pivot position
  pivot_row = 1
  pivot_col = 1
  
  # Set FLOPs counter to zero
  FLOPs = 0 
  
  # ============================================================================
  # Swap two rows 
  # FLOPs per use of `SwapRows`: 0
  # ============================================================================
  SwapRows = function(x, original_first_row, original_second_row){
    temp_first_row = x[original_first_row, ]
    x[original_first_row, ] = x[original_second_row, ]
    x[original_second_row, ] = temp_first_row
    return(x)
  }
  
  # ============================================================================
  # Row elimination 
  # FLOPs per row use of `EliminateRows`: 2n + 1
  # ============================================================================
  EliminateRows = function(x, pivot_row, pivot_col, eliminated_row){
    # Calculate scaling factor (1 FLOP per row)
    scaling = x[eliminated_row, pivot_col]/x[pivot_row, pivot_col]
    
    # Replace the newly eliminated row (2 FLOPs per element)
    x[eliminated_row, ] = x[eliminated_row, ]  - (scaling * x[pivot_row, ])
    return(x)
  }
  
  # ============================================================================
  # Perform elimination to simple echelon form with partial pivoting
  # ============================================================================
  while (pivot_row <= m & pivot_col <= n) {
    # --------------------------------------------------------------------------
    # Partial pivoting 
    # --------------------------------------------------------------------------
    # Current pivot entry
    current_pivot = x[pivot_row, pivot_col]
    # Subset the pivot column from current pivot row to the bottom of the matrix `x`
    subset_below_pivot = x[pivot_row:m, pivot_col]
    # Value of the entry in subset that contains the largest absolute value
    max_entry_pivot_col = max(abs(subset_below_pivot))
    # Index of the row in the original matrix that contains the entry defined above
    new_pivot_row = which(abs(x[, pivot_col]) == abs(max_entry_pivot_col))
    
    # If current pivot is not the max abs pivot value, swap rows:
    if (current_pivot != max_entry_pivot_col & pivot_row != m & pivot_col != n) {
      x = SwapRows(x, pivot_row, new_pivot_row)
    } 
    
    # --------------------------------------------------------------------------
    # Eliminate rows
    # --------------------------------------------------------------------------
    # Define a counter to loop through rows below the pivot row 
    start_elimination = pivot_row + 1
    
    # As long as the pivot row is not the last row, proceed with row elimination:
    if (pivot_row != m) {
      for (current_row in start_elimination:m) {
        x = EliminateRows(x, pivot_row, pivot_col, current_row)
        
        # Keep track of FLOPs per eliminated row 
        FLOPs = FLOPs + (2*n + 1)
      }
    }
    
    # Continue while loop
    pivot_row = pivot_row + 1
    pivot_col = pivot_col + 1
  }
  
  # Generate output
  output = list(FLOPs = paste0("FLOPs: ", FLOPs), Echelon_Matrix = x)
  
  return(output)
}
```

## Test Case: 11 x 10

An $11 \times 10$ `test_matrix` is defined below by sampling entries
from a uniform distribution from 0 to 1.

``` r
set.seed(1)
test_matrix = matrix(data = runif((11*10), min = 0, max = 1), nrow = 11, ncol = 10)
test_matrix
```

    ##             [,1]      [,2]       [,3]      [,4]       [,5]       [,6]
    ##  [1,] 0.26550866 0.1765568 0.65167377 0.1862176 0.52971958 0.09946616
    ##  [2,] 0.37212390 0.6870228 0.12555510 0.8273733 0.78935623 0.31627171
    ##  [3,] 0.57285336 0.3841037 0.26722067 0.6684667 0.02333120 0.51863426
    ##  [4,] 0.90820779 0.7698414 0.38611409 0.7942399 0.47723007 0.66200508
    ##  [5,] 0.20168193 0.4976992 0.01339033 0.1079436 0.73231374 0.40683019
    ##  [6,] 0.89838968 0.7176185 0.38238796 0.7237109 0.69273156 0.91287592
    ##  [7,] 0.94467527 0.9919061 0.86969085 0.4112744 0.47761962 0.29360337
    ##  [8,] 0.66079779 0.3800352 0.34034900 0.8209463 0.86120948 0.45906573
    ##  [9,] 0.62911404 0.7774452 0.48208012 0.6470602 0.43809711 0.33239467
    ## [10,] 0.06178627 0.9347052 0.59956583 0.7829328 0.24479728 0.65087047
    ## [11,] 0.20597457 0.2121425 0.49354131 0.5530363 0.07067905 0.25801678
    ##             [,7]      [,8]       [,9]     [,10]
    ##  [1,] 0.47854525 0.3899895 0.24548851 0.6049333
    ##  [2,] 0.76631067 0.7773207 0.14330438 0.6547239
    ##  [3,] 0.08424691 0.9606180 0.23962942 0.3531973
    ##  [4,] 0.87532133 0.4346595 0.05893438 0.2702601
    ##  [5,] 0.33907294 0.7125147 0.64228826 0.9926841
    ##  [6,] 0.83944035 0.3999944 0.87626921 0.6334933
    ##  [7,] 0.34668349 0.3253522 0.77891468 0.2132081
    ##  [8,] 0.33377493 0.7570871 0.79730883 0.1293723
    ##  [9,] 0.47635125 0.2026923 0.45527445 0.4781180
    ## [10,] 0.89219834 0.7111212 0.41008408 0.9240745
    ## [11,] 0.86433947 0.1216919 0.81087024 0.5987610

The `SimpleEchelonElimination` function is used to row reduce the
`test_matrix` below.

``` r
SimpleEchelonElimination(test_matrix)
```

    ## $FLOPs
    ## [1] "FLOPs: 1155"
    ## 
    ## $Echelon_Matrix
    ##            [,1]          [,2]      [,3]      [,4]      [,5]       [,6]
    ##  [1,] 0.9446753  9.919061e-01 0.8696908 0.4112744 0.4776196 0.29360337
    ##  [2,] 0.0000000  8.698298e-01 0.5426839 0.7560335 0.2135587 0.63166741
    ##  [3,] 0.0000000  0.000000e+00 0.4710189 0.1594782 0.4205791 0.09118309
    ##  [4,] 0.0000000  0.000000e+00 0.0000000 0.8304599 0.6686442 0.49555229
    ##  [5,] 0.0000000  0.000000e+00 0.0000000 0.0000000 0.9615550 0.26982308
    ##  [6,] 0.0000000 -2.775558e-17 0.0000000 0.0000000 0.0000000 0.46355639
    ##  [7,] 0.0000000 -2.319663e-17 0.0000000 0.0000000 0.0000000 0.00000000
    ##  [8,] 0.0000000  8.783279e-18 0.0000000 0.0000000 0.0000000 0.00000000
    ##  [9,] 0.0000000  1.538877e-17 0.0000000 0.0000000 0.0000000 0.00000000
    ## [10,] 0.0000000  2.669286e-17 0.0000000 0.0000000 0.0000000 0.00000000
    ## [11,] 0.0000000  4.935838e-17 0.0000000 0.0000000 0.0000000 0.00000000
    ##                [,7]       [,8]          [,9]      [,10]
    ##  [1,]  3.466835e-01  0.3253522  7.789147e-01  0.2132081
    ##  [2,]  8.695236e-01  0.6898416  3.591393e-01  0.9101296
    ##  [3,]  4.832976e-01  0.3796201  6.877597e-02  0.6519722
    ##  [4,]  4.790617e-01  0.8365767  3.925685e-01  0.4085357
    ##  [5,]  4.022985e-01  0.8093782  4.609766e-01  1.1873243
    ##  [6,]  6.590655e-01 -0.1695321 -5.269415e-02  0.7067609
    ##  [7,]  7.949547e-01 -0.3297916 -7.437696e-01  0.5851939
    ##  [8,]  0.000000e+00  0.8817402 -4.107245e-01  0.8499517
    ##  [9,]  0.000000e+00  0.0000000  1.156399e+00  0.1325684
    ## [10,] -5.551115e-17  0.0000000  0.000000e+00 -0.2911487
    ## [11,] -9.254178e-17  0.0000000  1.387779e-17  0.0000000

## Other Test Cases

The matrices defined below are used to show that the
`SimpleEchelonElimination` function also works for square matrices and
matrices with more columns than rows.

``` r
square_matrix = matrix(data = runif((6*6), min = 0, max = 1), nrow = 6, ncol = 6)
square_matrix
```

    ##            [,1]      [,2]      [,3]       [,4]      [,5]      [,6]
    ## [1,] 0.97617069 0.7155661 0.4843495 0.22865814 0.9286152 0.6827881
    ## [2,] 0.73179251 0.1031842 0.1734423 0.59571200 0.5980924 0.6015412
    ## [3,] 0.35672691 0.4462843 0.7548209 0.57487220 0.5609007 0.2388687
    ## [4,] 0.43147369 0.6401010 0.4538955 0.07706438 0.5260277 0.2581659
    ## [5,] 0.14821156 0.9918386 0.5111698 0.03554058 0.9850952 0.7293096
    ## [6,] 0.01307758 0.4955936 0.2075451 0.64279549 0.5076418 0.4525708

``` r
SimpleEchelonElimination(square_matrix)
```

    ## $FLOPs
    ## [1] "FLOPs: 195"
    ## 
    ## $Echelon_Matrix
    ##           [,1]          [,2]      [,3]         [,4]       [,5]         [,6]
    ## [1,] 0.9761707  7.155661e-01 0.4843495 0.2286581427 0.92861520  0.682788079
    ## [2,] 0.0000000  8.831945e-01 0.4376312 0.0008235159 0.84410399  0.625642214
    ## [3,] 0.0000000  0.000000e+00 0.4862568 0.4911402111 0.04493974 -0.141549573
    ## [4,] 0.0000000 -5.551115e-17 0.0000000 0.6794431962 0.03438000  0.087567585
    ## [5,] 0.0000000  3.263351e-17 0.0000000 0.0000000000 0.29349526  0.352394551
    ## [6,] 0.0000000  1.325410e-17 0.0000000 0.0000000000 0.00000000 -0.001170213

``` r
test_matrix_2 = matrix(data = runif((4*7), min = 0, max = 1), nrow = 4, ncol = 7)
test_matrix_2
```

    ##           [,1]      [,2]       [,3]      [,4]      [,5]       [,6]      [,7]
    ## [1,] 0.1751268 0.6146450 0.50044097 0.2777559 0.4462353 0.06380848 0.6304141
    ## [2,] 0.7466983 0.5571595 0.18086636 0.2126995 0.7799849 0.33548749 0.8406146
    ## [3,] 0.1049876 0.3287773 0.52963060 0.2847905 0.8806190 0.72372595 0.8561317
    ## [4,] 0.8645449 0.4531314 0.07527575 0.8950941 0.4131242 0.33761533 0.3913593

``` r
SimpleEchelonElimination(test_matrix_2)
```

    ## $FLOPs
    ## [1] "FLOPs: 90"
    ## 
    ## $Echelon_Matrix
    ##           [,1]      [,2]       [,3]        [,4]      [,5]         [,6]
    ## [1,] 0.8645449 0.4531314 0.07527575  0.89509410 0.4131242  0.337615333
    ## [2,] 0.0000000 0.5228563 0.48519272  0.09644097 0.3625507 -0.004580656
    ## [3,] 0.0000000 0.0000000 0.26645831  0.12559971 0.6406308  0.685125276
    ## [4,] 0.0000000 0.0000000 0.00000000 -0.57305250 0.3995732  0.143052769
    ##           [,7]
    ## [1,] 0.3913593
    ## [2,] 0.5511383
    ## [3,] 0.5200482
    ## [4,] 0.4020044

# **Theoretical: FLOPs**

In the Gaussian Elimination function above, the row reduction of an
$m \times n$ matrix requires $(\sum\limits_{i=1}^{m-1} i)(2n+1)$
operations.

The Row Elimination portion of the function is copied below with
comments describing the number of FLOPs required for each component of
the function.

``` r
# Calculate scaling factor ----------------------------------
# 1 FLOP per row due to division
scaling = x[eliminated_row, pivot_col]/x[pivot_row, pivot_col]
    
# Replace the newly eliminated row  -------------------------
# 2 FLOPs per element due to subtraction and multiplication
x[eliminated_row, ] = x[eliminated_row, ]  - (scaling * x[pivot_row, ])
```

Thus, for each row that is input to the elimination function above,
$2n + 1$ FLOPs are required.

The total number of FLOPs required in the entire row reduction algorithm
is then calculated by multiplying the number of rows that are input to
the elimination function by $2n + 1$. There are
$\sum\limits_{i=1}^{m-1} i$ rows that are eliminated (under the
assumption that there are not any zero entries in the matrix). Thus,
this row reduction of an $m \times n$ matrix requires
$(\sum\limits_{i=1}^{m-1} i)(2n+1)$ floating point operations.

# **Extension: Jacobi Method**

## Iterative Scheme

The Jacobi Method is useful for approximating a solution to the problem
$Ax = b$ for high-dimensional matrices for which Gaussian Elimination
may be inefficient. The method first decomposes an $n \times n$ matrix
$A$ into its diagonal component (a new matrix $D$) and its non-diagonal
component (a new matrix $N$). The initial problem can now be defined as
$(D+N)x = b$.

After algebraic rearrangement, the equation $x=D^{-1}(b-Rx)$ can be used
to iteratively estimate $x$, such that
$x_{i}^{(k+1)} = \frac{1}{a_{ii}}(b_i - \sum\limits_{j\neq i}^{n} a_{ij} \cdot x_j^k)$,
where $x^k$ is the current estimation of $x$, $x^{(k+1)}$ is the next
estimation of $x$, and $i = 1, 2, ...n$. This process is repeated until
the residuals between $x^k$ and $x^{(k+1)}$ stabilize.

The iterative process must be initialized with a guess of $x$. If
information about the system is known, an initial $x^{(0)}$ can be
defined as such, otherwise the zero vector is typically used as the
initial condition.

## Convergence

This iterative method converges when the matrix $A$ is diagonally
dominant such that $|a_{ii}| > \sum\limits_{j\neq i,j=1}^{n} |a_{ij}|$
for all $i$. The residuals of the approximations of $x$ converge in less
iterations when the initial guess $x^{(0)}$ is close to the real $x$.
The opposite is true when the initial guess is far from the real $x$.
The initial $x^{(0)}$ does not affect whether convergence occurs, but
rather the speed of convergence. The speed of convergence is also faster
for smaller matrices and faster for matrices with smaller conditioning
numbers.

The section below specifies characteristics of matrices that prevent the
Jacobi Method from converging.  

## Problematic Matrices

The Jacobi Method is ineffective for the following types of matrices:

1.  Matrices with at least one zero diagonal entry are problematic
    because a zero in the term $a_{i,i}$ in
    $x_{i}^{(k+1)} = \frac{1}{a_{ii}}(b_i - \sum\limits_{j\neq i}^{n} a_{ij} \cdot x_j^k)$
    results in an undefined approximation of $x_i$. This could be
    addressed by swapping columns so that all diagonal entries of $A$
    are non-zero, if possible, while keeping in mind that the matrix
    should be diagonally dominant.

$$
A_{n,n} =  
\begin{pmatrix}  
a_{1,1} & a_{1,2} & \cdots & a_{1,n} \\  
a_{2,1} & 0 & \cdots & a_{2,n} \\  
\vdots  & \vdots  & \ddots & \vdots  \\  
a_{n,1} & a_{n,2} & \cdots & a_{n,n}  
\end{pmatrix}
$$

2.  Matrices that are not diagonally dominant will not converge using
    the Jacobi Method.

$$
A =  \begin{pmatrix}  
0.1 & 2 & 3 \\  
20 & 0.15 & 8 \\  
  7 & 4 & 0.05  \end{pmatrix}
$$

3.  Ill-conditioned matrices with a large condition number will not
    converge using the Jacobi Method.

$$
A =  \begin{pmatrix}  
0.04 & 0.011 \\  
0.02 & 0.005 \end{pmatrix}
$$

4.  The Jacobi Method cannot approximate solutions for rectangular
    matrices.

## Jacobi Method vs. Row Reduction

Compared to Gaussian Elimination, the Jacobi Method is a more efficient
way to solve the problem $Ax = b$ for an $n \times n$ matrix $A$ that
satisfies the convergence criteria above, since the iterative algorithm
requires $\mathcal O{(n^2)}$ operations, while row reduction requires
$\mathcal O{(n^3)}$ operations.

# **References**

Kaur H, Kaur K. 2012. Convergence of Jacobi and Gauss-Seidel Method and
Error Reduction Factor. IOSR Journal of Mathematics. 2(2): p. 20–23.
doi: 10.9790/5728-0222023.

Pyzara A, Bylina B, Bylina J. 2011. The influence of a matrix condition
number on iterative methods’ convergence. In: 2011 Federated Conference
on Computer Science and Information Systems (FedCSIS). p. 459–464.
<https://ieeexplore.ieee.org/document/6078297>.

Rapp BE. 2017. Numerical Methods for Linear Systems of Equations.
Microfluidics: Modelling, Mechanics and Mathematics. p. 497–535. doi:
10.1016/b978-1-4557-3141-1.50025-3.
