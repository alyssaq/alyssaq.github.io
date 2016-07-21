title: Simple Movie Recommender using SVD
date: 2015-04-26 14:13
tags:
- computer vision
- linear algebra
- python
categories:
- data science
---

Given a movie title, we'll use [Singular Value Decomposition (SVD)](http://mathworld.wolfram.com/SingularValueDecomposition.html) to recommend other movies based on user ratings.

Filtering and recommending based on information given by other users is known as [collaborative filtering](http://en.wikipedia.org/wiki/Collaborative_filtering). The assumption is that people with similar movie tastes are most likely to give similar movie ratings. So, if I'm looking for a new movie and I've watched The Matrix, this method will recommend movies that have a similar rating pattern to The Matrix across a set of users.

## SVD Concept
The essence of SVD is that it decomposes a matrix of any shape into a product of 3 matrices with nice mathematical properties: $A = U S V^T$. 

By lucid analogy, a number can decompose into 3 numbers to always have the smallest prime in the middle. E.g $24 = 3 \times 2 \times 4$ or $57 = 1 \times 3 \times 19$.

For the interested, I previously wrote a post on [SVD visualisation](https://alyssaq.github.io/2015/singular-value-decomposition-visualisation) to view the properties of the decomposition.

The result of the decomposition leaves us with an ordered matrix of singular values which encompass the variance associated with every direction. We assume that larger variances means less redundancy and less correlation and encode more structure about the data. This allows us to use a representative subset of user rating directions or principal components to recommend movies.

I highly recommend reading [John Shlen's tutorial on PCA and SVD (2014)](http://arxiv.org/pdf/1404.1100.pdf) to fully understand the mathematical properties of the  two related methods.

## Simple Recommender
Python libraries we'll be using:
<pre><code class="language-python">
import numpy as np
import pandas as pd
</code></pre>

We'll be using 2 files from the [MovieLens 1M dataset](http://grouplens.org/datasets/movielens): `ratings.dat` and `movies.dat`.

1) Read the files with pandas
<pre><code class="language-python">
data = pd.io.parsers.read_csv('data/ratings.dat', 
	names=['user_id', 'movie_id', 'rating', 'time'],
	engine='python', delimiter='::')
movie_data = pd.io.parsers.read_csv('data/movies.dat',
	names=['movie_id', 'title', 'genre'],
	engine='python', delimiter='::')
</code></pre>

2) Create the ratings matrix of shape ($m \times u$) with rows as movies and columns as users
<pre><code class="language-python">
ratings_mat = np.ndarray(
	shape=(np.max(data.movie_id.values), np.max(data.user_id.values)),
	dtype=np.uint8)
ratings_mat[data.movie_id.values-1, data.user_id.values-1] = data.rating.values
</code></pre>

3) Normalise matrix (subtract mean off)
<pre><code class="language-python">
normalised_mat = ratings_mat - np.asarray([(np.mean(ratings_mat, 1))]).T
</code></pre>

4) Compute SVD
<pre><code class="language-python">
A = normalised_mat.T / np.sqrt(num_movies - 1)
U, S, V = np.linalg.svd(A)
</code></pre>

5) Calculate [cosine similarity](http://en.wikipedia.org/wiki/Cosine_similarity), sort by most similar and return the top N.
<pre><code class="language-python">
def top_cosine_similarity(data, movie_id, top_n=10):
    index = movie_id - 1 # Movie id starts from 1
    movie_row = data[index, :]
    magnitude = np.sqrt(np.einsum('ij, ij -> i', data, data))
    similarity = np.dot(movie_row, data.T) / (magnitude[index] * magnitude)
    sort_indexes = np.argsort(-similarity)
    return sort_indexes[:top_n]

# Helper function to print top N similar movies
def print_similar_movies(movie_data, movie_id, top_indexes):
    print('Recommendations for {0}: \n'.format(
    movie_data[movie_data.movie_id == movie_id].title.values[0]))
    for id in top_indexes + 1:
        print movie_data[movie_data.movie_id == id].title.values[0]
</code></pre>

6) Select $k$ principal components to represent the movies, a `movie_id` to find recommendations and print the `top_n` results.
<pre><code class="language-python">
k = 50
movie_id = 1 # Grab an id from movies.dat
top_n = 10

sliced = V.T[:, :k] # representative data
indexes = top_cosine_similarity(sliced, movie_id - 1, top_n)
print_similar_movies(movie_data, movie_id, indexes)
</code></pre>

	Recommendations for Toy Story (1995): 

    Toy Story (1995)
    Toy Story 2 (1999)
    Babe (1995)
    Bug's Life, A (1998)
    Pleasantville (1998)
    Babe: Pig in the City (1998)
    Aladdin (1992)
    Stuart Little (1999)
    Secret Garden, The (1993)
    Tarzan (1999)
    
We can change `k` and use different number of principal components to represent our dataset. This is essentially performing [dimensionality reduction](http://en.wikipedia.org/wiki/Dimensionality_reduction). 
 
## SVD and PCA relationship
Instead of computing SVD in step 4 above, the same results can be obtained by [computing PCA using the eigenvectors of the co-variance matrix](http://en.wikipedia.org/wiki/Principal_component_analysis#Computing_PCA_using_the_covariance_method):

<pre><code class="language-python">
normalised_mat = ratings_mat - np.matrix(np.mean(ratings_mat, 1)).T
cov_mat = np.cov(normalised_mat)
evals, evecs = np.linalg.eig(cov_mat)
</code></pre>

We re-use the same cosine similarity calculation in step 5. Instead of the matrix `V` from SVD, we can use the eigenvectors computed from the co-variance matrix:
<pre><code class="language-python">
k = 50
movie_id = 1 # Grab an id from movies.dat
top_n = 10

sliced = evecs[:, :k] # representative data
top_indexes = top_cosine_similarity(sliced, movie_id, top_n)
print_similar_movies(movie_data, movie_id, top_indexes)
</code></pre>

	Recommendations for Toy Story (1995): 

    Toy Story (1995)
    Toy Story 2 (1999)
    Babe (1995)
    Bug's Life, A (1998)
    Pleasantville (1998)
    Babe: Pig in the City (1998)
    Aladdin (1992)
    Stuart Little (1999)
    Secret Garden, The (1993)
    Tarzan (1999)

Exactly the same results!     
In step 4 above, our input matrix $A$ has shape $u \times m$. The computation of `V` from SVD is the result of the eigenvectors of $A^T A$. The columns of `V` are the eigenvectors that correspond to the sorted eigenvalues in the diagonal of $S$. 

By construction, $A^T A$ equals the covariance matrix of `normalised_mat`. Thus, the columns of $V$ are the principal components of `normalised_mat`. (Refer to section VI of [John Shlen's tutorial (2014)](http://arxiv.org/pdf/1404.1100.pdf) for the full mathematical proof of this relationship).

### Why use SVD over the covariance matrix?
  * Its faster (Facebook published a [fast randomized SVD](https://research.facebook.com/blog/294071574113354/fast-randomized-svd))
  * Singular values from SVD are sorted (we have to sort the eigenvalues in ascending order)
  
  
  