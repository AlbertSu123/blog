---
title: 'Data Science Notes'
date: '2021-10-21'
tags: ['data-science', 'class', 'berkeley']
draft: false
summary: 'Notes from Data 100 from UC Berkeley. Contains information about Inference, Modeling, Variance, Logistic Regression, Gradient Descent, and Cross Validation + Regularization.'
---

# Inference

- Moving from premises to logical consequences
- Induction is inference from a particular premise to a universal conclusion
- Statistical inference: Using data analysis to deduce properties of underlying distribution

## Prediction vs Inference

- Prediction: Using our model to make predictions for unseen data. (Given attributes of house, how much is it worth?) We don't care about how
- Inference: Using our model to draw conclusions about the underlying true relationships between our features and response. (How much extra will a house be worth if it has a view of the river?) We care about model parameters that are interpretable and meaningful

## Statistical Inference

- Draw conclusions about a population parameters given only a random sample
- Parameter: some function of population, ie population mean
- Estimator: some function of a sample, whose goal is to estimate a population parameter, ie sample mean. Estimators are random variables
  - Bias of an estimator: difference between estimator's expected value and the true value of the parameter being estimated.
  - Variance of an estimator: Expected squared deviation of an estimator from its mean

## Bootstrapping

- Idea: treat our random sample as a population and resample from it
- Psuedocode
  ```
  collect random sample of size n(aka bootstrap population)
  initialize list of estimates
  repeat 10,000 times:
      resample with replacement from bootstrap population
      apply estimator f to resample
      store in list
  list of estimates is the bootstrapped sampling distribution of f
  ```
- The median cannot be accurately drawn from bootstrapping
- If the sample is too small, bootstrapping won't work

## Confidence Interval

- What does a confidence interval p% mean?
  - If we take a sample from the population and compute P% confidence interval for the true population parameter, and repeat this many times, our population parameter will be in our interval P% of the time.
- to compute confidence interval(s,f,P), approximate sampling distribution of f using sample s. Choose middle P% of samples from this approximate distribution
- A 95% confidence interval does not mean that there is a 95% chance that the population parameter is in the interval, either the population parameter is in or isn't

## Bootstrapping Model Parameters

- Our estimate for theta depends on what our training data was.
- We want to think about all of the different ways that our training data and our parameter estimate could have come out
- We want to test whether a feature has any effect on the outcome. This works for linear and logistic regression models with any number of features.

```
Estimate theta1 each time
Make confidence interval for theta 1 and see if 0 is in the interval
    If yes, theta 1 is not significantly different than 0
    If no, theta 1 is significantly different than 0
```

## Multicollinearity

- If features are related to one another, it might not be possible to have a change in one while holding the others constant
- Multicollinearity: Where a feature can be predicted fairly accurately by a lienar combination of other features.
  - Doesn't impact model predictability, only interpretability
- Perfect Multicollinearity: one feature can be written exactly as linear combination of other features

## Summary

- Estimators are functions that provide estimates of true population parameters
- We can bootstrap to estimate the sampling distribution of an estimator
- Using the bootstrapped sampling distribution, we can compute a confidence interval for our estimator
  - This gives a rough idea of how uncertain we are about the true population parameter
  - Only valid if the original random sample is representative
- The assumption when performing linear regression is that there is some true parameter theta that defines the linear relationship between features X and response Y
  - We can use bootstrap to determine whether or not an individual feature is significant
- Multicollinearity arises when features are correlated with one another
- Supervised Learning = We have an X and Y
- Unsupervised Learning = We only have x, want to learn about X

# Modeling

## Motivation

Predict unknown values based on known values

## Modeling Process

Choose a Model:** Constant model
Choose a Loss Function:** Squared Loss, Absolute Loss
Minimize average loss across entire dataset to determine optimal parameters

## Correlation

**Correlation Coefficient (r):** Measures the strength of the LINEAR association between two variables

- r is unitless and ranges between -1 and 1
  - if r = 1, x = y and all points fall exactly on the line
  - if r = -1, -x = y and all points fall exactly on the line
  - if r = 0, there is no linear association between x and y
- r says nothing about causation or non-linear association. Remember correlation does not imply causation!

r = average of the product of x and y, both measured in standard units
x_i in standard units = (x_i - x) / o_x

**Covariance:** r _ std_x _ std_y

## Simple Linear Regression

Motivation: Want to predict value of y for and given x. A naive attempt for getting children heights given parent heights is to compute the average value of y for each x value, then use those as predicts.

Simple linear regression: y = a + bx
To determine optimal a and b, choose a loss function. If the loss function is squared loss, the objective function is mean squared error(MSE).

To solve for the optimal parameters, we use the objective function and minimize the mean squared error by hand using calculus.

b = r _ (o_y/o_x)
a = y_bar - b _ x_bar
This gives us parameter estimates for x and y.

## Loss Surfaces

Usually 3d, with axes being a, b and loss(y = a + bx)

## Model Interpretation

Slope - measured in units of y per unit of x
New data needs to be similar to original data - you cannot predict the weight of a chihuahua's weight given a model for golden retrievers
Visualize, then quantify - watch out for anscombe's quartet

## Terminology

**Names for the x variable:**

- Feature
- Covariate
- Independent variable
- Explanatory variable
- Predictor
- Input
- Regressor

**Names for the y variable:**

- Output
- Outcome
- Response
- Dependent variable

## Adding independent variables

Use a weighted sum of coefficients and input variables.

## Evaluating Models

- Look at Mean Squared Error(MSE) or Root Mean Squared Error(RMSE)
- Look at the correlations
- Look at a residual plot

Root Mean Squared Error: Square root of the mean squared error. RMSE is in the same units as y. A lower RMSE indicates more accurate predictions. It is impossible to lower the RMSE just by adding features using the same data

**R squared**: Used to measure the strength of the linear association between our actual y and predicted y. aka coefficient of determination.

R^2 = variance of fitted values / variance of y

## Ordinary Least Squares

### Multiple regression using matrix multiplication

Multiple regression is of the form
`y = theta_0 + theta_1 * x_1 + theta_2 * x_2 + ... + theta_p * x_p`
We can restate this as a dot product
`y = x^T * theta`

### Design Matrix

Motivation: the mean squared error involves all observations at once, it would be nice to express our model in terms of all observations, not just one. We can put them into a design matrix.

**Rows:** Correspond to observations. e.g. all features for data point 3
**Columns:** Correspond to features. e.g. feature 1, for all data points

## Residuals

Residuals are the difference between an actual and predicted value, in the regression context. We use the letter `e` to denote residuals, `e_i = y_i - yhat_i`

The mean squared error is equal to the mean of the squares of its residuals.
We can stack all n residuals into a vector, called the residual vector.
`residual vector = true y values - predicted y values`

Residuals are orthogonal to the span of X.
If our model has an intercept term(when our design matrix has a column of all 1s)

- The sum and mean of the residuals is 0
- The average true y value is equal to the average predicted y value

**Residual Plots:**

- With simple linear regression with only 1 independent variables, we plot residuals vs x
- In the general case, use residuals on y axis vs fitted values on x
- A good residual plot has no pattern, if there is a curve, this is a sign that transformations or additional variables can help
- A residual plot should have a similar vertical spread throughout the entire plot. If it doesn't there are probably issues with the accuracy of the predictions

## Unique Solutions

- There is always at least one model parameter that minimizes average loss.
- Constant models with a squared loss: a unique solution always exists
- Simple linear model with a squared loss: Any non constant value has unique mean, SD, correlation coefficient
- Constant model with absolute loss: Unique when there is an odd number y values, if there is an even number of y values, there are infinitely many solution.

## Invertability of X transpose \* X

- Invertible iff it is full rank
- X transpose \* X and X have the same rank
- Thus, X^T \* X is invertible iff X has rank p + 1 (full column rank)

# Real World Example - Fairness in Housing Appraisal

## Situation

The cook county assessor's office is in charge of assessing property values in order to determine property taxes.

## Problem

The biased property value assessment resulted in a regressive tax, where rich people paid less and poor people paid more. In addition, rich people appealed more often than poor people, resulting in an even greater reduction of property tax.

## Solution

1. **Ask a Question:** What do we want to know? How to fairly value things for tax purposes. What are our metrics for success? Have both fairness and transparency in projections.
2. **Data Acquisition and Cleaning:** What data do we have and what do we need? Housing Sales data between 2013-2019, Property Characteristics-ie age, bedrooms, baths, etc.How will we sample more data? Is our sample representative?
3. **Exploratory Data Analysis and Visualization:** What attributes are most predictive of sales price? Which are potentially problematic? Is the data predictive of sales price?

## Takeaways

1. Accuracy is a necessary, but not sufficient condition of a fair system.
2. Fairness and transparency are context-dependent
3. Learn to work with contexts and consider how your data analysis will reshape them
4. Keep in mind the power and limits of data analysis?

# Probability and Generalization

## Random Variables

**Random Variable:** Represents a numerical value determined by a probabilistic event.

### Probability Mass Function

- The distribution of a random variable X provides the probability that X takes on each of its possible values(discrete)
- The probabilities for all possible values of random variable X in a Probability Mass Function must sum to 1
- Each individual probability for a given value X must be between 0 and 1.
  **Joint Distributions:** Probability of two or more random variables taking on a specific set of values. Ie P(X=0, Y= 10) = (0.5) ** 10 for coin flips where X is heads and Y is tails
  **Marginal Distribution:\*\* A way to go from the joint distribution to the distribution for a single variable. Ie consider all possible values of Y that can simultaneously happen with X and sum over all of the joint probabilities.  
  ∑y∈Y P(X=x,Y=y) = P(X=x)

**Independent Random Variables:** Any two random variables are independent if and only if knowing the outcome of one variable does not alter the probability of observing any outcomes of the other variables.

## Expectation and Variance

### Expectation

- The long run average of a random variable, also known as the expected value or expectation of a random variable.
- E[X] = ∑ x∈X x ⋅ P(X=x)

**Linearity of Expectation:**

- Use when working with linear combinations of random variables. This holds true even when the random variables are dependent on each other.
- E[X+Y] = E[X] + E[Y]
- E[cX] = c \* E[X]
- E[X−Y] = E[X] − E[Y]

However, E[XY]=E[X]E[Y] is only true when X and Y are independent random variables.

### Variance

- The variance of a random variable is a description of the variable's spread, or how far values are apart from each other.
- Var(X) = E[(X − E[X])\*\*2]
- Var(X) = E[X**2] − (E[X])\*\*2
- Var(aX + b) = a\*_2 _ Var(X) Is true if X is a random variable
- Var(X + Y) = Var(X) + Var(Y) Holds true if X and Y are independent

### Covariance

If the covariance is positive, the random variables are positively correlated(ie move in the same direction for stocks). If the covariance is negative, the random variables are negatively correlated. A covariance of 0 indicates the variables are independent.

- Cov(X,Y) = E[(X − E[X]) \* (Y − E[Y])]
- Cov(X,Y)= E[XY] − E[X]\*E[Y]

## Risk

**Risk:** Statistical risk is known as the expected loss, or the expected value of the model's loss on randomly chosen points from the population.

- R(θ) = E[(X − θ)**2]
- To minimize risk, use R(θ) = E[(X − E[X]) ** 2] + (E[X] − θ) ** 2
- R(θ) = Bias + Variance = (E[X] − θ) ** 2 + E[(X − E[X]) ** 2]
- A low variance means the random variable will likely take a value close to θ, while a high variance means the random variable will take a value far from θ

### Empirical Risk Minimization

- Since calculating the expected value of X requires complete knowledge of the population, since expected value is defined as probability X takes a specific value \* that specific value.
- We can use a large random sample instead of the population when calculating the expected value of X
- Thus we can approximate E[X] ~ mean(x)
- Therefore, the empirical risk is the risk from using the large random sample instead of the population

# Variance

**Random Variable:** A numerical function of a random sample, a statistic.
**Expectation:** Weighted average of the values of X, where the weights are the probabilities of the values
**Variance:** Expected squared deviation from the expectation of X, the units of variances are the units of X squared.

## Interpretation of Variance

- Var(X) = E[X^2] - (E[X])^2
- The main use of variance is to quantify chance error
- **Chebyshev's inequality:** The vast majority of the probability is around the expectation plus minus a few standard deviations
- If X is centered, ie E[X] = 0, then Var(X) = E[X^2]

## Linear Transformations

- E[aX + b] = aE[X] + b
- Var(aX+b)= a^2 \* Var(X)
- SD(aX+b) = |a|SD(X)
- Var(aX+b)=Var(aX)
- A shift by b units does not affect spread
- The multiplication by a does affect spread

## Standardization of random variables

- X in standard units = (X-E[X]) / SD(X)
- X in standard units measures the number of SDs from expectation
- It is a linear transformation of X
- E[X_su] = 0, SD[X_su] = 1
- E[X_su^2] = Var[X_su] = 1

## Variance of a sum -> Covariance

- Var[X+Y] = Var[X] + Var[Y] + 2\*Covariance
- Covariance = 2E[(x-E[X])(Y-E[Y])] =
- Covariance is 0 if X and Y are independent
- To get right of units for covariance, scale it by the standard deviation to get correlation
- Correlation = Cov[X,Y] / (SD[X] \* SD[Y])
- Uncorrelated random variables is when covariance = 0

## I.I.D. sample sum

- independent and identically distributed
- draws at random with replacement from a population are i.i.d.
- E[S_n] = n _ sample mean, Var[S_n]=n_(std dev)^2, SD[S_n] = sqrt(n) \* std dev

## Model Risk

- Mean squared error of prediction
- Model risk = E[(Y-Y(x))^2], Y(x) is a model that we are using
- **Chance Error:** Due to randomness alone in the new observations
- **Bias:** Non random error due to model being different from the true underlying function

## Bias and Overfitting

- Overfitting: small differences in random samples which leads to large differences in the fitted model
- Overfitting solution: Reduce model complexity
- Model risk = std dev^2 + (model bias)^2 + model variance

# Logistic Regression

## Linear Regression

Goal: Predict a number from a set of features
Output: A real number
Process: determine optimal parameters by minimizing some average loss and something with a regularization penalty.

## Classification

Goal: Predict a categorical variable(ie win or lose, disease or no disease, spam or ham)
Types: Binary Classification(Two classes, ie spam or not spam), Multiclass classication(type of animal)

## Logistic Regression Model

P(Y=1|x) = 1 / (1 + e^((-x^t)_theta)) = sigma(x^T _ theta)
The probability of an observation belonging to class 1 given x(vector of features) = Linear regression function on linear transformation of a linear combination of x(vector of features)

Thus, the logistic regression is sometimes known as a generalized linear model since it is a non-linear transformation of a linear model.

## Pitfalls of Squared Loss functions for logitisic regression

Can get trapped in a flat region if the surface is not convex
Solution: Use Cross-entropy Loss!
Empirical Risk for logistic regression when using cross-entropy loss  
`R(theta) = -(1/n)sum from i=1 to n( y_i * log(logistic regression(X_i^T * theta)) + (1 - y_i) * (1 - logistic regression(X_i^T * theta)))`

### Advantages of Cross-entropy loss

- Loss surface is guaranteed to be convex
- More strongly penalizes bad predictions
- Has roots in probability and information theory

## Logistic Regression

- Good for outliers since we wrap our data into a logistic function call
- L2 and L1 Loss Motivation: We want to measure the size of the error(difference between prediction and data), and for it to be indifferent to sign.
- To measure the difference between two probability distributions, the cross entropy loss is useful
- Use a threshold to classify a point on the logistic curve into a graph, this turns our decision rule into a classifier

## Evaluating Classifiers

- The most basic evaluation for a classifier is accuracy. This is widely used, but in the presence of class imbalance it doesn't mean much(since if we classify all the pictures as dog and dogs compose of 95% of the dataset our accuracy is 95%)
- Types of classification errors: True positive and True negatives(correct when classify observations as positive or negative), False positive(False alarm), False negative(We failed to detect)
- Confusion matrix: Gives us true + false positives and negatives for a given classifier

## Precision and Recall

- `Accuracy = (True positive + True negative) / number of data points`
  - This tells us what proportion of points did our classifier classify correctly
- `Precision = True positive / (True positive + False positive)`
  - Of all observations that were predicted to be 1, what proportion is actually 1? Penalized false positives
- `Recall = True positive / (True positive + false negative)`
  - Of all observations that were actually 1, what proportion did we predict to be 1? Penalizes false negatives
  - Can achieve 100% recall by making the classifier output 1 regardless of input, but makes precision low
- Generally, a higher classification threshold for positives equals fewer false positives, increases precision.
- On the other hand, a lower classification threshold has fewer false negatives, increases recall.

## Visual Metrics

- The only thing we can change after getting our model is to play around with our classification threshold
- Accuracy vs threshold: `Threshold too high = false negatives`, `Threshold too low = false positives`
- Precision vs threshold: `Threshold increases = fewer false positives`, precision tends to increase(not always)
- Recall vs threshold: `Threshold increases = more false negatives`, recall tends to decrease

## Other Metrics

- `False Positive Rate = False Positives / (False Positives + True Negative)`
  - What proportion of innocent people did I convict
- `True Positive Rate = True Positives / (True Positives + False Negatives)`
  - What proportion of guilty people did I convict? aka recall
- ROC Curve(Receiver Operating Characteristic): plots the false positive rate vs true positive rate
- AUC(Area under curve): Area under ROC curve, best possible = 1, worst possible = 0.5(randomly guessing)

# Cross Validation and Regularization

## The train-test Split

- **Training Data:** Used to fit model
- **Test Data:** Used to check generalization error
- How to split? Randomly, Temporally, Geographically (Usually Random)
- What size? Larger training set = more complex models, Larger test set = better estimate of generalization error (Usually 90%-10% or 75%-25%)

# Recipe for Successful Generalization

1. Split data into training(90%) and testing set(10%)
2. Use only training data when training, use cross validation to test generalization(do not look at testing data!)
3. Commit to the model and train once more using only training data
4. Test the model using testing data. If accuracy is not acceptable, return to step 2
5. Train on all available data and ship it

# Regularization

Parametrically controlling the model complexity
**Naive Solution:** Find the best value of theta which uses less than B features. Unfortunately, this is an NP hard combinatorial search problem.

- **L0 Norm Ball** Ideal for feature selection but combinatorically difficult to optimize
- **L1 Norm Ball** Encourages sparse solutions and is convex!
- **L2 Norm Ball** Spreads weight over features(robust) and does not encourage sparsity
- **L1 + L2 Norm(Elastic Net)** Compromise, need to tune two regularization parameters

**Standardization:** Ensure each dimension has the same scale and is centered around 0
**Intercept Terms:** Typically doesn't regularize the intercept term

# Ridge Regression

**Model:** Y = X \* theta
**Loss:** Squared Loss
**Regularization:** L2 regularization
**Objective Function** Squared Loss + added penalty

Note that there is always a unique optimal parameter vector for Ridge Regression!

# Lasso Regression

**Model:** Y = X \* theta
**Loss:** Squared Loss
**Regularization:** L2 regularization
**Objective Function** Average Squared Loss + added penalty

Note that there is NO closed form solution for the optimal parameter vector for LASSO so we must use numerical methods like gradient descent
