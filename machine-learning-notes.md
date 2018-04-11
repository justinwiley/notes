# General Machine Learning Notes

A collection of notes while reviewing machine learning / artificial intelligence sources.

### Marginal Probability

Probability of heads and tails, basically adding the probabilities

### Joint Probability

Frequency

P(X,Y) - probability x given Y

When rolling a die

P(heads, "# 3") = P(1/2) * P(1/6) = 1/12

Probability of A and B given B

### Conditional Probability

Probability one event will happen following another event

### Bayes Theory

P(H|E) = (P(E|H) / P(E)) * P(H)

### Central Limit Theorem

( 0.56 - 05 ) / ( 0.5 / sqrt(1000) ) = 3.79

y=f(x)+ϵ

y prediction
f(x) ideal function that describes how x influences y
ϵ error


### Parametric vs Non-Parametric

Parametric
- ML that masks assumption about distribution of data
- regressions

Non-parametric
- makes not assumptions
- SVM, neural nets

### Sensitivity (True Positive Rate), Specificity (True Negative Rate)

Sensitivity = TP / P

Specificity = TN / N

### Accuracy (ACC)

ACC = (TP + TN) / (TP + FP + FN + TN)

### F1

Harmonic mean of precision and sensitivity

F1 = 2TP / (2TP + FP + FN)

### AUC-ROC curve

Sensitivity / true positive rate vs specificity / false positive rate

### Type I / Type II Errors

Null hypothesis - nothing happened

### Type I

- "False positive" - you said something happened, rejected null hypothesis.  Surprise surprise, nothing happened.  False positive

### Type II

- "False negative" - you said nothing happened, accepted null hypothesis.  Something did happen.  False negative

### Confusion Matrix

```
    TP TN
TP  3  1
TN  1  3
```

TP / TN

### "Well-Calibrated"

### Over-fitting and Under-fitting

Over-fitting: high bias, model "hugs" the data too closely, considers factors that are not really important

Under-fitting: high variance, model doesn't hug the data at all, fails to consider important features

```
y = a + b <--- under fit (neglects important variable c)
y = a + b + c <--- good fit (gets important variables)
y = a + b + c + d + e + f <--- over fit (considers minor factors, gives them same weight as major factors)
```

Neural nets may be more resilient to over-fitting?

https://ml.berkeley.edu/blog/2017/07/13/tutorial-4/

### How do you prevent over-fitting / under-fitting?

Dimensionality reduction -> eliminate less important variables / variation

Over-fitting: low training error, high test error
Under-fitting: high training and test error

k-fold cross-validation

high bias? pick a better algorithm, or add complexity to the model

### How do you cut down on variance

Regularization

Add a penalty as model complexity increases.  dropout for neural nets, soft margin SVMs

#### Bagging and Re-sampling

- Bagging / replacement: bagging is "bootstrap aggregating"
- Sample the data multiple times to construct many different models, ensemble them, average results

### Dimensionality reduction

- Easy example, you are predicting housing prices from length and width of house (2 variables).
- Combine them to square footage (1 variable)
- Eigenvectors - favorite example, gives direction to tensor
PCA - principal component analysis, based on eigenvectors

### Supervised vs unsupervised

### Non-parametric

The machine learning model does NOT require data to fit particular distribution

### Instance-based / Lazy

### Parameter vs Hyper-parameter

Parameter: internal to the model, estimated / learned from the data

Hyper-parameter: external to the model, assumptions

Example parameters:
 - weights in neural network
 - coefficients in linear regression

Example hyper-parameters:
- learning rate in neural network
- C and sigma in support vector machines
- k in k nearest neighbors

### Random Forest

### Time-series Data

- remove seasonality, removing noise
- build model, starting with naive weighted average
- exponential smoothing model
- ARIMA

## Neural Networks

### Why is naive Bayes so ‘naive’?

Sees all features as equally important, and unrelated
You like picks, you like ice cream, there for you like pickle ice cream

### Prior probability, likelihood and marginal likelihood in context of naiveBayes

Proportion of dependent variable in the data set.  70% of of email is spam, there for 70% chance 

Likelihood is the probability of classifying a given observation as 1 in presence of some other variable. For example: The probability that the word ‘FREE’ is used in previous spam message is likelihood. Marginal likelihood is, the probability that the word ‘FREE’ is used in any message.

### Deep Learning



#### Tricks

- Pre-train on a related dataset!  
- If its expensive / impossible to get a radiologist to look at 1,000,000 images and label them, train on an analogous dataset first, then move to 10,000 labeled images.

## Comparing Machine Learning Models

### Decision-trees vs Time-series Regression

Decision trees: non-linear
Time-series regression: linear

If dataset has time aspect, linear regression potentially better

## General

### Data Issues

Unbalanced data

- In credit scoring: majority of people don't default, for medical imaging majority of patients are healthy
- Data is unbalanced, lots of examples of one category, not too many of another.  One possibility is to join two classes

### When to use ML vs algorithm

ML best suited for cases where:

- a pattern to exist
- an defined analytic solution is intractable
- you have data

### Machine Learning Algorithm In Simple Terms: k-nearest neighbors

During the last election, speculation about "bubbles".  Democrats talking to democrats, republicans talking to republicans.

So a being in a bubble involves interacting to people who think like you.

k-nearest neighbors gives us a method for describing:
  - the size of the bubble (the number of people nearest to you in opinions)
  - how intense the interactions are (the distance metric)

so for example, if we took a set of voters and plotted them by attitudes:

  - x: wealth
  - y: religiousness


How does wealth and religiousness predict republican / democrat?  With k nearest neighbors we look at each voter, and compare them with "k" other neighbors, where k is an integer.

We use a "distance metric" to figure out who those neighbors should be.  An easy one is "euclidean distance", which is (q1 - p1)^2 + (q2 - p2)^2 similar to the Pythagorean theorom from geometry.

So for each voter, find the k neighbors nearest to them 

Do bubbles make us more passionate, and if so to what extent

increasing k will decrease variance and increase bias

Err(x)= Bias^2 + Variance + Irreducible Error

k-nearest neighbors

unsupervised

bias / variance

### Designing Machine Learning Systems

Data layer

- like 90% of the ice berg
- logging
- retraining, upgrading the model
- update and deploy
- Machine-Learning Technical debt per [Sculley](http://static.googleusercontent.com/media/research.google.com/en//pubs/archive/43146.pdf)
    + if the model is poorly understood, its treated as cargo cult
    + lots of glue code, locks in assumptions
    + !!! models are brittle, Changing Anything Changes Everything
- get the data layer right
- data, data, data
- How do you guarantee the quality of the data you have?
- How do you guarantee the quality of data over time from outside sources?
    + generate summary statistics?  setup alerts?

UI / Product

- designing for uncertainty
- example: 
  - Uber or Lyft says they will arrive in 4 minutes, thats a probability
  - users treat it like a certainty
  - magical or not?
- work with designers, tell them what level of uncertainty we have with the numbers
- what level of stability and scale, how to present to customers
- customers who are talented, smart, and whose statistical learning stops at standard deviation