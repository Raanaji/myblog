---
template: BlogPost
path: /neural
mockup: 
thumbnail:
date: 2019-12-01
name: Machine Learning Series - Implementing RidgeCV and LassoCV
title: In this post of Machine Learning Series we will implement RidgeCV and LassoCV on selected data sets 
category: Machine Learning
description: 'Using the sklearn library in python we will implement some machine learning algorithms to fit data sets.'
tags:
  - Machine Learning 
  - RidgeCV
  - LassoCV
  - Data Engineering
---

## Implementation

We will implement a simple 1 hidden layer neural network in PyTorch using tanh as our activation function and a hidden layer of 10 neurons. We use this to fit the 2 data sets from Problem 3. We will use the following skeleton

- Using the same test data sets as before, we  calculate the in-sample and out-of-sample error for the network.
- We then compare this to the error from the RidgeCV and LassoCV (refer previous post) 
- For the 1st data set, roughly we calculate the epochs it took to converge
- For the 1st data set, roughly we calculate the epochs it took to converge if we change the activation function to ReLu

```python
def NN_model(F,H,data_set,test_set,epoch=100,is_ReLu=False):
    # model initialize
    out_size=1
    torch.manual_seed(1)
    
    x=torch.tensor(data_set[0],dtype=torch.float32)
    y=torch.tensor(data_set[1][:,np.newaxis],dtype=torch.float32)

    model=torch.nn.Sequential(
        torch.nn.Linear(F,H),
        torch.nn.Tanh() if not is_ReLu else torch.nn.ReLU(),
        torch.nn.Linear(H,out_size),
        )

    loss_fn=torch.nn.MSELoss()
    
    learning_rate=5e-2
    optimizer=torch.optim.Adam(model.parameters(),lr=learning_rate)
    
    # training  
    loss_history=[]

    for t in range(epoch):
        y_pred=model(x)

        loss=loss_fn(y_pred,y)
        #print(t,loss.item())
        loss_history.append(loss.item())

        optimizer.zero_grad()

        loss.backward()

        optimizer.step()

    plt.figure(figsize=(14,7))
    plt.plot(range(len(loss_history)),loss_history)
    plt.xlabel("epoch")
    plt.ylabel("loss")
    plt.show()
    
    # test
    x_test=torch.tensor(test_set[0],dtype=torch.float32)
    y_test=torch.tensor(test_set[1][:,np.newaxis],dtype=torch.float32)

    err_in=loss_fn(model(x),y).item()
    err_out=loss_fn(model(x_test),y_test).item()

    print("in-sample error: ",err_in,"\n")
    print("out-of-sample error: ",err_out)
    
    return (loss_history,err_in,err_out)
```

#### Data Set I with parameter I

```python
loss11_history,err11_nn_in,err11_nn_out=NN_model(5,10,data_set11,test_set11)
```

![5](/assets/ml/5.png)

```
in-sample error:  1.0389487743377686 

out-of-sample error:  1.062627911567688
```

#### Data Set I with parameter II

```python
loss12_history,err12_nn_in,err12_nn_out=NN_model(20,40,data_set12,test_set12,epoch=1000)
```

![6](/assets/ml/6.png)

```
in-sample error:  7490.26611328125 

out-of-sample error:  12400.5537109375
```

#### Data Set II with parameter I

```python
loss21_history,err21_nn_in,err21_nn_out=NN_model(5,10,data_set21,test_set21)
```

![7](/assets/ml/7.png)

```
in-sample error:  0.9991199970245361 

out-of-sample error:  1.016243577003479
```

#### Data Set II with parameter II

```python
loss22_history,err22_nn_in,err22_nn_out=NN_model(20,40,data_set22,test_set22,epoch=1000)
```

![8](/assets/ml/8.png)

```
in-sample error:  7474.736328125 

out-of-sample error:  12461.8173828125
```

We compare Ridge and Lasso results under best cross-validation alpha with neutral network.

```python
print("data set I with parameter I:")
print("  Lasso:")
print("    train: ",err11_lasso_in)
print("    test:  ",err11_lasso_out)
print("  Ridge:")
print("    train: ",err11_ridge_in)
print("    test:  ",err11_ridge_out)
print("  Neutral Network:")
print("    train: ",err11_nn_in)
print("    test:  ",err11_nn_out)

print("\ndata set I with parameter II:")
print("  Lasso:")
print("    train: ",err12_lasso_in)
print("    test:  ",err12_lasso_out)
print("  Ridge:")
print("    train: ",err12_ridge_in)
print("    test:  ",err12_ridge_out)
print("  Neutral Network:")
print("    train: ",err12_nn_in)
print("    test:  ",err12_nn_out)

print("\ndata set II with parameter I:")
print("  Lasso:")
print("    train: ",err21_lasso_in)
print("    test:  ",err21_lasso_out)
print("  Ridge:")
print("    train: ",err21_ridge_in)
print("    test:  ",err21_ridge_out)
print("  Neutral Network:")
print("    train: ",err21_nn_in)
print("    test:  ",err21_nn_out)

print("\ndata set II with parameter II:")
print("  Lasso:")
print("    train: ",err22_lasso_in)
print("    test:  ",err22_lasso_out)
print("  Ridge:")
print("    train: ",err22_ridge_in)
print("    test:  ",err22_ridge_out)
print("  Neutral Network:")
print("    train: ",err22_nn_in)
print("    test:  ",err22_nn_out)
```

```
data set I with parameter I:
  Lasso:
    train:  0.9976911151951062
    test:   1.0100305858523435
  Ridge:
    train:  0.997691116466355
    test:   1.010030288962445
  Neutral Network:
    train:  1.0389487743377686
    test:   1.062627911567688

data set I with parameter II:
  Lasso:
    train:  9946.885342292077
    test:   10167.840989737377
  Ridge:
    train:  9944.33367959123
    test:   10164.693756090375
  Neutral Network:
    train:  7490.26611328125
    test:   12400.5537109375

data set II with parameter I:
  Lasso:
    train:  0.9976954062887348
    test:   1.0100769985242344
  Ridge:
    train:  0.9976911476552519
    test:   1.010033461678113
  Neutral Network:
    train:  0.9991199970245361
    test:   1.016243577003479

data set II with parameter II:
  Lasso:
    train:  9955.339279508502
    test:   10168.207859223168
  Ridge:
    train:  9955.33546436624
    test:   10168.20632008298
  Neutral Network:
    train:  7474.736328125
    test:   12461.8173828125
```

For two data sets with parameter I, with only 100 epochs, the NN's performance seems a little worse than Ridge and Lasso. This is due to the limitation of epoch number. With more epochs, the error from NN should be smaller or at least equal than linear models.

For two data sets with parameter II, even run 1000 epochs, the error is relative large. This is due to the big sigma of data generating parameters. And NN's in-sample error is smaller than linear model while the out-of-sample is larger. This reflects NN is prone to over-fitting.



Because the parameter II makes the model very hard to converge. We can see the in-sample error was still 7000+ even after 1000 epochs. So here we only discuss data set I with parameter I.

Zoom in the figure of loss history of data set I with parameter I during the training.

```python
plt.figure(figsize=(14,7))
plt.plot(range(len(loss11_history))[30:],loss11_history[30:])
plt.xlabel("epoch")
plt.ylabel("loss")
plt.show()
```

![9](/assets/ml/9.png)

It seems that after 80 epochs, it converges. 

In order to decide precisely after how many epochs, the model converges, we need a quantitative criterion. Here we define if the improvement of mse in one epoch is smaller than 1e-3, than we say model converges.

```python
for i in range(1,100):
    if loss11_history[i-1]>loss11_history[i] and loss11_history[i-1]-loss11_history[i]<1e-3:
        print("Tanh activation converges after",i,"epochs")
        break
```

```
Tanh activation converges after 76 epochs
```

Here we change activation function to ReLu

```python
loss11_relu_history,err11_nn_relu_in,err11_nn_relu_out=NN_model(5,10,data_set11,test_set11,is_ReLu=True)
```

![10](/assets/ml/10.png)

```
in-sample error:  1.0168170928955078 

out-of-sample error:  1.0350059270858765
```

Zoom in the last figure

```python
plt.figure(figsize=(14,7))
plt.plot(range(len(loss11_relu_history))[40:],loss11_relu_history[40:])
plt.xlabel("epoch")
plt.ylabel("loss")
plt.show()
```

![11](/assets/ml/11.png)

From in-sample and out-of-sample error, we see ReLU converges faster than Tanh. Apply same quantitaive criterion for convergence, if the improvement of mse in one epoch is smaller than 1e-3, than we say model converges.

```python
for i in range(1,100):
    if loss11_relu_history[i-1]>loss11_relu_history[i] and loss11_relu_history[i-1]-loss11_relu_history[i]<1e-3:
        print("ReLU activation converges after",i,"epochs")
        break
```

```
ReLU activation converges after 70 epochs
```

We can conclude in this data set, ReLU activation converges faster than Tanh activation.
