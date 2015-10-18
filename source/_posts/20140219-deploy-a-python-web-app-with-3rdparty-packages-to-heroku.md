title: Deploy a Python web app with 3rd-party packages to Heroku
date: 2014-02-19 00:34 
tags: 
- heroku
- python
- numpy
- pandas
- automation
categories: 
- tech
---

## Goal
To deploy a Python web app with various mathematical, scientific, text-processing packages to a cloud hosting [platform-as-a-service](http://en.wikipedia.org/wiki/Platform_as_a_service).

My required 3rd-party Python packages/modules:

 * [Numpy](http://numpy.org) (fundamental package for scientific computing with numbers)
 * [Scipy](http://scipy.org) (scientific computing tools)
 * [Pandas](http://pandas.pydata.org) (high-performance, easy-to-use data structures for data pre-processing and analytics)
 * [Scikit-learn](http://scikit-learn.org) (machine learning, data mining, data analysis) 
 * [Textblob](textblob.readthedocs.org) (simplified text-processing version of [nltk](http://nltk.org))

### What I tried and failed
I spent a few hours attempting to create a boilerplate web application with the required packages to be deployed on Google App Engine. Creating a restful API for GAE was easy ([heres mine on Github](https://github.com/alyssaq/flask-restful-api-appengine) using Flask). However, adding the scientific packages I needed was Hell. 

In summary, GAE does [NOT support a lot of C-based modules](https://developers.google.com/appengine/kb/general#libraries). So, say goodbye to a handful of Python scientific packages which depend on compiling various C-based libraries.   
While GAE does provide numpy, if you want pandas at some imaginary point in the future, you'll have to [vote on their thread](https://code.google.com/p/googleappengine/issues/detail?id=8620). 

Next up, PiCloud. Oh, they joined Dropbox and no longer allow sign-ups. 

No, I do not want to bother with AWS. No, I do not want to merely run analytics up in the cloud with wakari or pythonanywhere.

## Deploy scientific Python with Heroku buildpack
Building numpy, scipy, pandas, etc over and over again with each deployment gets old VERY quick.     
_Solution:_ Use a [buildpack](https://devcenter.heroku.com/articles/buildpacks) to prepare your dyno before your app executes. The custom buildpacks listed below uses the binary distributions of the required packages so this means not having to build numpy with each push to Heroku (joy!). 

Useful Buildpacks:

* [Heroku's python buildpack](https://github.com/heroku/heroku-buildpack-python) - for simple python web apps
* [Above buildpack + base scientific packages](https://github.com/dbrgn/heroku-buildpack-python-sklearn) - for python web apps with numpy, scipy and/or scikit-learn (only Python 2.7.4)
* [Above buildpack + textblob with core nltk corpora](https://github.com/alyssaq/heroku-buildpack-python-sklearn) - this is my fork of the above buildpack with a script to download and cache core nltk corpora for textblob. 

### How to deploy with a Heroku buildpack
1) Create a simple Python web app that imports some scientific packages. As an example, Im using [my bottle-skeleton app](https://github.com/alyssaq/bottle-heroku-skeleton)

2) Clone the repo and add to git:

```bash
$ git clone git@github.com:alyssaq/bottle-heroku-skeleton.git
$ git init
$ git add .
$ git commit -m "init"
```

3) Add any additional dependencies in `requirements.txt` (the skeleton app already includes numpy, textblob and pandas)

4) Login to heroku, set the custom buildpack and deploy!

```bash
$ heroku login
$ heroku config:set BUILDPACK_URL=https://github.com/alyssaq/heroku-buildpack-python-sklearn
$ git push heroku master
```

5) Open your browser and ensure that the home page loads with the `Hi` text. If it does, its all good to go.
