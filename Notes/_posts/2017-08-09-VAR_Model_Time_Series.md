---
layout: post
title: Vector Autoregression Overview and Proposals
---
## Introduction
Often we try to analyze huge amounts of data to find useful information or to predict future events. One of the most important types of dataset is <a href="https://en.wikipedia.org/wiki/Time_series">time series</a>. Time series represent a series of data points indexed in time order.
There are plenty of models to analyze this kind of series; one of those is the Vector Autoregression Model.

The <a href="https://en.wikipedia.org/wiki/Vector_autoregression">Vector Autoregression Model</a>, better known as *VAR*, is a model for time series that has been widely used in econometrics. The main idea of this model is that the value of a variable at a time point depends linearly on the value of different variables at previous instants of time. For example the number of births in a given month might be predicted from the value of the fertile population 9 months earlier.

However, these kinds of model have an important computational cost. Fortunately, as in many other methods of machine learning now we have the required computing capacity to apply them to huge datasets. Furthermore, we can apply it not only to econometrics but also to different fields like health or weather, or simply to any problem which works with time series.

In this article I will explain the fundamentals of VAR,  how to build and evaluate this model, an approach to find VAR models from a given data and parameters; and a proposal to take this method further using a guided search to find the best configuration for the model.<!--more-->

<br/>
## Overview
VAR is actually a variation of <a href="https://en.wikipedia.org/wiki/Autoregressive_model">autoregressive models</a> (AR) where we extend the autoregressive scheme to multiple variables with linear dependencies between them. For that reason, for this explanation we will start from a univariate AR and then we will extend it to multiple variables.


### Univariate
If we have a variable *x*, we can try to express the value of the variable at an instant *j* as a linear combination of the values of the variable at *i* previous time points. Furthermore, we add a constant *c* to fit better the data.

$$
x^{(j)}\approx x^{(j-1)}a_1+\ldots + x^{(j-i)}a_i + c
$$

Ok, this is how the model should look like but, how can we find the *a's* and the *c* that define our model? In this case, we can design an equation system that should be solvable using <a href="https://en.wikipedia.org/wiki/Linear_least_squares_(mathematics)">linear least squares</a>, *LLS*. If we have the values of *x* at *t* time points we can create the following least squares problem.

$$
\left(\begin{array}{ccc}
x^{(i)} \\
\vdots \\
x^{(t)}
\end{array}\right)
=
\left(\begin{array}{ccc}
x^{(i-1)} & \ldots  & x^{(0)} & 1\\
\vdots &  & \vdots &\vdots \\
x^{(t-1)} &  \ldots & x^{(t-i)} & 1
\end{array}\right)
\left(\begin{array}{ccc}
a_1 \\
\vdots \\
a_i \\
c
\end{array}\right)
$$

The solution of this system is the variables that we need to build  the autoregressive model.


## Multivariate
Once we have our model with one variable we can consider more variables; because it's much more interesting to consider that the value of a variable will depend not only on its previous value but also on other variables' values.


Therefore now instead of a variable *x* we will have a vector *y* with *d* variables. So our data can be represented as a matrix.

$$
\left(
\begin{array}{ccc}
y^{(1)} \\
\vdots  \\
y^{(t)}
\end{array}
\right)
=
\left(
\begin{array}{ccc}
y^{(1)}_1 & \ldots & y^{(1)}_d \\
\vdots & &\vdots \\
y^{(t)}_1 & \ldots & y^{(t)}_d
\end{array}
\right)
$$


And if the value of each variable depends on the value of other variables we will need a matrix *A<sub>i</sub>* instead of a variable *a<sub>i</sub>* (the same for *c*). After these changes our expression will look this way:

$$
y^{(j)}\approx y^{(j-1)}A_1+\ldots + y^{(j-i)}A_i + C
$$

As you can see the expression hasn't changed much. We are basically replacing variables by matrices. So the LLS system will look like this. (Notice that vectors and matrices are not expanded).

$$
\left(\begin{array}{ccc}
y^{(i)} \\
\vdots \\
y^{(t)}
\end{array}\right)
=
\left(\begin{array}{ccc}
y^{(i-1)} & \ldots  & y^{(0)} & 1\\
\vdots &  & \vdots & \vdots \\
y^{(t-1)} &  \ldots & y^{(t-i)} & 1
\end{array}\right)
\left(\begin{array}{ccc}
A_1 \\
\vdots \\
A_i \\
C
\end{array}\right)
$$


### Independent Variables
Let's take this further. Most of the time is too ambitious to assume that all the variables can be modeled using this method. Nevertheless, it will be more common to model a few variables taking advantage from all the variables that we have.

For example, if we have information about temperature and electricity consumption, maybe we can predict the latter from its own value and the value of the temperature. There's no point in trying to predict the temperature from the electricity consumption (well, it doesn't make enough sense).

Hence we must consider two vectors of variables:

 - The variables that depend on other variables: *y*
 - The variables that are independent: *z*

Once again, after considering these changes our model would look like this:

$$
y^{(j)}\approx y^{(j-1)}A_1+\ldots +
y^{(j-i)}A_i+z^{(j-1)}B_1+\ldots +
z^{(j-k)}B_k+C
$$

Because there are variables that we don't want to predict our LLS system will look a bit different, but only on the right side of the equality.

$$
\left(\begin{array}{ccc}
y^{(i)} \\
\vdots \\
y^{(t)}
\end{array}\right)
= 
\left(\begin{array}{ccc}
y^{(i-1)} & \ldots  & y^{(0)} & z^{(i-1)} & \ldots  & z^{(i-k)} & 1\\
\vdots &  & \vdots & \vdots & & \vdots & \vdots \\
y^{(t-1)} &  \ldots & y^{(t-i)} & z^{(t-1)} & \ldots  & z^{(t-k)} & 1
\end{array}\right)
\left(\begin{array}{ccc}
A_1 \\
\vdots \\
A_i \\
B_1 \\
\vdots \\
B_k \\
C
\end{array}\right)
$$

<br/>
## Evaluating The Model
Once we have a model for our data it's important to analyze how we can evaluate its quality. 

## Residuals
The first option is to use the residuals obtained from the LLS solution, which are basically the squared difference between the values predicted by our model and the real values. However, this option is, in my opinion, too simple and doesn't really tell us how accurate our model is when using VAR models.

On the one hand, we shouldn't use simply the sum of all the residuals because they are proportional to the amount of data that we are using to create the model. For that reason, I suggest dividing the residual by the number of time instants multiplied by the number of dependent variables.

But that is not the only problem. Most of the time, the data is not normalized, so if I tell you that the residual of my model is 2.3 you can't really know how good my model is. If the dataset I'm using have values around 15,000 probably a 2.3 residual per element is quite good. However, if I'm measuring values around 0.8, a 2.3 residual is huge. Thus it would be interesting to obtain a relative residual. For example, computing the mean of the values of the variable and then expressing the residual as a percentage with respect to the mean. 

Nevertheless, the convenience of relative residuals depends on the case. If we work with positive and negative values the relative residual might be incoherent. For example if we have data about temperature, with positive and negative values where the mean is 0. That would produce very big relative residuals.

Finally, we have to consider how we understand the residual when working with multiple variables. If we obtain relative residuals we can sum the residual of the different variables. However, if we think that fitting a certain variable is more important than fitting the others we should consider evaluating the residuals separately.

<br/>
## Training
When we've talked about residuals we have always considered that we are using all the information that we have to design the model. However, if we use all the data our algorithm may fall into <a href="https://en.wikipedia.org/wiki/Overfitting">overfitting</a> and obtain good results just for the data that we have but not for any other data with the same behavior. Therefore, we are not evaluating the quality of our model on forecasting.

For that reason, we could consider the possibility of not using all the information available to create the model, just a part of it. That would let us to use the rest of the data to evaluate how accurate our model is on forecasting by comparing the predicted values against the real values.

However, notice that not using all the data for training has some drawbacks. The main one is that we might need all the available information for the model. Just imagine that you have information about the temperature at each hour, these values usually follow a pattern where the values in a day are similar to the values the day before. Thus if we just use less than 24 hours for training it would be very difficult to find a good model. For that reason, it is recommended to have some understanding about what does the dataset represents to choose the amount of data that is going to be used for training the model.

<br/>
## Finding The Model
Historically, the task of choosing the right variables and the number of time points that variables depend on has been done by hand by experts based on their experience.

However, if the task of choosing these parameters were automatic we could apply the model to different datasets without even knowing the meaning of their data. In the best case we would find the best model configuration for that time series, and in the worst case we would find out that our data series cannot be properly represented with a vector autoregressive model.

Nevertheless, choosing the best parameters is a <a href="https://en.wikipedia.org/wiki/Combinatorial_optimization">combinatorial optimization</a> problem which requires <a href="https://en.wikipedia.org/wiki/Time_complexity#Exponential_time">exponential time</a> in the worst case. Thus even if our method is damn fast we won't be able to check all the possibilities in most problems' instances. Fortunately we expect that our understanding of the VAR model will allow us to develop an algorithm to find the optimal configuration in <a href="https://en.wikipedia.org/wiki/Time_complexity#Polynomial_time">polynomial time</a>.

The process of finding the best model can be divided in the following two blocks.

<br/>
### Variables Classification
As I mentioned previously, it's important to identify which variables can be predicted with the available data to avoid creating inconsistent models. Because the number of combinations is exponential we can't check all the possibilities. We could apply a well-studied generic <a href="https://en.wikipedia.org/wiki/Metaheuristic">metaheuristic</a> for this task, although it might not be necessary.

Our experience with this model has teach us that even if all the variables can be perfectly predicted, the best residual is obtained when taking only one variable as dependent and the rest as independent. Its residual is a bit lower than the one obtained considering all the variables as dependent. That's basically because there is less data to be fitted. Therefore we must promote models with more dependent variables.


Furthermore, we have also noticed that models which take independent variables as dependent have a much higher residual than those that don't make this mistake. Hence we can find a residual threshold that tells us if a model is taking an independent variables as dependent wrongly. This can let us use a guided search to classify the variables.

Based on what I've just mentioned, we can test a configuration for each variable, where that variable is considered dependent and the rest independent. If the residual is under the threshold the variable is indeed dependent otherwise the variable is independent.

And because we said that we are looking for the model that works properly with as many dependent variables as possible we would take all the variables that we identified individually as dependent and put them all together. In most cases that should be the best model based on the mentioned heuristics.

<br/>
### Time Dependency
The other parameters that we have to choose are the number of time instants that the variables depend on, named *i* and *k*.

First of all we should notice that if we use a good LLS solver we will notice that models with higher time dependency always obtain better residuals. That is basically because we are adding more variables to fit the model. And in the worst case where the real model only depends on a few time points, higher time dependency will still fit well because the algorithm will set most of the dependencies to 0.

However, if we want to understand the meaning of the data we cannot simply choose a huge time dependency; we should choose the one more reasonable. For that reason, we should avoid using higher time dependencies when the model already fits well.

An initial approach to find the best time dependency is considering the number of previous time instants that the variables depend on as a function. We should stop increasing the time dependency once the model starts to improve slower.

<br/>
## Computing
I'm a computer scientist, so all these equation systems are really nice but, how are we going to solve them?

Well, the best part of the model is that is basically linear algebra. Therefore we can solve it using high performance linear algebra libraries. I would suggest any library based on <a href="https://en.wikipedia.org/wiki/Basic_Linear_Algebra_Subprograms">BLAS</a> and <a href="https://en.wikipedia.org/wiki/LAPACK">LAPACK</a>. If we make a reasonable good implementation with these sorts of libraries we will be able to solve very big problems' instances.

<br/>
### Matrix Decomposition
Because the computational demanding part of this problem is solving the LLS system, we should focus on optimizing it. One of the best approaches is decomposing the matrix before solving the system, because it would simplify our matrices allowing the LLS solver to run faster. However <a href="https://en.wikipedia.org/wiki/Matrix_decomposition">matrix decomposition</a> is not the topic of this article, so I will just pick up the one that I consider the best option. <a href="https://en.wikipedia.org/wiki/QR_decomposition">QR</a> decomposition.

<br/>
### Parallelism
To obtain the highest performance, it's crucial to use parallel algorithms to solve the least squares system and the decomposition as fast as possible. Fortunately, most of the libraries that implement these functions know pretty well the importance of supporting parallelism.

But that's not all. If we want to check different parameters for the model it would be very interesting to run those attempts in parallel, resulting in two levels of parallelism.

<br/>
## Code
Fortunately, we have already developed an initial implementation to test most of these promising ideas. 

Firstly, we've created a time series datasets generator to test our algorithm. This generator allows us to create ideal time series that are based on a VAR model. 

Secondly, using LAPACK we've implemented the algorithm to find the VAR model from a given parameters. To measure the quality we are considering the residuals and using the option of considering a part of the dataset to train the model and the rest to evaluate it. In addition initial work has been done developing and exhaustive search to find optimal parameters.

<a href="https://github.com/fylux/tsf_var">Here</a> is the project repository.

<br/>
## Further Research
Many ways open to improve this method. On the one hand, the ways related with making the model more powerful and flexible. For example, overcoming the limitations of a linear model using nonlinear relations or considering VAR models can vary over time.

On the other hand, the performance could be improved adding constraints to the LLS solver or taking advantage of the matrix structure which is similar to a <a href="https://en.wikipedia.org/wiki/Toeplitz_matrix">Toeplitz</a> matrix.

Finally, it is very important to keep improving the methods to measure the quality of the models obtained.

<br/>
## Conclusion
In this article, we've presented some interesting ideas for being able to use VAR as a machine learning technique by using high performance linear algebra libraries with matrix decomposition to increase performance combined with finding the best configuration for the model automatically. However, this is only an initial approach and a lot of research has to be made.




