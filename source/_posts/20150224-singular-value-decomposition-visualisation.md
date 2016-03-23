title: Singular Value Decomposition (SVD) visualisation
date: 2015-02-24 03:03
tags:
- computer vision
- linear algebra
- python
categories:
- data science
---

["The SVD is absolutely a high point of linear algebra" (Gilbert Strang, Kai Borre)](https://books.google.com.sg/books?id=MjNwWUY8jx4C&pg=PA259)

This mathematical technique has been used across various industries and example applications include [recommender systems (Netflix prize)](http://www2.research.att.com/~volinsky/papers/ieeecomputer.pdf), [face and object recognition](http://www.iaarc.org/publications/fulltext/isarc2007-4.5_4_035.pdf), [risk modelling in equity options](http://www.orie.cornell.edu/engineering2/customcf/iws_events_calendar/files/Marco_Avellaneda_Presentation_10_16_14.pdf) and [identifying genes in brain imaging that make up Parkinson disease](http://www.ncbi.nlm.nih.gov/pubmed/12045141).

## Definition
SVD is a matrix factorisation and decomposes a matrix of any size into a  product of 3 matrices:

$$ A = U S V^T $$

$A$ : $n \times m$ : number of records as rows and number of dimensions/features as columns.    
$U$ : $n \times n$ : orthogonal matrix containing eigenvectors of $AA^T$.   
$S$ : $n \times m$ : ordered singular values in the diagonal. Square root of eigenvalues associated with $AA^T$ or $A^TA$ (its the same).   
$V$ : $m \times m$ : orthogonal matrix containing eigenvectors of $A^TA$.

_Orthogonal matrix_: square matrix where columns make $90^\circ$ angles between each other and its inner dot product is zero. $Q^TQ = QQ^T = I$ and $Q^T=Q^{-1}$.    
_Orthonormal matrix_: orthogonal matrix where columns are unit vectors.

## Example matrix
For this post, Im going to use the same matrices from [my post on eigenvectors and eigenvalues](https://alyssaq.github.io/2015/understanding-eigenvectors-and-eigenvalues-visually). We have a matrix $x = \begin{bmatrix}
-10 & -10 & 20 & 20\\\
-10 & 20 & 20 & -10
\end{bmatrix}$ and a transformation matrix $A = \begin{bmatrix}
1 & 0.3 \\\
0.45 & 1.2
\end{bmatrix}$. In the plot below, the dashed square shows $x$ as the corners and the transformed matrix $Ax$ as the solid shape.

![transformed_plot](https://alyssaq.github.io/blog/images/eigens-transformation_matrix.png)

Python's numpy provides us with a handy [`svd`](http://docs.scipy.org/doc/numpy/reference/generated/numpy.linalg.svd.html) function:
<pre><code class="language-python">
In[1]:
A = np.matrix([[1, 0.3], [0.45, 1.2]])
U, s, V = np.linalg.svd(A)

Out[1]:
U:[[-0.58189652 -0.81326284]
   [-0.81326284  0.58189652]]
s:[ 1.49065822  0.71444949]
V:[[-0.63586997 -0.77179621]
   [-0.77179621  0.63586997]]
</code></pre>

Lets verify some properties of the SVD matrices.
<pre><code class="language-python">
# Verify calculation of A=USV
In[2]:  np.allclose(A, U * np.diag(s) * V)
Out[2]: True

# Verify orthonormal properties of U and V. (Peformed on U but the same applies for V).
#  1) Dot product between columns = 0
In[3]:  np.round([np.dot(U[:, i-1].A1,  U[:, i].A1) for i in xrange(1, len(U))])
Out[3]: [ 0. 0.]

#  2) Columns are unit vectors (length = 1)
In[4]:  np.round(np.sum((U*U), 0))
Out[4]: [[ 1.  1.  1.]]

#  3) Multiplying by its transpose = identity matrix
In[5]:  np.allclose(U.T * U, np.identity(len(U)))
Out[5]: True
</code></pre>

## Geometric interpretation of SVD
Transformation of a matrix by $U S V^T$ can be visualised as a _rotation and reflection_, _scaling_, _rotation and reflection_. We'll see this as a step-by-step visualisation.


#### (1) $V^Tx$
We can see that multiplying by $V^T$ rotates and reflects the input matrix $x$. Notice the swap of colours red-blue and green-yellow indicating a reflection along the x-axis.
![Vx_plot](https://alyssaq.github.io/blog/images/svd_Vx.png)

#### (2) $SV^Tx$
Since $S$ only contains values on the diagonal, it simply scales the matrix. The singular values $S$ are ordered in descending order so $s_1 > s_2 > ... > s_n$. $V$ rotates the matrix to a position where the singular values now represent the scaling factor along the x and y-axis. This is now the _V-basis_.

The black dots shows this ($V^Tx$ is dashed and $SV^Tx$ solid):

 * $V^Tx$ intercepts the x-axis at $x = -25.91$.
 * Largest singular value of $s_1 = 1.49$ is applied and $s_1x = -38.61$.
 * $V^Tx$ intercepts the y-axis at $y = 12.96$.
 * Smallest singular value of $s_2 = 0.71$ is applied and $s_2y = 9.20$.

![SVx_plot](https://alyssaq.github.io/blog/images/svd_SVx.png)

While not shown, there is a similar rotation and scaling effect in the _U-basis_ with $Ux$ and $SUx$.

#### (3) $USV^Tx$
Finally, $U$ rotates and reflects the matrix back to the standard basis. As expected, this is exactly the same as $Ax$.

![USVx_plot](https://alyssaq.github.io/blog/images/svd_USVx.png)

## Eigen decomposition vs singular value decomposition
In an eigen-decomposition, $A$ can be represented by a product of its eigenvectors $Q$ and diagonalised eigenvalues $\Lambda$:
 $$ A = Q \Lambda Q^{-1}$$.

* Unlike an eigen-decomposition where the matrix must be square ($n \times n$), SVD can decompose a matrix of any dimension.
* Column vectors in $Q$ are not always orthogonal so the change in basis is not a simple rotation. $U$ and $V$ are orthogonal and always represent rotations (and reflections).
* Singular values in $S$ are all real and non-negative. The eigenvalues in $\Lambda$ can be complex and have imaginary numbers.

## Resources
SVD ties together the core concepts of linear algebra --- matrix transformations, projections, subspaces, change of basis, eigens, symmetric matrices, orthogonalisation and factorisation. For a complete proof and backgound, I highly recommend the entire [MIT Linear Algebra lectures by Gilbert Strang](http://ocw.mit.edu/courses/mathematics/18-06-linear-algebra-spring-2010/). I also found the examples and proofs for SVD and QR factorisation from the [University of Texas' MOOC](http://www.ulaff.net) useful.

Other posts covering SVD visualisations and its applications:

 * <http://www.ams.org/samplings/feature-column/fcarc-svd>
 * <http://neurochannels.blogspot.sg/2008/02/visualizing-svd.html>
 * <http://www.math.umn.edu/~lerman/math5467/svd.pdf>
 * <http://www.mathworks.com/company/newsletters/articles/professor-svd.html>
