title: Installing OpenCV on Mac OSX with homebrew
date: 2014-08-18 16:24 
tags: 
- mac
- opencv
categories: 
- tech
---

1. Install the package manager [Homebrew](http://brew.sh/). It beats MacPorts any day.
2. Go ahead and install [OpenCV](http://opencv.org/):

		$ brew install opencv

3. If all is well, it should be available at:
    
		/usr/local/Cellar/opencv/2.4.9
    
4. Create a symlink to the compiled OpenCV files. If you don't mind setting up your `PYTHONPATH` and messing up python2 and python3 support, you may use `brew link opencv`. Otherwise, I recommend doing the manual symlink to your Python 2.7 site-packages.

    	$ sudo ln -s /usr/local/Cellar/opencv/2.4.9/lib/python2.7/site-packages/cv.py /Library/Python/2.7/site-packages/cv.py
    	$ sudo ln -s /usr/local/Cellar/opencv/2.4.9/lib/python2.7/site-packages/cv2.so /Library/Python/2.7/site-packages/cv2.so 
    
5. Check that it works!

		$ ipython
		Python 2.7.9
		[GCC 4.2.1 (Apple Inc. build 5666) (dot 3)] on darwin
		Type "help", "copyright", "credits" or "license" ...

		In [1]: import cv2
		In [2]: img = cv2.imread('path-to-image/helloworld.jpg')
		In [3]: cv2.imshow('Test image', img)

To check locations of site-packages folders:

		In [4]: import site; site.getsitepackages()
		Out[4]:
		['/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/site-packages',
 		 '/Library/Frameworks/Python.framework/Versions/2.7/lib/site-python',
 		 '/Library/Python/2.7/site-packages']
