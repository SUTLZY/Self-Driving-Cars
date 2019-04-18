

## Module 1: Least Squares

> The method of least squares, developed by Carl Friedrich Gauss in 1795, is a well known technique for estimating parameter values from data. This module provides a review of least squares, for the cases of unweighted and weighted observations. There is a deep connection between least squares and maximum likelihood estimators (when the observations are considered to be Gaussian random variables) and this connection is established and explained. Finally, the module develops a technique to transform the traditional 'batch' least squares estimator to a recursive form, suitable for online, real-time estimation applications.

### 学习目标

- Understand the squared error optimization criterion and its use
- Explain how least squares is employed in parameter estimation problems
- Apply the unweighted and weighted least squares methods to parameter estimation
- Apply a recursive version of least squares to update parameter estimates as new measurements arrive
- Explain how Jacobian matrices are used

---

### Lesson 1 (Part 1): Squared Error Criterion and the Method of Least Squares

> 平方误差准则和最小二乘法

Hello, and welcome to course two of the self-driving car specialization. In this course, we'll cover state estimation with a focus on localization. Let's first begin by defining a few key terms . 

Localization is the method by which we determine the position and orientation of a vehicle within the world. As you can probably imagine, accurate localization is a key component of any self-driving car software stack. If we want to drive autonomously, we certainly need to know where we are. 

To accomplish this, we can use state estimation. This is the process of computing a physical quantity like position from a set of measurements. Since any real-world measurements will be imprecise, we will develop methods that try to find the best or optimal value given some assumptions about our sensors and the external world. 

Related to state estimation is the idea of **parameter estimation**. Unlike a state, which we will define as a physical quantity that changes over time, a parameter is constant over time. Position and orientation are states of a moving vehicle, while the resistance of a particular resistor in the electrical sub-system of a vehicle would be a parameter. 

---

#### 1. Overview

> - The history of the method of least squares
> - Ordinary and weighted least squares
> - Recursive least squares
> - Maximum likelihood and the method of least squares

In the first module or week one of the course, we will cover a common technique in state estimation, the method of least squares. By the end of this week, you'll know a little bit about the history of least squares and you'll learn about the method of ordinary least squares and its cousin, the method of weighted least squares. Then, we'll cover the method of recursive least squares and finally, discuss the link between least squares and the maximum likelihood estimation technique. 

In the first lesson of this module, we'll introduce the method of least squares and something called the squared error criterion. By the end of the video, you'll be able to describe how the method of least squares was first used by Carl Friedrich Gauss in the discovery of the planetoid Ceres. Describe the least error criterion and how it's used in estimating the best parameters. Derive the necessary normal equations that we'll need to solve to use the method. Let's begin. 

---

#### 2. History

The method of least squares dates to the late 18th century well before anyone had considered the concept of automobiles. On January 1st, 1801, an Italian priest and astronomer Giuseppe Piazzi discovered a new celestial object in the night sky. The asteroid or planetoid now called Ceres. You can see it here next to the moon and to the earth. 

![1555593389925](assets/1555593389925.png)

Piazzi made 24 telescope observations of this new object over 40 days before it was lost in the glare of the sun. Since Ceres is only about 900 kilometers in diameter, finding it again it was extremely challenging. This meant that other astronomers could not confirm Piazzi's discovery. To help locate Ceres again, Carl Friedrich Gauss who has been called the prince of mathematicians for his prodigious contributions to many different fields, used a method of least squares to accurately estimate Ceres orbital parameters based on Piazza's published measurements. 

![1555593417053](assets/1555593417053.png)

With Gauss's calculations in hand, astronomers were successfully able to rediscover Ceres nearly a year after Piazzi had made his first observations.  Although he published the method in 1809, Gauss claimed that he developed least squares in 1795, predating Lesion's work. 

Gauss summarized the approach as follows:

*"The most probable value of the unknown quantities will be that in which the sum of the squares of the differences between the actually observed and the computed values multiplied by numbers that measure the degree of precision is a minimum."*

---

#### 3. Example

To illustrate how this works, let's take a simple example. Suppose you are trying to measure the value in ohms of a simple resistor within the drive system of an autonomous vehicle. 

![1555593522437](assets/1555593522437.png)

To do this, you grab a relatively inexpensive multimeter that's lying around your lab.  Now, let's say you collect these four separate measurements, sequentially. 

![1555593565530](assets/1555593565530.png)

If you've studied electrical circuits before, you'll probably recall that the type of carbon film resistors shown here is color-coded according to its rated resistance value. This resistor is rated at one kilo ohm. **However, due to a number of factors, the true resistance can vary from the rated value.** In this case, the resistor has a gold band which indicates that it can vary by as much as five percent. Furthermore, let's imagine that our multimeter is particularly poor and that the person making the measurements is not particularly careful.For now, it's sufficient to treat this noise as equivalent to a general error but we'll come back to ways we can interpret the noise from a probabilistic perspective later in the module. 

For each of the four measurements, we define a scalar noise term that is independent of the other noise terms. Statistically, we would say in this case that the noise is independent and identically distributed or IID, and we'll discuss this more later. Next, we define the error between each measurement and the actual value of our resistance x.

![1555593675488](assets/1555593675488.png)

But remember, we don't yet know what x is. To find x, we square these errors to arrive at an equation that is a function of our measurements and the unknown resistance that we're looking for. 

![1555592701365](assets/1555592701365.png)

With these errors defined, the method of least squares says that the resistance value we are looking for, that is the best estimate of x is one which minimizes the squared error criterion, also sometimes called the squared error cost function or loss function. 
$$
\hat{x}_{\mathrm{LS}}=\operatorname{argmin}_{x}\left(e_{1}^{2}+e_{2}^{2}+e_{3}^{2}+e_{4}^{2}\right)=\mathscr{L}_{\mathrm{LS}}(x)
$$
To minimize the squared error criterion, we'll rewrite our errors in matrix notation. 
$$
\mathbf{e}=\left[ \begin{array}{l}{e_{1}} \\ {e_{2}} \\ {e_{3}} \\ {e_{4}}\end{array}\right]=\mathbf{y}-\mathbf{H} x=\left[ \begin{array}{l}{y_{1}} \\ {y_{2}} \\ {y_{3}} \\ {y_{4}}\end{array}\right]-\left[ \begin{array}{l}{1} \\ {1} \\ {1} \\ {1}\end{array}\right] x
$$
This will be especially helpful when we have to deal with hundreds or even thousands of measurements. We'll define an error vector identified as bold e, that is a function of our observations stacked into a vector y and a matrix H called the Jacobian. 

Finally, our true resistance x. H has the dimensions m by n, where m is the number of measurements and n is the number of unknowns or parameters that we wish to estimate. In general, this will be a rectangular matrix which we can write down quite easily in this linear case. It will require some more mathematical effort to compute when we discuss non-linear estimation in an upcoming lesson. Note here that x is a single scaler. We'll see later that it can be a vector comprising multiple unknowns.

With these definitions in mind, we can convert our squared error criterion to vector notation as follows. 
$$
\begin{aligned} \mathscr{L}_{\mathrm{LS}}(x)=e_{1}^{2}+e_{2}^{2}+e_{3}^{2}+e_{4}^{2} &=\mathbf{e}^{T} \mathbf{e} \\ &=(\mathbf{y}-\mathbf{H} x)^{T}(\mathbf{y}-\mathbf{H} x) \\ &=\mathbf{y}^{T} \mathbf{y}-x^{T} \mathbf{y}-\mathbf{y}^{T} \mathbf{H} x+x^{T} \mathbf{H}^{T} \mathbf{H} x \end{aligned}
$$

---

#### 4. Minimizing the Squared Error Criterion

Expanding out the brackets, we arrive at this somewhat intimidating expression. 
$$
\mathscr{L}(x)=\mathbf{e}^{T} \mathbf{e}=\mathbf{y}^{T} \mathbf{y}-x^{T} \mathbf{H}^{T} \mathbf{y}-\mathbf{y}^{T} \mathbf{H} x+x^{T} \mathbf{H}^{T} \mathbf{H} x
$$
So, what now? Well, remember that we need to minimize the squared error with respect to our true resistance x. From calculus, we know that we can do this by taking a derivative of the error function with respect to the unknown x and setting the derivative to zero. Let's do exactly that. We'll have to use some standard matrix expressions. 
$$
\begin{array}{r}{\left.\frac{\partial \mathscr{L}}{\partial x}\right|_{x=\hat{x}}=-\mathbf{y}^{T} \mathbf{H}-\mathbf{y}^{T} \mathbf{H}+2 \hat{x}^{T} \mathbf{H}^{T} \mathbf{H}=0} \\ {-2 \mathbf{y}^{T} \mathbf{H}+2 \hat{x}^{T} \mathbf{H}^{T} \mathbf{H}=0}\end{array}
$$
Re-arranging, we arrive at what are called the normal equations, which can be written as a single matrix formula. 
$$
\hat{x}_{\mathrm{LS}}=\left(\mathbf{H}^{T} \mathbf{H}\right)^{-1} \mathbf{H}^{T} \mathbf{y}
$$
We can solve these to find x-hat, the resistance which minimizes our squared error criterion. This expression has a unique solution if and only if H transpose H is not singular. In other words, if the matrix has an inverse. If we have m measurements and n unknown parameters, H transpose H will be n by n. 
$$
\mathbf{H} \in \mathbb{R}^{m \times n} \quad \mathbf{H}^{T} \mathbf{H} \in \mathbb{R}^{n \times n}
$$
The matrix will be invertible if and only if m is greater than or equal to n. So, we need at least as many measurements as unknowns in order to derive the least squares solution. This will usually not be a problem. In fact, we'll often face the challenge of dealing with too many measurements. But nevertheless, you should keep this limitation in mind when working with the formula. 

Coming back to our particular resistance problem, let's fill out our variables. 

![1555594363143](assets/1555594363143.png)

Once we have these quantities, it's just a matter of plug and chug. The expression is quite straightforward to code up as you'll see in your module assignment. 

Note here that the expression simplifies to the arithmetic mean of our four measurements. 

![1555594513788](assets/1555594513788.png)

Perhaps this is something you thought of doing all along. Now, we have another justification for using the arithmetic mean, it minimizes the simple least squares criterion. Finally, we should note two important assumptions that we've made. 

---

#### 5. Method of Least Squares | Assumptions

First, we've assumed that our measurement model is linear. This is a very important assumption that is often broken in complex systems. We'll discuss non-linear measurement models in later lessons. Second, we've assumed that all of our measurements have an equal weight in our error equation. To put this another way, we've assumed that we care about each of our measurements equally.

To summarize, in this video, you've learned that the method of least squares was pioneered by Carl Friedrich Gauss who used it to accurately predict the orbit of a new planet like object called Ceres. You saw how we can minimize the least squares criterion to solve for parameter values of interest. We noted that the method of ordinary least squares assumes a linear measurement model and can't handle measurements of unequal importance. In the next video, we'll extend the method of least squares to the method of weighted least squares, which can account for measurements of varying importance.

---

### Lesson 1 (Part 2): Squared Error Criterion and the Method of Least Squares

> - Derive the weighted least squares criterion given varying measurement noise variance
> - Compare weighted least squares to ordinary least squares

In the last video, we saw how we could use the method of least squares to solve for a more correct value of resistance given a set of noisy measurements. In this video, we'll ask the question, what can we do if we suspect that some of our measurements are of better quality than others? By the end of this video, you'll be able to derive and minimize the weighted least squares criterion that will let us handle measurements of different quality and compare this new method to the method of regular or ordinary least squares that we discussed in the previous video. Let's begin. 

---

#### 1. Method of *Weighted* Least Squares

One reason we may want to trust certain measurements more than others is that they may come from a better sensor. For example, a number of our resistance measurements could have come from a much more expensive multi-meter than the others. Further, from now on, we'll also drop the assumption that we're only estimating one parameter and derive the more general normal equations. 

![1555595877734](assets/1555595877734.png)

This will allow us to formulate a way to estimate multiple parameters at one time. For example, if we wanted to estimate several resistance values at once. 

Let's begin by using the following general notation. We'll have a set of m measurements that are related to a set of n unknown parameters through a linear model.Recall that H is rj cobian matrix whose form and entries will depend on the particular problem at hand. 
$$
\left[ \begin{array}{c}{y_{1}} \\ {\vdots} \\ {y_{m}}\end{array}\right]=\mathbf{H} \left[ \begin{array}{c}{x_{1}} \\ {\vdots} \\ {x_{n}}\end{array}\right]+\left[ \begin{array}{c}{v_{1}} \\ {\vdots} \\ {v_{m}}\end{array}\right]\\ 
\\
y = Hx + v
$$
One way to interpret the ordinary method of least squares is to say that we are implicitly assuming that each noise term v_i is an independent random variable across measurements and has an equal variance or standard deviation if you prefer, IID as we mentioned in the previous video. 
$$
\mathbb{E}\left[v_{i}^{2}\right]=\sigma^{2} \quad(i=1, \ldots, m)\quad \quad    \mathbf{R}=\mathbb{E}\left[\mathbf{v} \mathbf{v}^{T}\right]=\left[ \begin{array}{cc}{\sigma^{2}} & {0} \\ {0} & {\sigma^{2}}\end{array}\right]
$$
If we instead assume that each noise term has a different variance, we can define our noise covariance as follows.
$$
\mathbb{E}\left[v_{i}^{2}\right]=\sigma_i^{2} \quad(i=1, \ldots, m)\quad \quad    \mathbf{R}=\mathbb{E}\left[\mathbf{v} \mathbf{v}^{T}\right]=\left[ \begin{array}{cc}{\sigma_1^{2}} & {0} \\ {0} & {\sigma_m^{2}}\end{array}\right]
$$
With this definition, we can define a weighted least squares criterion. 
$$
\begin{aligned} \mathscr{L}_{\mathrm{WLS}}(\mathbf{x}) &=\mathbf{e}^{T} \mathbf{R}^{-1} \mathbf{e} \\ &=\frac{e_{1}^{2}}{\sigma_{1}^{2}}+\frac{e_{2}^{2}}{\sigma_{2}^{2}}+\ldots+\frac{e_{m}^{2}}{\sigma_{m}^{2}} \end{aligned} \quad \quad where \quad \quad
\left[ \begin{array}{c}{e_{1}} \\ {\vdots} \\ {e_{m}}\end{array}\right]=\mathbf{e}=\left[ \begin{array}{c}{y_{1}} \\ {\vdots} \\ {y_{m}}\end{array}\right]-\mathbf{H} \left[ \begin{array}{c}{x_{1}} \\ {\vdots} \\ {x_{n}}\end{array}\right]
$$
By expanding this expression, we can see why we call this weighted least squares. Each squared error term is now weighted by the inverse of the variance associated with the corresponding measurement. **In other words, the lower the variance of the noise, the more strongly it's associated error term will be weighted in the loss function.** We care more about errors which come from low noise measurements since those should tell us a lot about the true values of our unknown parameters. 

---

#### 2. Re-deriving Regular Least Squares

Before we see how we can minimize this new weighted criterion, let's look at what happens if we set all of the noise standard deviations to the same value sigma. 
$$
\begin{aligned} \mathscr{L}_{\mathrm{WLS}}(\mathbf{x}) &=\frac{e_{1}^{2}}{\sigma^{2}}+\frac{e_{2}^{2}}{\sigma^{2}}+\ldots+\frac{e_{m}^{2}}{\sigma^{2}} \\ &=\frac{1}{\sigma^{2}}\left(e_{1}^{2}+\ldots+e_{m}^{2}\right) \end{aligned}
$$
In this case, we can factor out the variance in the denominator. Since the sigma squared term is constant it will not affect the minimization. This means that in the case of equal variances, the same parameters that minimize our weighted least squares criterion will also minimize our ordinary least squares criterion as we should expect. 
$$
\sigma_{1}^{2}=\sigma_{2}^{2}=\ldots \sigma_{m}^{2}=\sigma^{2} \rightarrow \quad \hat{\mathbf{x}}_{\mathrm{WLS}}=\hat{\mathbf{x}}_{\mathrm{LS}}=\operatorname{argmin}_{\mathbf{x}} \mathscr{L}_{\mathrm{LS}}(\mathbf{x})=\arg \min _{\mathbf{x}} \mathscr{L}_{\mathrm{WLS}}(\mathbf{x})
$$

---

#### 3. Minimizing the *Weighted* Least Squares Criterion

Returning to our weighted least squares criterion, we approach it's minimization the same way as before, we take a derivative. 
$$
\begin{aligned} \mathscr{L}_{\mathrm{WLS}}(\mathbf{x}) &=\mathbf{e}^{T} \mathbf{R}^{-1} \mathbf{e} \\ &=(\mathbf{y}-\mathbf{H} \mathbf{x})^{T} \mathbf{R}^{-1}(\mathbf{y}-\mathbf{H} \mathbf{x}) \end{aligned}
$$
In the general case where we have n unknown parameters in our bold vector x, this derivative will actually be a gradient. 
$$
\hat{\mathbf{x}}=\operatorname{argmin}_{\mathbf{x}} \mathscr{L}(\mathbf{x}) \quad \longrightarrow \quad\left.\frac{\partial \mathscr{L}}{\partial \mathbf{x}}\right|_{\mathbf{x}=\hat{\mathbf{x}}}=\mathbf{0}=-\mathbf{y}^{T} \mathbf{R}^{-1} \mathbf{H}+\hat{\mathbf{x}}^{T} \mathbf{H}^{T} \mathbf{R}^{-1} \mathbf{H}
$$
Setting the gradient of the zero vector, we then solve for our best or optimal parameter vector x hat. 
$$
\mathbf{H}^{T} \mathbf{R}^{-1} \mathbf{H} \hat{\mathbf{x}}_{\mathrm{WLS}}=\mathbf{H}^{T} \mathbf{R}^{-1} \mathbf{y}
$$
This leads to another set of normal equations this time called the weighted normal equations. 

---

#### 4. Method of Weighted Least Squares

Let's take a look at an example of how this method of weighted least squares works. We'll take the same data we collected before, but now assume that the last two measurements were actually taken with a multimeter that had a much smaller noise variance. 

![1555596933164](assets/1555596933164.png)

Be careful here, the numbers we list are standard deviations which is why they have the units of ohms. In order to use them in our formulation, we will need to square them to get the variances. 
$$
\mathbf{H}=\left[ \begin{array}{l}{1} \\ {1} \\ {1} \\ {1}\end{array}\right] \quad \mathbf{y}=\left[ \begin{array}{c}{1068} \\ {988} \\ {1002} \\ {996}\end{array}\right] \quad \mathbf{R}=\left[ \begin{array}{c}{\sigma_{1}^{2}} \\ {\sigma_{2}^{2}} \\ {\sigma_{3}^{2}} \\ {\sigma_{3}^{2}}\end{array}\right]=\left[ \begin{array}{c}{400} \\ {400} \\ {4}\end{array}\right]
$$
Defining our variables and then evaluating our weighted least squares solution, we can see that the final resistance value we get is much closer to what the more accurate multimeter measured as expected.Here's a quick summary of the methods of least squares and weighted least squares. 

![1555597136611](assets/1555597136611.png)

---

#### 5. Ordinary versus  | Weighted Least Squares

By using weighted least squares, we can vary the importance of each measurement to the final estimate. 

![1555597212598](assets/1555597212598.png)

It's important to be comfortable working with different measurement variances and also with measurements that are sometimes correlated. A self-driving car will have a number of different and complex sensors on board and we need to make sure that we model our error sources correctly. So, there you have it, weighted least squares. 

---

> - Measurements can come from sensors that have different noisy characteristics
> - Weighted least squares lets us weight each measurement according to noise variance

In this video, we discussed how certain measurements may come from sensors with better noise characteristics and should therefore be weighted more heavily in our least squares criterion. Using this intuition, we derived the weighted least squares criterion and the associated weighted normal equations that can be solved to yield the weighted least squares estimate of a set of constant parameters. In the next video, we'll look at modifying the method of least squares to work recursively, that is to compute an optimal estimate based on a stream of measurements without having to acquire the entire set beforehand. This will be very important when we look at state estimation, or the problem of estimating quantities that change continuously over time.