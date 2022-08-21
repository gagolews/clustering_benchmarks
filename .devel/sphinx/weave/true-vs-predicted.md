



(sec:true-vs-predicted)=
# True vs Predicted Clusters


## Motivation

Let $X=\{\mathbf{x}_1, \dots, \mathbf{x}_n\}$ be the input dataset
that consists of $n$ objects.

As an illustration, in this section we will be considering
the [`wut/x2`](https://github.com/gagolews/clustering-data-v1) dataset,
which consists of 120 points in $\mathbb{R}^2$.




```python
import numpy as np
import pandas as pd
dataset = "https://github.com/gagolews/clustering-data-v1/raw/master/wut/x2"
X = np.loadtxt(dataset + ".data.gz")
X[:5, :]  # preview
## array([[ 0.29087439,  0.27966267],
##        [ 0.24996994, -0.97430785],
##        [ 0.43587577, -0.31895699],
##        [ 0.63048803, -2.15249344],
##        [-1.46511622,  0.36344556]])
```

::::{epigraph}
*To learn more about Python, which we use in this tutorial,
check out Marek's recent open-access (free!) textbook*
[Minimalist Data Wrangling in Python](https://datawranglingpy.gagolewski.com/)
{cite}`datawranglingpy`.
*Also note that in a {ref}`further section <sec:how-to-access>`,
we explain how to use our benchmark framework in other programming
environments.*
::::


With each dataset like the one above,
we are given a reference[^footmanyreference]
partition assigned by experts, representing
a desired grouping of the points into $k \ge 2$ clusters.

[^footmanyreference]: {ref}`Later <sec:many-partitions>` we will note
    that there may be many possible ways to split a dataset into groups.





```python
(y_true := np.loadtxt(dataset+".labels0.gz", dtype=np.intc))
## array([2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2,
##        2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 1, 1, 1, 1,
##        1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,
##        1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,
##        1, 1, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3,
##        3, 3, 3, 3, 3, 3, 3, 3, 3, 3], dtype=int32)
```


More formally, we have here a
$k$-partition[^footpart] $\{X_1,\dots,X_k\}$ of $X$
encoded by means of a *label vector*[^footsurj] $\mathbf{y}$, where
$y_i\in[1:k]$ gives the cluster number (ID) of the $i$-th object.


[^footpart]: We say that $\{X_1,\dots,X_k\}$ is a $k$-partition of $X$,
    whenever $\bigcup_{i=1}^k X_i=X$, each $X_i$ is nonempty,
    and the subsets are pairwise disjoint, i.e., $X_i\cap X_j=\emptyset$
    for $i\neq j$.

[^footsurj]: More precisely, a surjection $[1:n]\stackrel{\text{onto}}{\to}[1:k]$.


The number of clusters $k\ge 2$ is thus an inherent part of the
reference set.



```python
(k := np.max(y_true))
## 3
```



Now, let say that we have a clustering algorithm whose usefulness we would
like to assess. We apply it on $X$ to discover a new $k$-partition.

For example, let's consider the output of the
[*Genie*](https://genieclust.gagolewski.com) algorithm:



```python
import genieclust
g = genieclust.Genie(n_clusters=k)  # using default parameters
(y_pred := g.fit_predict(X) + 1)  # +1 makes cluster IDs in 1..k, not 0..(k-1)
## array([1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,
##        1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 2, 1,
##        2, 2, 2, 2, 2, 2, 2, 2, 1, 2, 2, 2, 1, 1, 2, 3, 2, 2, 2, 2, 2, 1,
##        2, 2, 2, 2, 2, 1, 2, 2, 2, 2, 2, 1, 2, 2, 2, 1, 2, 1, 2, 2, 2, 2,
##        2, 1, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3,
##        3, 3, 3, 3, 3, 3, 3, 3, 3, 3])
```

We thus obtained a label vector $\hat{\mathbf{y}}$
encoding a different partition, $\{\hat{X}_1,\dots,\hat{X}_k\}$.

The figure below depicts the two partitions side by side.




```python
plt.subplot(1, 2, 1)
genieclust.plots.plot_scatter(X, labels=y_true-1)
plt.axis("equal")
plt.title("y_true")
plt.subplot(1, 2, 2)
genieclust.plots.plot_scatter(X, labels=y_pred-1)
plt.axis("equal")
plt.title("y_pred")
plt.show()
```

(fig:partition-similarity-example-2)=
```{figure} true-vs-predicted-figures/partition-similarity-example-2-1.*
Two example paritions that we would like to compare
```

Our aim is to *relate* the predicted labels to the reference ones.

::::{important}
Ideally, we would like to work with algorithms that output partitions
matching the reference ones closely on as wide a set of problem tasks
as possible.

Note that the automated discovery of $\hat{\mathbf{y}}$
never relies on the information included in $\mathbf{y}$.
In other words, clustering is an unsupervised[^footsemisupervised]
learning process.
::::


[^footsemisupervised]: Tasks such as semi-supervised clustering,
    where the right assignment of *some* of the input points is known in
    advance, are of our interest here. However, the current framework can
    trivially be adjusted to fit such scenarios.




## Confusion Matrix

We can determine the confusion matrix $\mathbf{C}$ for the two label vectors:



```python
(C := genieclust.compare_partitions.confusion_matrix(y_true, y_pred))
## array([[12, 37,  1],
##        [40,  0,  0],
##        [ 0,  0, 30]])
```

Here, $c_{u, v}$ denotes the number of points in the $u$-th reference cluster
that the algorithms assigned as members of the $v$-th cluster.

TODO:
A partition is a *set* of point groups, and sets are unordered by definition

red (cluster ID of 2) <-> black (cluster ID of 1)


## Measuring Partition Similarity

A confusion matrix nicely summarises all the information we need to
judge the clustering quality. However,
if we wish to compare the quality of different algorithms,
we would rather have the information summarised as a single number.

By using *partition similarity scores*,
we can quantify the similarity between $\mathbf{y}$ and $\hat{\mathbf{y}}$,
and thus assess the degree to which the output of a clustering algorithm
matches the reference (ground truth) labels.

TODO ...

ARI only here

more: {ref}`sec:partition-similarity-scores`


some axioms

