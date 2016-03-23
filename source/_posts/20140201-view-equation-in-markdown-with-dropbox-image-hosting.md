title: View math equations in markdown with dropbox image hosting
date: 2014-02-01 19:03
tags:
- math
- markdown
- dropbox
categories:
- blog
---

## Goal
I wanted to show math equations in my github README.md

Update: I now use [MathJax to view math equations in markdown](https://alyssaq.github.io/2015/view-math-equations-in-markdown-with-mathjax). I use dropbox to show images in this blog.

### How to show math equations in the browser
To do this, I was going to use [MathML](http://www.w3.org/Math/mathml-faq.html), which
is an XML-based syntax for mathematical symbols to render in browsers.
Unfortunately,  markdown does *NOT* support MathML
☹

### How to show math equations in markdown
My work around was to convert the equations into images, host them somewhere and embed them in my README.md file.

You will need:

1. An editor that is able to render MathML equations. (I use [mouapp](mouapp.com))   
2. [Dropbox](dropbox.com) or Google Drive to host your images for free. (I'll be using dropbox)

### Steps:

1) Copy the your MathML into your markdown editor (example below):    
2) Take a screenshot of the rendered equation  <br>
   (On Mac, to take an area-selected screenshot: command + shift + 4)    
3) Place it in your dropbox folder and "Share Dropbox Link":    
![image](https://www.dropbox.com/s/z4ng9s65ot0kcws/share-dropbox-link.png?dl=1)

4) Copy the link and add **?dl=1** at the end of the url to make it viewable when embedded in markdown:

	https://www.dropbox.com/s/{guid}/equation1.png?dl=1

5) Add it to your markdown!

	![image](https://www.dropbox.com/s/{guid}/share-dropbox-link.png?dl=1)

To center the image in markdown, you will have to use HTML:

	p align="center"
	  img src="https://www.dropbox.com/s/{guid}/{}.png?dl=1"
	/p

<p align="center">
  <img src="https://www.dropbox.com/s/opu9yg1a6j1zdve/equation1.png?dl=1">
</p>

6) Done! You have hosted your image with dropbox and embedded it
into your README.md for viewing on github
