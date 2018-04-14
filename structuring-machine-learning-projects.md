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

## Mismatched Training and Test Data

- Often, labeled training data is hard to come by, and differs from real-world data distributions
- If your train and test sets have different distributions, there are subtle issues
- For example, high-resolution images with labels, but real-world images are from mobile and lower-resolution

| Type | # |
| ------------- | ------------- |
| High-resolution Web Images | 200,000 |
| Low-resolution Mobile Images | 10,000 |

- If you have a lot of high-resolution images and a few low-resolution images you can
- One approach is to combine them, and shuffle them for training.  This results in the same distribution for dev and test

| Training Set | Dev Set | Test Set |
| ------------- | ------------- | ------------- |
| 200,000 (high and low resolution images) ▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬ | 10,000 (high and low resolution images) ▬▬▬▬ | 10,000 (high and low resolution images) ▬▬▬▬ |

- But the distribution is still wrong! Your dev and test are heavily biased towards high-resolution
- Andrew suggestion instead:
    + Train - all high resolution, a few from low-resolution
    + Dev / test - all low resolution

| Training Set | Dev Set | Test Set |
| ------------- | ------------- | ------------- |
| 200,000 (high-resolution) ▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬ | 10,000 (low-resolution) ▬▬▬▬ | 10,000 (low-resolution) ▬▬▬▬ |

- So the target your aiming at is the real distribution
- The downside of course is training distribution is different from dev / test, but in the long run this approach is better

### Bias and Variance With Mis-Matched Data

In the cat-classifier example:

| Type | Error |
| ------------- | ------------- |
| Training error | 1% |
| Dev error | 10% |

- Create a new set called the *Training-dev set*, same distribution as test, but not used for training.

| Training Set | Train-dev | Dev Set | Test Set |
| ------------- | ------------- | ------------- |
| ▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬ | ▬▬▬▬ | ▬▬▬▬ | ▬▬▬▬ |

- When you evaluate the model on training-dev as well:

| Type | Error |
| ------------- | ------------- |
| Training error | 1% |
| Training-dev error | 9% |
| Dev error | 10% |

- The training set obviously doesn't contain dev data.  But when the model is run on a mix of dev data, error shoots up.  This indicates a *variance* problem
- Even though it does well on the train set, it's failing to generalize

#### More Examples:

| Type | Error |
| ------------- | ------------- |
| Training error | 1% |
| Training-dev error | 1.5% |
| Dev error | 10% |

- This is an example of a data-mismatch problem

| Type | Error |
| ------------- | ------------- |
| Human / Bayes error | 0% |
| Training error | 10% |
| Training-dev error | 11% |
| Dev error | 12% |

- Lots of avoidable bias

| Type | Error |
| ------------- | ------------- |
| Human / Bayes error | 0% |
| Training error | 10% |
| Training-dev error | 11% |
| Dev error | 20% |

- Avoidable bias, data mismatch

#### In-General

| Type | Error | Type |
| ------------- | ------------- | ------------- |
| Human / Bayes error | 4% | Avoidable bias ↓ |
| Training error | 7% |Variance ↓ |
| Training-dev error | 10% |Data mis-match ↓ |
| Dev error | 12% |Over-fitting with dev set ↓ |
| Test error | 12% | |

Over-fitting with the dev set, could be corrected by getting more dev set data.

### Addressing Data-Mismatch

- It's hard, no systematic way to do this
- Manual error analysis can help, what's the difference between dev / test
    + Maybe it's noisy (for speech recognition, maybe it can't recognize car noise)
    + Mis-recognizing specific items (like street numbers)
- You can simulate this data, for example mixing samples of car noise into training utterances
- If there's not a lot of sample data, you can end up over fitting to that, for example 100,000 hour of utterances and 1 hour of car noise
- The solution is to add more car noise samples

#### Common Issues

- You're working on car recognition, and have limited examples
- So you get a video game with cars, and now you have unlimited 
- But if there are only 20 types of cars in the video game, it will probably over-fit to these 20

## Learning from Multiple Tasks

### Transfer learning!

- **Transfer learning!**  Use a model on general image recognition, re-purpose it to a different task like radiology

Given an image recognition model (x general images, y classification):

|     |     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
|     |     |     | ◻   | ◻   |     |     |     |     |
|     | ◻   | ◻   | ◻   | ◻   | ◻   |     |     |     |
| x ->  | ◻   | ◻   | ◻   | ◻   | ◻   | ◻   | -> y   |     |
|     | ◻   | ◻   | ◻   | ◻   | ◻   |     |     |     |
|     |     |     | ◻   | ◻   |     |     |     |     |

Delete output layer, weights feeding into the last layer.  Randomly initialize weights for the last layer. 

|     |     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
|     |     |     | ◻   | ◻   |     |     |     |     |
|     | ◻   | ◻   | ◻   | ◻   | ◻   |     |     |     |
| x ->  | ◻   | ◻   | ◻   | ◻   | ◻   | X   | -> y   |     |
|     | ◻   | ◻   | ◻   | ◻   | ◻   |     |     |     |
|     |     |     | ◻   | ◻   | ^ randomize   |     |     |     |

- Lock all the layers except the last one
- Feed in a new data set, where x are radiology images, and y are diagnosis.  This is called *pre-training*
- If you have more data, un-lock additional layers for more accuracy
- The initial weights are considered to be *pre-training*
- You can also add additional layers
- Learning from a larger data set helps with general feature recognition

#### When does transfer learning make sense?

- You have a pre-trained model on a similar problem
- You have limited data for the specific problem you're working on
- But if the reverse is true, you have more radiology images that cats and dogs for example, the value of the transfered-model is low

In general:

- when Task A and Task B have the same input x
- You have more data for Task A than Task B
- You suspect low-level features for A could be useful for B

### Multi-Task Learning

- Example: you're building a self-driving car.  It will need to detect:
    + Pedestrians
    + Other cars
    + Stop signs
    + Traffic lights

In earlier examples 1 image resulted in one classification:

|| x(i) | y(i) |
| ------------- | ------------- | ------------- |
| Image 1 | 1 |
| Image 2 | 0 |


In multi-task, 1 image results in multiple classifications.

|| x(i) | y(i) |
|-------------| ------------- | ------------- |
|| Pedestrians | 0 |
|| Cars | 1 |
|| Stop signs | 1 |
|| Traffic Lights | 0 |

```
Y = [y1, y2 .. yn], where y is a vector (4,n)
```

You can update your neural-architecture to add a 4 dimensional output layer:

|     |     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
|     |     |     | ◻   | ◻   |     |     |     |     |
|     | ◻   | ◻   | ◻   | ◻   | ◻   | ◻   | -> pedestrian    |     |
| x ->  | ◻   | ◻   | ◻   | ◻   | ◻   | ◻   | -> car  |  -> y   |
|     | ◻   | ◻   | ◻   | ◻   | ◻   | ◻   | -> stop sign    |     |
|     |     |     | ◻   | ◻   |     | ◻   | -> traffic light    |     |


The loss function is the sum of the losses for all predictions for each of the four categories:

```
    m   4
1/m ∑   ∑
    i=1 j=1 
```

- The loss will be the sum of the components, usual logistic loss
- Unlike softmax regression which was 1 image to 1 label
- You have 1 image to multiple labels
- *This will work even if some of the images don't contain all the objects.*  In that case when doing the summation, skip adding the category that's missing from the image

#### When to use Multi-Task Learning

- You could just train 4 networks to recognize each object, and that would perhaps be more accurate
- But it might not be more accurate, and it would potentially be way more expensive
- You should use multi-task when amount of data for each lower level feature is similar
- And multi-task learning requires bigger networks.  If it's big enough, performance should be similar to training a bunch of little networks
- Multi-task is ideal when features being detected are similar, and the network can use information it has learned about one class to help recognize another

## End-to-End Learning

- Earlier approaches to learning required multiple stages of learning and processing.  Example audio processing:

```
audio clip -> features -> phonemes -> words -> transcript
```

End-to-End deep-learning skips this:

```
audio clip -> transcript
```

- For smaller data sets (< 10,000 samples maybe), the old approach still dominates
- For large data sets (> 1,000,000 samples), deep learning dominates
- For machine translation, there are lots of (x,y) pairs where X is english and Y is french, so E2E works great

### Where E2E is more difficult

- It's easier to do E2E when you can map directly to labeled data
- For example, recognizing people from headshots is much easier than recognizing them in group photos
- So in this case, E2E wont work as well as a first step of cropping the group image to get headshots first
- For determine child's age from X rays you could
    + Segment out bones from the X ray
    + Figure out length of each bone
    + Compare with table of bones length to age
- If you did E2E, not much labeled data, so not that great

*In general, E2E can help you skip pre-processing and build simpler models, if the data and the domain are appropriate.*

### Pros and Cons of E2E Learning

#### Pros

- Let's the data speak, less human pre-conditions
    + In the case of speech recognition, people were focusing on phonemes, forcing model to think in that way
    + But Andrew says "phonemes are a fantasy of linguists", if you let the algorithm to learn whatever it wants to learn, it's overall performance might be better
- Less hand-designing of components, simpler

#### Cons

- Requires lots of data
- Sometimes hand-designed components are useful, especially if you have limited data
- Justin's insight: hand-designed components are like transfer learning from humans to algorithms 😁

#### Applying E2E

- Automated Driving traditional approach
    + Get video, radar, lidar data
    + Figure out where the road is, pedestrians, other cars are
    + Figure out what path to take

```
image
radar   ->  cars            -> route    -> steering
lidar       pedestrians

       deep                motion       control
       learning            planning     software
```

- In this case, deep learning is used for categorization, motion planning and control software, customized hand-made components for later pipeline tasks