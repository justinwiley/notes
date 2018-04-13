# Structuring Machine Learning Projects

Notes while working through Coursera's [Structuring Machine Learning Projects course](https://www.coursera.org/learn/machine-learning-projects) by Andrew Ng.  **Warning: heavily paraphrased and incomplete.**

### Author Bio

"[Andrew Ng](http://www.andrewng.org/) is VP & Chief Scientist of Baidu; Co-Chairman and Co-Founder of Coursera; and an Adjunct Professor at Stanford University."

## Overview

The course provides guidance on how to structure machine learning projects to successfully meet goals in the most effective way possible.

It's common to run into issues when machine learning.  For example, you start a project and it isn't performing as expected.  What next?

- Collect more data
- Collect a more diverse training set
- Train longer
- Add dropout
- Add L2 regularization
- Apply N additional optimizations.

How, when and where should you apply these optimizations?

The course provides strategies for choosing optimization techniques.

### In general this involves the following framework:

1. Setup smart metrics to measure your progress
2. Build initial system quickly, favoring speed over complexity
3. Use Bias / Variance analysis, and Error Analysis to prioritize improvements. Iterate!

## Orthogonalization

*What to tune to achieve what effect.*

Examples of common parameters in real-life:

- knobs on a TV or the steering wheel and brakes on a car
- both are easy to tune, for example the steering wheel controls your direction
- but imagine the steering wheel was instead controlled by 0.3 * angle - 0.8 * speed, controlling the car is much harder
- so you want a control that affects one dimension only, either speed or direction, not both
- this seems similar to the concept of [orthogonality](https://en.wikipedia.org/wiki/Orthogonality) in geometry you're tuning parameters that are affect different axis

### Relationship to stages of ML training, and knobs

- In general ML involves fitting the "training set well on [a] cost function"
    + and then evaluating it on some metrics, human-level performance for example
- This is performed:
    + on a training set
    + with a dev set
    + with a test set
    + and on real world data after being deployed
- Andrew describes this as the *"Chain of Assumptions in ML"*
- Each of the techniques mentioned in the intro is (ideally) a knob you can tune at each stage, for example:
    + Fitting the model isn't work on the training set isn't working: try a bigger network, switch to Adam
    + Dev set isn't performing as well as initial model: regularization, more data
- Some knobs like stopping training early affect several stages, so are not as orthogonal


### Choosing ML Project Goals

### Metrics

- A single real number evaluation metric is crucial for fast iteration
- Machine learning is an empirical process, Idea -> Experiment -> Code
- [Precision and Recall](https://en.wikipedia.org/wiki/Precision_and_recall) are two possible metrics for evaluating ML, but if you use them both, it's harder to compare experiments
- Define a new metric that combines them, for example [F1 score](https://en.wikipedia.org/wiki/F1_score), and use that for the comparison

### Optimizing vs Satisficing

- If training speed is a consideration (and it often is), you can combine your metric with run-time
- This can be unwieldy and hard to average, so converting run-time to a threshold of say 100ms could be better
- *Optimizing* are metrics like accuracy, *Satisficing* are metrics that are considered minimums for successful models
- So 90% accuracy and a minimum 100ms runtime in combination could be a good set of metrics for evaluating models after you change hyper-parameters
- In general choose 1 optimizing metrics, and 1 or more satisficing metrics

### Seting up Test Sets

- Dev set = [hold out / cross-validation data](https://www.kdnuggets.com/2017/08/dataiku-predictive-model-holdout-cross-validation.html) not used to train the model
- Don't choose dev sets that have different distributions than your training data!  For example: chosing data from low-income zip codes to train and high-income as test
- Shuffle the data instead

#### How much for test / train split?

- In the old days 70% train, 30% test used to be common.  Or 60% train, 30% dev, 20% test with 100 to 10,000 examples
- But if you have a million examples, it's much more reasonable to use larger fractions, like 98% train
- In Andrew's parlance, compared to the [Wikipedia](https://en.wikipedia.org/wiki/Training,_test,_and_validation_sets) definition:
    + test = test (fitting the model)
    + dev = validation (tuning hyper-parameters)
- Some projects skip test and just use dev/test, which may be acceptable given requirements

#### When to change?

- If some of your metrics are more important than others (like the model including pornography when it's not allowed), you can come up with a weighted metric that includes that metric, heavily weighted 
- Andrew includes an example but says the exact method isn't important
- This is an example of orthogonality, since it takes two or more metrics and converts to one

*In general, when choosing metrics, choose ones that lead to actual performance in the real world.  If your model works poorly when deployed, you've picked the wrong metrics and/or dev/test set.*

## Comparing with Humans

- easily understandable metric for comparison
- often after surpassing human performance, progress on the algorithm slows down to the [Bayes optimal error](https://en.wikipedia.org/wiki/Bayes_error_rate), i.e. the best theoretical performance
- one reason: people are really good at a lot of tasks
- another reason: humans can label data if performance <= human performance, but can't after
    - humans also can't easily judge [bias and variance](https://en.wikipedia.org/wiki/Bias%E2%80%93variance_tradeoff) for super-human performance

*On many ML tasks, like image recognition, human performance is a good proxy for Bayes error.*

### Bias

- If human accuracy is 99% and model training 8%, and dev error 10% this is could be considered an indicator of high bias / under-fitting.  One solution could be increasing the data set
- If human accuracy is 97.5%, training 8% and dev 10%, it may indicate the test data doesn't reflect how messy real-world data is.  Tune for variance
- Andrew describes difference between human / bayes accuracy and training error as *avoidable bias* (the model is over-fitting on the training set), between training error and dev error *avoidable variance* (the model is failing to recognize new data, failing to generalize).

### More precise definition of human performance

- Given medical image diagnosis, an untrained human has 10% error, a doctor has 1% error.  What is human performance?  In this case the doctor is closer to Bayes optimal error so that's a good choice
- For cat recognition, the definition can be much looser
- What value is chosen directly influences avoidable bias and variance, and which to target when improving the model

### Superhuman Performance

- When human performance > training error, avoidable bias and variance harder to judge.  What's the real Bayes error rate?  Which one to focus on?
- On-line advertising, logistics, product recommendations, loan prediction examples of problem domains where ML has achieved super-human performance
- Usual these are domains with lots of structural data, vs natural perception tasks like image recognition

## Finale: Improving Model Performance

#### Reducing Avoidable Bias

- Train a bigger model
- Train longer, choose better optimization algorithms
    + [Momentum](http://ruder.io/optimizing-gradient-descent/index.html#momentum)
    + [RMSProp](http://ruder.io/optimizing-gradient-descent/index.html#rmsprop
    + [Adam](http://ruder.io/optimizing-gradient-descent/index.html#adam)
- Use a bigger training set
- Change neural-network architecture (RNN, CNN, etc), hyper-parameters search

#### Reducing Avoidable Variance

- More data
- Data regularization
    + [L1/L2](https://towardsdatascience.com/l1-and-l2-regularization-methods-ce25e7fc831c)
    + [Dropout](https://medium.com/@amarbudhiraja/https-medium-com-amarbudhiraja-learning-less-to-learn-better-dropout-in-deep-machine-learning-74334da4bfc5)
    + [Data augmentation](https://www.kaggle.com/dansbecker/data-augmentation)

### Flight-Simulator Examples

Andrew provides a few examples to test these concepts out.  Refer to the course for more information.

## Error Analysis

- You've built an image classifier, you've got 90% accuracy on a wide range of images
- Someone notices one way it's misclassifying is saying dogs are cats
- Should you focus on improving dog classification?
    + Collect more dog pictures
    + Change algorithm
    + etc?
- First look at how import and dog classification is to the importance of the overall algorithm
- If 5% of mislabeled images are dogs, totally eliminating it will only improve accuracy by 5% (i.e. 10% becomes 9.5%)

### Multi-Category Analysis

- You look at other sources of mis-categorization, and find its messing up on Zebras and blurry images
- You can construct a table like: 

| Misclassified Image | Dog | Zebra | Blurry |
| --- | --- | --- | --- |
| 1 |✓| |
|2 | |✓| |
|3| |✓|✓|
| N | ... | | |
| Total | 8% | 43% | 61% |

- So clearly some mis-classifications are more important to improve than others

### Incorrectly Labeled Examples

- What if your labeled images have mistakes, i.e. dogs are labeled as cats etc
- Deep learning algorithms are pretty resilient to this, if the errors are *reasonably random*
- Systematic errors, where all white dogs are labeled as cats for example, are much more problematic
- You can update your errors table with incorrect labels:

| Misclassified Image | Dog | Zebra | Blurry | Incorrect Labels |
| --- | --- | --- | --- | --- |
| 1 |✓| ||
|2 | |✓| ||
|3| |✓|✓||
| N | ... | | ||
| Total | 8% | 43% | 61% |6%|

- Once again, if incorrectly labeled errors don't impact overall error, don't focus on it
- Also evaluate the dev and test sets as well, to make sure they have the same distribution
- Consider examining examples your algorithm got right (if feasible)
- If you correct labels in the train set only, because it's must smaller, that's ok in practice
- *Manually reviewing examples is no-fun, but important!*  A few hours spent reviewing data can greatly improve results

### Building Your First System (Putting it all together)

- Build quickly and then iterate
- Example: speech recognition, many directions you can go in
    + Eliminating noisy input
    + Recognizing different accents
    + Distance from microphone
    + Stuttering
- Which one is the most important

#### Recommendations

1. Setup dev/test set, metric
2. Build initial system quickly
3. Use Bias / Variance analysis, and error analysis to prioritize

If you're tackling a new problem for the first time, don't make the system too complicated.  If it's a very well known problem like face-recognition, it may make sense to review existing literature and get more complicated from the get-go.

### Mismatched Training and Test Data

- Often, labeled training data is hard to come by, and differs from real-world data distributions
- If your train and test sets have different distributions, there are subtle issues
- For example, high-resolution images with labels, but real-world images are from mobile and lower-resolution
- If you have a lot of high-resolution images and a few low-resolution images you can
- One approach is to combine them, and shuffle them for training.  This results in the same distribution for dev and test
- But the distribution is still wrong! Your dev and test are heavily biased towards high-resolution
- Andrew suggestion instead:
    + Train - all high resolution, a few from low-resolution
    + Dev / test - all low resolution
- So the target your aiming at is the real distribution
- The downside of course is training distribution is different from dev / test, but in the long run this approach is better
