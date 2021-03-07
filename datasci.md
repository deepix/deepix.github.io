---
layout: page
title: Data Science
permalink: /datasci/
newlink: /datasci/
---

This page showcases a portfolio of Data Science projects I've done.

## Public

### Capstone project at UC Berkeley: Global Irrigation Map

We use satellite imagery, soil and climate data to map worldwide extent
of cropland irrigation, and how it has changed from 2001 to 2015.

I worked on this even after graduation, and a revised version is under
review for publication.

In collaboration with Dept of Environmental Studies, UC Berkeley.

[Version 2](http://deepix.github.io/2020/07/05/global-irrigation-map.html) |
[Version 1](http://www.globalirrigationmap.org/) |
[Code](https://github.com/deepix/globalirrigationmap) (all public)

### Improving diversity in NYC schools

_Which schools in New York need the most help, to improve diversity in SHSAT exam?_

SHSAT is a competitive exam to enter specialized high schools in NYC.
It is currently overwhelmingly White and Asian.  We check which
schools with Black and Latino populations hold potential but need
help.

KNN, Random Forests, Logistic Regression, and Perceptron Neural Network.

[Report](https://github.com/deepix/W207-final-project/blob/master/final_project_overview.ipynb) (public)

### Detecting fake news

_Can we detect fake news using machine learning?_

We try both classical techniques as well as recurrent neural networks.
Within the classical realm, we use "AutoML", which searches for the
optimal machine learning algorithm.  Our best accuracy was 81%.

[Report](https://github.com/deepix/w266finalproject/blob/master/Detecting%20Fake%20News%20with%20NLP%20-%20Merritt%2C%20Nagaraj%2C%20and%20Powers.pdf)
(public)

### Recognizing landmarks in Paris

We use convolutional neural networks to train and recognize landmarks
in Paris.

We use an NVIDIA GPU to train the network.  We train with transfer
learning (bottlenecks) as well as retraining pre-trained models.

MobileNet, Inception v3, AlexNet, VGG16.

The best performing model (MobileNet) had 95% accuracy.

[Report](https://github.com/deepix/w251finalproject/blob/master/w251_final_project_paper.pdf) [Presentation](https://github.com/deepix/w251finalproject/blob/master/W251_Final_Presentation_Deepak_Mike_Roland.pdf) (public)

### Can distractions affect your focus?

Statistical field experiment to measure causal inference: we use a
difference-in-differences, within-subject design.  We recruited about
150 subjects on Amazon MTurk for this experiment.  We measured focus
via response time and correctness, with both "relaxing" and
"busy-work" distractions.  We found statistical significance (at 95%
confidence level) for one of our claims.

[Report](https://docs.google.com/document/d/10j1c6jT9Xl_NE6KocqmeKRCGW8Nqkc7gSyFAX2GqLFg/edit?usp=sharing)
(public, dataset available upon request)


## Private

The following projects are private in accordance with Berkeley's
academic policy.  However, I can make them available to non-students
on individual basis upon request.

### Predicting success rate for online ads

_Will this user click on this ad?_

Online advertising is big business.  Ad companies make money only when
a user clicks on an ad, so they try hard to show ads on which a user
is likely to click.

On a large, anonymized dataset, we run logistic regression.  We get
76% accuracy, AUC 0.58.

[Report](https://github.com/deepix/w261-final-project/blob/master/final.ipynb) (private, available upon request)

### Effect of regulation on Internet access

_Does government regulation improve Internet connectivity?_

We explore whether regulation can lower costs, improve speed, or
provide more people with the Internet.

Exploratory data analysis with R statistical language.

[Report](https://github.com/deepix/W203/blob/master/W203_Project1_Chandrasekaran_Datta_Nagaraj.pdf) (private, available upon request)

### What is associated with crime?

We analyze whether factors like police density, chances of conviction,
proportion of young men or minorities affect crime rates.

Linear regression with R.

[Report](https://github.com/deepix/w203-deepix/blob/master/Lab_3/Chandrasekaran_Datta_Nagaraj_Lab3_Final.pdf) (private, available upon request)

### Classifying text into topics

We use word counts to build a text classifier.  We try with word
sequences (n-grams) and word frequency scores (TF-IDF), and logistic
regression.  We could classify text into newsgroups correctly 77% of
the time.

Natural language processing: CountVectorizer, TfidfVectorizer.
Classifier: logistic regression.

[Report](https://github.com/deepix/w207-projects/blob/master/deepak_nagaraj_p2.ipynb) (private, available upon request)

### Recognizing handwritten digits

How accurately can we recognize digits written by hand?  We use
classical machine learning algorithms.  We also check if blurring the
pictures can improve accuracy.

k-nearest neighbor, naive Bayes, and Gaussian NB.

[Report](https://github.com/deepix/w207-projects/blob/master/deepak_nagaraj_p1.ipynb) (private, available upon request)

### Detecting poisonous mushrooms

Based on a variety of properties, we check if a mushroom is poisonous.
We reduce the complexity of data and group mushrooms into poisonous
and non-poisonous categories.

Principal component analysis (PCA), k-means clustering, Gaussian mixture models

[Report](https://github.com/deepix/w207-projects/blob/master/deepak_nagaraj_p3.ipynb) (private, available upon request)


### Filtering spam at scale

We run naive-Bayes at scale with old-school Hadoop Map-Reduce.  Enron dataset.

[Notebook](https://github.com/deepix/w261/blob/master/Assignments/HW2/hw2_workbook.ipynb) (private, available upon request)

### Detecting synonyms on large data

We use Spark and document-similarity metrics to detect synonyms on Google's n-gram corpus.

[Notebook](https://github.com/deepix/w261/blob/master/Assignments/HW3/hw3_Workbook.ipynb) (private, available upon request)

### What determines wine quality?

Is it acidity? SO<sub>2</sub>? Density? Alcohol? Something else?

We use OLS, ridge and lasso regressions to determine features that predict wine quality.

[Notebook](https://github.com/deepix/w261/blob/master/Assignments/HW4/hw4_Workbook.ipynb) (private, available upon request)

### How Google works

PageRank was the original Google search algorithm.

We implement PageRank graph search algorithm on a large Wikipedia dataset, with Spark.

[Notebook](https://github.com/deepix/w261/blob/master/Assignments/HW5/hw5_workbook.ipynb) (private, available upon request)

### Analyzing movie-goer sentiment with neural networks

We implement a "bag-of-words" model, running on a neural network, to
analyze sentiment on the Stanford Sentiment TreeBank.  It cannot beat
a simple naive-Bayes model.

[Notebook](https://github.com/deepix/w266-assignments/blob/master/assignment/a2/NeuralBOW.ipynb) (private, available upon request)

### Can a computer learn a language?

We implement n-gram language models.  We first do smoothed models on
scikit-learn.  We then ramp this up with a recurrent neural network on
TensorFlow.

[Notebooks](https://github.com/deepix/w266-assignments/tree/master/assignment/a3) (private, available upon request)

### Tag sentences for grammar

We use the Viterbi algorithm to tag each word in a sentence with the
part of speech in English grammar.  We use a "hidden Markov model"
(HMM).

[Notebook](https://github.com/deepix/w266-assignments/blob/master/assignment/a4/Part-of-Speech.ipynb) (private, available upon request)

### Could we have prevented the Challenger space shuttle accident?

Statistical models say yes: given the temperature at the time of launch, it was likely to fail.

Logistic regression, binomial and binary.  Likelihood-ratio tests (LRTs).  Profile and Wald intervals.  Bootstrap.

[R Notebook](https://github.com/deepix/w271-labs/blob/master/Lab_1/DanielleAdler_CraigFujii_DeepakNagaraj_Lab1.pdf) (private, available upon request)

### Given how retailers place cereal boxes, can we predict where kids' cereals are?

Sugar and sodium were statistically significant, and sweeter cereal
boxes were likely sitting on the bottom shelves.

Multinomial logistic regression.  Odds ratios.

[R Notebook](https://github.com/deepix/w271-labs/blob/master/Lab_2/DanielleAdler_CraigFujii_DeepakNagaraj_Lab2.pdf) (private, available upon request)

### Forecasting e-commerce sales, given historical data

We develop a statistical time-series model.

Seasonal auto-regressive moving average (SARIMA).  Our best model was ARIMA(0,1,0)(1,1,0)<sub>4</sub>.

[R Notebook](https://github.com/deepix/w271-labs/blob/master/Lab_3/DanielleAdler_CraigFujii_DeepakNagaraj_Lab3_v2.pdf) (private, available upon request)

### Do traffic laws really reduce deaths due to accidents?

Speed limits, seat belts, blood alcohol limits: do these reduce fatalities?  (Yes.)

Panel data models with pooled data, fixed effects and random effects.

[R Notebook](https://github.com/deepix/w271-labs/blob/master/Lab_4/DanielleAdler_CraigFujii_DeepakNagaraj_Lab4_final.pdf) (private, available upon request)

Page under construction.  More projects on the way!
