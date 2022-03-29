---
template: BlogPost
path: /cart
mockup: 
thumbnail:
date: 2019-12-11
name: Machine Learning Series - Implementing CART Variance Reduction Measure
title: In this post of Machine Learning Series we will implement CART Variance Reduction Measure using point estimates
category: Machine Learning
description: 'Using CART Variance Reduction measure for splitting criteria.'
tags:
  - Machine Learning 
  - Regression
  - CART
  - Data Engineering
---

## Implementation

We will implement a simple regression tree by using point estimates in the leaves and use the CART Variance Reduction measure for a splitting criteria.

<img src="/assets/ml/cart.png" alt="cart" style="zoom:33%;" />

- For simplicity’s sake, we divide each attribute up into 5 equal sized bins, and test each end point of a bin as a potential split point. Then we test our algorithm on a 50000 row dataset generated using the attached generate test data function. Then we test it against different max depths and report a graph of depth vs R2. 
- We will improve the above method by building a random forest of these trees. Generate 100 of these trees, each with a different bootstrapped sample of X. At each split point, we will select 13 of the features randomly as potential split variables. For values of T ∈ {1,2,5,10,20,30,50,75,100}, we select multiple random boostrapped samples of these 100 trees, and for each forest of size T, predict on a test data set and calculate the mean and median R2 for the forests. Finally these will be plotted as a function of T and calculate the average pairwise correlation of the predictions from each tree.
- The number of features to consider splitting on at each split point a parameter instead of hard-coded will be 13. More trees will be generated with this set to 1 and  the pairwise correction of these trees will be calculated.

```python
import numpy as np
import math
import matplotlib.pyplot as plt
from sklearn.metrics import r2_score
```

```python
# I modified this function by adding larger noise at the final step
def generate_test_data(N):
    x = np.random.randn(N, 5)
    y = np.where(x[:, 0] > 0, 2, 5)
    y = y + np.where(x[:, 1] > 0, -3, 3)
    y = y + np.where(x[:, 2] > 0, 0, 0.5)
    
    # larger noise
    y = y + 10*np.random.randn(N)
    return x, y
```

```python
class TreeNode:
    def predict(x, y):
        assert False

    def depth(self):
        assert False


class BranchNode(TreeNode):
    def __init__(self, left, right, split_var_index, split_var_value):
        self.left = left
        self.right = right
        self.split_var_index = split_var_index
        self.split_var_value = split_var_value

    def predict(self, x):
        svar = x[:, self.split_var_index]
        is_left = svar < self.split_var_value
        leftx = x[is_left]
        rightx = x[~is_left]

        rv = np.zeros(x.shape[0])
        rv[is_left] = self.left.predict(leftx)
        rv[~is_left] = self.right.predict(rightx)

        return rv

    def depth(self):
        return 1 + max(self.left.depth(), self.right.depth())


class LeafNode(TreeNode):
    def __init__(self, mu):
        self.mu = mu

    def predict(self, x):
        return np.repeat(self.mu, x.shape[0])

    def depth(self):
        return 1


class RegressionTree:
    def __init__(self, max_depth, min_points_in_leaf, feature_portion=1):
        self.max_depth = max_depth
        self.min_points_in_leaf = min_points_in_leaf
        self.feature_portion=feature_portion

    def predict(self, x):
        assert self.fitted
        return self.root.predict(x)

    def fit(self, x, y):
        self.fitted = True
        self.root = self.fit_internal(x, y, 1)

    def fit_internal(self, x, y, current_depth):
        # implement this
        num_features = x.shape[1]
        num_rows = x.shape[0]
        var_orig = np.var(y)

        if current_depth == self.max_depth:
            return LeafNode(np.mean(y))

        best_variable = None
        best_split_point = None
        vr_gain = 0

        # Here, we have to loop over all features and figure out which one
        # might be splittable, and if it is, how to split it to maximize Variance Reduction
        
        # randomly choose features as potential split feature
        feature_chosen=np.random.choice(num_features,math.ceil(num_features*self.feature_portion),False)
        for i in feature_chosen:
            # a lot of code goes here
            # pass
            svar = x[:, i]
            max_val = max(svar)
            min_val = min(svar)
            for j in range(4):
                # divide into 5 equal bins
                split_point = (max_val-min_val)/5*(j+1)+min_val
                is_left = svar < split_point
                leftx = x[is_left]
                rightx = x[~is_left]
                lefty = y[is_left]
                righty = y[~is_left]
                
                # if number of points in leaf is too small
                if len(lefty) < self.min_points_in_leaf or len(righty) < self.min_points_in_leaf:
                    continue
                
                # calculate VR gain
                vr_now = var_orig-np.var(lefty)*len(lefty)/(len(lefty)+len(righty)) \
                    - np.var(righty)*len(righty)/(len(lefty)+len(righty))
                
                # if bigger than recorded maximum VR gain, update
                if vr_now > vr_gain:
                    vr_gain = vr_now
                    best_variable = i
                    best_split_point = split_point

        # it is a leaf note
        if best_variable is None:
            return LeafNode(np.mean(y))
        
        # it is not a leaf note
        else:
            # return BranchNode(....) FILL THIS IN
            # pass
            svar = x[:, best_variable]
            is_left = svar < best_split_point
            leftx = x[is_left]
            rightx = x[~is_left]
            lefty = y[is_left]
            righty = y[~is_left]
            # recursively build the tree
            return BranchNode(self.fit_internal(leftx, lefty, current_depth+1),
                              self.fit_internal(
                                  rightx, righty, current_depth+1),
                              best_variable,
                              best_split_point)

    def depth(self):
        return self.root.depth()
```

```python
# NOTE: I modified generate_test_data by adding larger noise in final step
# see second code block where this function is defined for more detail
x_train,y_train=generate_test_data(50000)
x_test,y_test=generate_test_data(50000)
```

Let minimum points number in leaf be 100. We build regression tree model with different max depth and see how it influence in-sample and out-of-sample R square

```python
depths=range(1,20)
in_sample_r2=[]

for depth in depths:
    model=RegressionTree(depth,100)
    model.fit(x_train,y_train)
    in_sample_r2.append(r2_score(y_train,model.predict(x_train)))
```

```python
out_of_sample_r2=[]

for depth in depths:
    model=RegressionTree(depth,100)
    model.fit(x_train,y_train)
    out_of_sample_r2.append(r2_score(y_test,model.predict(x_test)))
```

```python
plt.figure(figsize=(14,7))
plt.plot(depths,in_sample_r2,"bo--",label="in-sample r2")
plt.plot(depths,out_of_sample_r2,"ro--",label="out-of-sample r2")
plt.xlabel("depth")
plt.ylabel("R2")
plt.legend()
plt.show()
```

![cart1](/assets/ml/cart1.png)

When increase max-depth, we grid X-pace into more sub-area hence we can fit train data better. So when we increase max-depth, the in-sample R2 always increase. But this will cause over-fitting problem, when max-depth is bigger than some optimal value, the out-of-sample R2 will decrease. This is a typical over-fitting phenomenon.

In test sense, the best max-depth is 7.

```python
class RandomForest:
    def __init__(self, max_tree_num, max_depth, min_points_in_leaf, feature_portion):
        self.max_tree_num = max_tree_num
        self.max_depth = max_depth
        self.min_points_in_leaf = min_points_in_leaf
        self.feature_portion = feature_portion

    def Bootstrap(self, x, y):
        length = len(y)
        # sample with replacement
        choice = np.random.choice(length, length)
        return x[choice], y[choice]

    def fit(self, x, y):
        self.fitted = True
        trees = []
        # build max_tree_num regression trees and fit
        for i in range(self.max_tree_num):
            x_, y_ = self.Bootstrap(x, y)
            tree = RegressionTree(
                self.max_depth, self.min_points_in_leaf, self.feature_portion)
            tree.fit(x_, y_)
            trees.append(tree)
        self.trees = np.array(trees)

    def predict(self, x, tree_num):
        assert self.fitted
        if tree_num > self.max_tree_num:
            assert False
        
        # random choose tree_num trees in fitted trees with replacement
        trees_idx = np.random.choice(self.max_tree_num, tree_num)
        trees = self.trees[trees_idx]
        
        # all prediction from chosed trees
        predicts = []
        for tree in trees:
            predicts.append(tree.predict(x))

        # return average prediction as forest's prediction
        return np.mean(predicts, axis=0), predicts
```

```python
# create a forest with 100 trees. max_depth = 7 from previous result
# min_leaf_node = 100 and feature_portion = 1/3
model = RandomForest(100, 7, 100, 1/3)

# fit data
model.fit(x_train, y_train)
```

For each T, generate 10 forests and calculate their median and average R2.

```python
Ts = [1, 2, 5, 10, 20, 30, 50, 75, 100]

averageR2 = []
medianR2 = []
correlations = []
for T in Ts:
    R2 = []
    # record all trees prediction to calculate pairwise correlation
    all_predicts = None
    # generate 10 forests
    for i in range(10):
        y_predict,tree_predicts = model.predict(x_test, T)
        R2.append(r2_score(y_test, y_predict))
        
        # record all predictions from each tree
        if all_predicts == None:
            all_predicts = tree_predicts
        else:
            all_predicts = all_predicts+tree_predicts

    averageR2.append(np.mean(R2))
    medianR2.append(np.median(R2))
    
    # calculate pairwise correlations
    # warning: very time consuming
    all_correlations = []
    for i in range(len(all_predicts)-1):
        for j in range(i+1, len(all_predicts)):
            all_correlations.append(np.corrcoef(
                all_predicts[i], all_predicts[j])[1][0])
    correlations.append(np.mean(all_correlations))
```

```python
plt.figure(figsize=(14,7))
plt.plot(Ts,averageR2,"bo--",label="average r2")
plt.plot(Ts,medianR2,"ro--",label="median r2")
plt.xlabel("T")
plt.ylabel("R2")
plt.legend()
plt.show()
```

![cart2](/assets/ml/cart2.png)

```python
plt.figure(figsize=(14,7))
plt.plot(Ts,correlations,"bo--",label="pairwise correlation")
plt.xlabel("T")
plt.ylabel("correlation")
plt.legend()
plt.show()
```

![cart3](/assets/ml/cart3.png)

```python
# create a forest with 100 trees. max_depth = 7 from previous result
# min_leaf_node = 100 and feature_portion = 1
model = RandomForest(100, 7, 100, 1)

# fit data
model.fit(x_train, y_train)
```

```python
Ts = [1, 2, 5, 10, 20, 30, 50, 75, 100]

averageR2 = []
medianR2 = []
correlations = []
for T in Ts:
    R2 = []
    # record all trees prediction to calculate pairwise correlation
    all_predicts = None
    # generate 10 forests
    for i in range(10):
        y_predict,tree_predicts = model.predict(x_test, T)
        R2.append(r2_score(y_test, y_predict))
        
        # record all predictions from each tree
        if all_predicts == None:
            all_predicts = tree_predicts
        else:
            all_predicts = all_predicts+tree_predicts

    averageR2.append(np.mean(R2))
    medianR2.append(np.median(R2))
    
    # calculate pairwise correlations
    # warning: very time consuming
    all_correlations = []
    for i in range(len(all_predicts)-1):
        for j in range(i+1, len(all_predicts)):
            all_correlations.append(np.corrcoef(
                all_predicts[i], all_predicts[j])[1][0])
    correlations.append(np.mean(all_correlations))
```

```python
plt.figure(figsize=(14,7))
plt.plot(Ts,averageR2,"bo--",label="average r2")
plt.plot(Ts,medianR2,"ro--",label="median r2")
plt.xlabel("T")
plt.ylabel("R2")
plt.legend()
plt.show()
```

![cart4](/assets/ml/cart4.png)

```python
plt.figure(figsize=(14,7))
plt.plot(Ts,correlations,"bo--",label="pairwise correlation")
plt.xlabel("T")
plt.ylabel("correlation")
plt.legend()
plt.show()
```

![cart5](/assets/ml/cart5.png)

It is higher than the correlation when set portion of feature to 1/3, which is consistent with what we expected.
