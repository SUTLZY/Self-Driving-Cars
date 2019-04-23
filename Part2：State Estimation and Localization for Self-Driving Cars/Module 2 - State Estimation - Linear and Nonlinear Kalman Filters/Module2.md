# Module 2 - State Estimation - Linear and Nonlinear Kalman Filters

Any engineer working on autonomous vehicles must understand the Kalman filter, first described in a paper by Rudolf Kalman in 1960. The filter has been recognized as one of the top 10 algorithms of the 20th century, is implemented in software that runs on your smartphone and on modern jet aircraft, and was crucial to enabling the Apollo spacecraft to reach the moon. This module derives the Kalman filter equations from a least squares perspective, for linear systems. 

The module also examines why the Kalman filter is the best linear unbiased estimator (that is, it is optimal in the linear case). The Kalman filter, as originally published, is a linear algorithm; however, all systems in practice are nonlinear to some degree. Shortly after the Kalman filter was developed, it was extended to nonlinear systems, resulting in an algorithm now called the ‘extended’ Kalman filter, or EKF. The EKF is the ‘bread and butter’ of state estimators, and should be in every engineer’s toolbox. This module explains how the EKF operates (i.e., through linearization) and discusses its relationship to the original Kalman filter. The module also provides an overview of the unscented Kalman filter, a more recently developed and very popular member of the Kalman filter family.

### 学习目标

- Describe the relationship between least squares and the linear Kalman filter
- Explain the (in)sensitivity of the Kalman filter to new data and the need for process noise
- Describe how the linear Kalman filter can be extended to nonlinear systems via linearization
- Describe how the EKF uses first-order linearization to turn a nonlinear problem into a linear one
- Contrast the linearization approach of the EKF with that of the UKF, and explain why the UKF is superior for many problems
- Assess the performance of the extended Kalman filter and its variants

---

## Lesson 1: The (Linear) Kalman Filter

### 1. Overview

Welcome back. In module two, we'll learn about one of the most famous algorithms in all of engineering; the Kalman filter. In today's world of advanced machine learning, the Kalman filter remains an important tool to fuse measurements from several sensors to estimate in real-time the state of a robotic system such as a self-driving car. In this module, we'll learn some of the history of the Kalman filter and introduce its basic linear formulation. We'll present why the Kalman filter is the best linear unbiased estimator and then extend the linear formulation to nonlinear systems through linearization. Discuss limitations of this linearization approach, and finally, present a modern alternative to linearization through the unscented transform. 

By the end of this lesson, you'll be able to 

- Describe the common filter as a state estimator that works in two stages：（1）prediction（2）correction. 

- Understand the difference between motion and measurement models

- Use the Kalman filter formulation in a simple 1D localization example. 

---

### 2. History

Let's begin with a bit of history. The Kalman filter algorithm was published in 1960 by Rudolf E. Kalman, a Hungarian born professor and engineer who was working at the Research Institute for Advanced Studies in Baltimore Maryland. Years later in 2009, American President Barack Obama awarded Kalman the prestigious National Medal of Science for his work on the Kalman filter and other contributions to the field of control engineering. 

![1555943834657](assets/1555943834657.png)

After its publication in 1960, the Kalman filter was adopted by NASA for use in the Apollo guidance computer. It was this ground-breaking innovation that played a pivotal role in successfully getting the Apollo spacecraft to the moon, and to our first steps on another world. The filter helped guide the Apollo spacecraft accurately through its circumlunar orbit. 

![1555943868943](assets/1555943868943.png)

The engineers at NASA's Ames Research Center, adopted Kalman's linear theory and extended it to nonlinear models. We'll look at this specific extension in upcoming module. But first, let's talk about the basic linear Kalman filter. 

---

### 3. The Kalman Filter | Prediction and Correction

> 虽然递推最小二乘法更新了静态参数的估计，但卡尔曼滤波器能够更新和估计一个进化状态

The Kalman filter is very similar to the linear recursive least squares filter we discussed earlier. While recursive least squares updates the estimate of a static parameter, but Kalman filter is able to update and estimate of an evolving state. **The goal of the Kalman filter is to take a probabilistic estimate of this state and update it in real time using two steps; prediction and correction.** To make these ideas more concrete, let's consider a problem of estimating the 1D position of the vehicle. 

Starting from an initial probabilistic estimate at time k minus 1, our goal is to use a motion model which could be derived from wheel odometry or inertial sensor measurements to predict our new state. 

![1555943972575](assets/1555943972575.png)

Then, we'll use the observation model derived from GPS for example, to correct that prediction of vehicle position at time k. Each of these components, the initial estimate, the predicted state, and the final corrected state are all random variables that we will specify by their means and covariances. In this way, we can think of the Kalman filter as a technique to fuse information from different sensors to produce a final estimate of some unknown state, taking into account, uncertainty in motion and in our measurements. 

---

### 4. The Kalman Filter | Linear Dynamical System

For the Kalman filter algorithm, we had been able to write the motion model in the following way; the estimate at time step k is a linear combination of the estimate at time step k minus 1, a control input and some zero-mean noise. 

![1555944141213](assets/1555944141213.png)

The input is an external signal that affects the evolution of our system state. In the context of self-driving vehicles, this may be a wheel torque applied to speed up and change lanes, for example. Next, we will also need to define a linear measurement model. 

![1555944203679](assets/1555944203679.png)

Finally, we'll need a measurement noise as before and a process noise that governs how certain we are that our linear dynamical system is actually correct, or equivalently, how uncertain we are about the effects of our control inputs. 

![1555944249190](assets/1555944249190.png)

---

### 5. The Kalman Filter | Recursive Least Squares + Process Model

Once we have our system in hand, we can use an approach very similar to that we discussed in the recursive least squares video. The Kalman filter is a recursive least squares estimator that also includes a motion model.

Except this time, we'll do it in two steps. First, we use the process model to predict how our states, remember, that we're now typically talking about evolving states and non-state parameters evolved since the last time step, and will propagate our uncertainty. 
$$
\begin{aligned} \check{\mathbf{x}}_{k} &=\mathbf{F}_{k-1} \mathbf{x}_{k-1}+\mathbf{G}_{k-1} \mathbf{u}_{k-1} \\ \check{\mathbf{P}}_{k} &=\mathbf{F}_{k-1} \hat{\mathbf{P}}_{k-1} \mathbf{F}_{k-1}^{T}+\mathbf{Q}_{k-1} \end{aligned}
$$
Second, we'll use our measurement to correct that prediction based on our measurement residual or innovation and our optimal gain. 
$$
\mathbf{K}_{k}=\check{\mathbf{P}}_{k} \mathbf{H}_{k}^{T}\left(\mathbf{H}_{k} \check{\mathbf{P}}_{k} \mathbf{H}_{k}^{T}+\mathbf{R}_{k}\right)^{-1}
$$
Finally, we'll use the gain to also propagate the state covariance from our prediction to our corrected estimate. 
$$
\begin{aligned} \hat{\mathbf{x}}_{k} &=\check{\mathbf{x}}_{k}+\mathbf{K}_{k}\left(\mathbf{y}_{k}-\mathbf{H}_{k} \check{\mathbf{x}}_{k}\right) \\ \hat{\mathbf{P}}_{k} &=\left(\mathbf{1}-\mathbf{K}_{k} \mathbf{H}_{k}\right) \mathbf{P}_{k} \end{aligned}
$$
In our notation, the hat indicates a corrected prediction at a particular time step. Whereas a check indicates a prediction before the measurement is incorporated. 

![1555944542344](assets/1555944542344.png)

If you've worked with the Kalman filter before, you may also have seen this written with plus and minus signs for the corrected and predicted quantities, respectively. 

---

### 6. The Kalman Filter | Prediction & Correction

Let's recap. We start with a probability density over our states and also maybe parameters at time step k minus 1, which we represent as a multivariate Gaussian. We then predict the states at time step k using our linear prediction model and propagate both the mean and the uncertainty; the covariance, forward in time. 

![1555944615676](assets/1555944615676.png)

Finally, using our probabilistic measurement model, we correct our initial prediction by optimally fusing the information from our measurement together with the prior prediction through our optimal gain matrix k. Our end result is an updated probabilistic estimate for our states at time step k. The best way to become comfortable with the Kalman filter is to use it. Let's look at a simple example. Consider again the case of the self-driving vehicle estimating its own position. Our state vector will include the vehicle position and its first derivative velocity. 

Our input will be a scalar acceleration that could come from a control system that commands our car to accelerate forward or backwards. For our measurement, we'll assume that we're able to determine the vehicle position directly using something like a GPS receiver. Finally, we'll define our noise variances as follows: 

![1555944648178](assets/1555944648178.png)

Given this initial estimate and our data, what is our corrected position estimate after we perform one prediction step and one correction step using the Kalman filter? Here's, how we can use these definitions to solve for our corrected position and velocity estimates. 

![1555944698834](assets/1555944698834.png)

Pay attention to the fact that our final corrected state covariance is smaller. That is we are more certain about the car's position after we incorporate the position measurement. 

![1555944718984](assets/1555944718984.png)

This uncertainty reduction occurs because our measurement model is fairly accurate. That is, the measurement noise variance is quite small. Try increasing the measurement variance and observe what happens to the final state estimate. 

---

### 7. Summary

To summarize, the Kalman filter is similar to recursively squares, but also adds a motion model that defines how our state evolves over time. The Kalman filter works in two stages: First, predicting the next state using the motion model, and second, correcting this prediction using a measurement. But how can we be sure that the Kalman filter is giving us an accurate state estimate? In the next video, we'll discuss a few appealing theoretical properties of the Kalman filter that have made it such a staple in the engineering field.

---

## Lesson 2: Kalman Filter and The Bias BLUEs

Now, we've introduced the Kalman Filter, let's discuss little bit about what makes it such an appealing estimation method. By the end of this lesson, you'll be able to define a few terms that are important in state estimation. 

- Define bias
- Define consistency
- Explain why the Kalman filter is the Best Linear Unbiased Estimator(BLUE)

---

### 1. Bias in State Estimation

Let's dive in. First, let's discuss bias. Let's consider our Kalman Filter from the previous lesson and use it to estimate the position of our autonomous car. If we have some way of knowing the true position of the vehicle, for example, an oracle tells us, we can then use this to record a position error of our filter at each time step k. Since we're dealing with random noise, doing this once is not enough. We'll need to repeat this same process over and over and record our position error at each time step. 

![1556028440311](assets/1556028440311.png)

Once we've collected these errors, if they average to zero at a particular time step k, then we say the Kalman Filter estimate is unbiased at this time step. 

![1556028641962](assets/1556028641962.png)

Graphically, this is what the situation may look like. Say the particular time step, we know that the true position is the following. We build a histogram of the positions that our filter reports over multiple trials, and then compute the difference between the average of these estimates and the true position. If this difference does not approach zero, then our estimate is biased. However, if this difference does approach zero as we repeat this experiment many more times, and if this happens for all time intervals, then we say that our filter is unbiased. 

This filter is an unbiased if for all k
$$
E\left[\hat{e}_{k}\right]=E\left[\hat{p}_{k}-p_{k}\right]=E\left[\hat{p}_{k}\right]-p_{k}=0
$$
Although we could potentially verify this lack of bias empirically, what we'd really like are some theoretical guarantees. Let's consider the error dynamics of our filter. 

- Predicted state error：$\check{\mathbf{e}}_{k}=\check{\mathbf{x}}_{k}-\mathbf{x}_{k}$
- Corrected estimate error : $\hat{\mathbf{e}}_{k}=\hat{\mathbf{x}}_{k}-\mathbf{x}_{k}$

Defining our predicted and corrected state errors, we can then use the common filter equations to write the following relations. 
$$
\begin{aligned} \check{\mathbf{e}}_{k} &=\mathbf{F}_{k-1} \check{\mathbf{e}}_{k-1}-\mathbf{w}_{k} \\ \hat{\mathbf{e}}_{k} &=\left(\mathbf{1}-\mathbf{K}_{k} \mathbf{H}_{k}\right) \check{\mathbf{e}}_{k}+\mathbf{K}_{k} \mathbf{v}_{k} \end{aligned}
$$
For the Kalman Filter, we can show the expectation value of these errors is equal to zero exactly. For this to be true, we need to ensure that our initial state estimate is unbiased and that our noise is white, uncorrelated, and zero mean. 

![1556029088893](assets/1556029088893.png)

While this is a great result for linear systems, remember that this doesn't guarantee that our estimates will be error free for a given trial, only that the expected value of the error is zero. 

---

### 2. Consistency in State Estimation

Kalman Filters are also what is called consistent. By consistency we mean that for all time steps k, the filter co-variants P sub k matches the expected value of the square of our error. 

![1556029158538](assets/1556029158538.png)

For scalar parameters, this means that the empirical variance of our estimate should match the variance reported by the filter. 

The filter is consistent if for all k,
$$
E\left[\hat{e}_{k}^{2}\right]=E\left[\left(\hat{p}_{k}-p_{k}\right)^{2}\right]=\hat{P}_{k}
$$
Practically, this means that our filter is neither overconfident, nor underconfident in the estimate it has produced. A filter that is overconfident, and hence inconsistent, will report a covariance that is optimistic. That is, the filter will essentially place too much emphasis on its own estimate and will be less sensitive to future measurement updates, which may provide critical information. 

One can also show (with more algebra!) that for all k,
$$
E\left[\mathbf{è}_{k} \check{\mathbf{e}}_{k}^{T}\right]=\check{\mathbf{P}}_{k} \quad E\left[\hat{\mathbf{e}}_{k} \hat{\mathbf{e}}_{k}^{T}\right]=\hat{\mathbf{P}}_{k}
$$
Consistent predictions !

Provided,
$$
E\left[\hat{\mathbf{e}}_{0} \hat{\mathbf{e}}_{0}^{T}\right]=\check{\mathbf{P}}_{0} \quad E[\mathbf{v}]=\mathbf{0} \quad E[\mathbf{w}]=\mathbf{0}
$$
It's easy to see how an overconfident filter might have a negative or dangerous effect on the performance of self-driving car. Showing the consistency property formally is outside of the scope of the course. You can take my word that it is indeed true for a common filter. So long as our initial estimate is consistent, and we have white zero mean noise, then all estimates will be consistent. 

---

### 3. The Kalman Filter is the BLUE | Best Linear Unbiased Estimator

Putting everything together, we've shown that given white uncorrelated zero mean noise, the Kalman Filter is unbiased and consistent. 

In general , if we have white ,uncorrelated zero-mean noise ,the Kalman filter is the best (i,e.., lowest variance ) unbiased estimator that uses only a linear combination of measurements

Because of these two facts, we say that the Kalman Filter is the BLUE, the best linear unbiased estimator. It produces unbiased estimates with the minimum possible variance. 

---

### 4. Summary

To summarize, in this lesson we've defined the terms bias and consistency, and showed that the Kalman Filter is unbiased, consistent, and the Best Linear Unbiased Estimator, or BLUE. Remember that best here refers to the fact that the Kalman Filter minimizes the state variance. Although this is a fantastic result, most real systems are not linear. For self-driving cars, we'll generally need to estimate non-linear quantities like vehicle poses, position, and orientation in 2D and 3D. To do this, we'll need to extend the linear Kalman Filter into the non-linear domain. We'll do this in the next lesson.