title: View math equations in markdown with MathJax
date: 2014-02-02 19:00
tags:
- math
- markdown
categories:
- blog
---

[MaxJax](http://docs.mathjax.org/en/latest/mathjax.html) is an open-source JavaScript display engine for LaTeX, MathML, and AsciiMath notation that works in all modern browsers.

1) Add the MathJax script from cdn into your HTML. I use the TeX syntax but you can also use mathML by including it in the config parameter. 

	<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS_HTML"></script>

2) Change the default inline math delimiters of `\(...\)` as the brackets mean something else in markdown.

    <script type="text/x-mathjax-config">
      MathJax.Hub.Config({
        extensions: ["tex2jax.js"],
        jax: ["input/TeX", "output/HTML-CSS"],
        tex2jax: {
          inlineMath: [ ['$','$']],
          displayMath: [ ['$$','$$']],
          processEscapes: true
        },
        "HTML-CSS": { availableFonts: ["TeX"] }
      });
    </script>
So, `$$ y = mx + b $$` appears on a new line:
$$ y = mx + b $$

and `$y= mx + b$` appears inline: $y = mx + b$

3) Enjoy math scripting with TeX!
Here are some [TeX samples](http://www.mathjax.org/demos/tex-samples/)
