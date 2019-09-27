---
layout: page
title: Data Science
permalink: /datasci/
---

This page showcases a portfolio of Data Science projects I've done.

In accordance with academic conduct policy of UC Berkeley, I cannot
make some projects public.  Contact me if interested.

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

### Bike share in San Fracisco

Which routes are the most popular for commuters, and how can we
improve efficiency of bike usage?

Advanced SQL on Google BigTable.

[Report coming soon]

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

### Analyzing crime

We analyze whether factors like police density, chances of conviction,
proportion of young men or minorities affect crime rates.

Linear regression with R.

[Report](https://github.com/deepix/w203-deepix/blob/master/Lab_3/Chandrasekaran_Datta_Nagaraj_Lab3_Final.pdf) (private, available upon request)

### Tracking user activity with a big-data pipeline

We did projects in data engineering with applications in user activity
tracking.

In the first project, user activity events come in a big data pipeline
via Kafka; we use Spark to create summary report.  Nested JSON format.

In the second project, we also _produce_ events: the server logs
events to Kafka, and we write a Spark job to produce real-time
(streaming) analytics report.  We use ApacheBench to send mock events.

[Reports coming soon]

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

Page under construction.  More projects on the way!
