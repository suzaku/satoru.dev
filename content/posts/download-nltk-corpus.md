+++
title = "How to manually download a nltk corpus?"
date = 2020-02-20T17:22:59+08:00
draft = false
tags = ["programming", "Python", "NLP"]
categories = []
+++

1. Go to http://www.nltk.org/nltk_data/ and download whichever data file you want
1. Now in a Python shell check the value of `nltk.data.path`
1. Choose one of the path that exists on your machine, and unzip the data files into the `corpora` subdirectory inside.
1. Now you can import the data `from nltk.corpus import stopwords`
