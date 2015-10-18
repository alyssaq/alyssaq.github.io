title: How to run SlimerJS and PhantomJS with CircleCI
date: 2014-05-25 17:56 
tags: 
- javascript
categories: 
- tech
---

## Goal
Automatically run user-driven tests (BDD) with [SlimerJS](http://slimerjs.org/) for testing on Gecko (firefox) and [PhantomJS](http://phantomjs.org/) for WebKit (chrome/safari) on the continuous integration platform, [CircleCI](https://circleci.com/).

### Auto testing on CircleCI

[CircleCI's test environment](https://circleci.com/docs/environment#browsers) is preinstalled with CasperJS and PhantomJS but does not have SlimerJS installed. To get started, all you need is a [`circle.yml`](https://circleci.com/docs/configuration) file in your repository, activate it in your CircleCI account and it'll automatically run `npm test` if you're using NodeJS. I also use it for my Python applications.

Running unit tests and just PhantomJS tests is simple and documented on their site but getting SlimerJS working took a little more Googling and trial-and-error. Heres how I set it up.

### Custom commands to use SlimerJS on CircleCI

Firstly, in the `circle.yml` file, specify custom commands  to download the SlimerJS standalone installer (currently at 0.9.1). Extract the files and move them to a known directory. Moving the files to a known directly is the key! 


    test:
      pre:
        - curl -k -L -o slimerjs.tar.bz2 http://download.slimerjs.org/v0.9/0.9.1/slimerjs-0.9.1-linux-x86_64.tar.bz2
        - tar -jxf slimerjs.tar.bz2
        - mv slimerjs-0.9.1 $HOME/slimerjs

I do this in the `test:pre` section to have SlimerJS available before running the tests.

Next, add the required environment variables to CircleCI.

    machine:
      environment:
        SLIMERJSLAUNCHER: $(which firefox)
        PATH: $PATH:$HOME/slimerjs

Just ensure that the `PATH` environment contains the same directory as where you extracted the SlimerJS files.

Heres my full `circle.yml` file with auto-deployment to Heroku and a web hook to my circleCI 2 slack herokuapp with names removed.

    machine:
          environment:
            SLIMERJSLAUNCHER: $(which firefox)
            PATH: $PATH:$HOME/slimerjs
    test:
      pre:
        - bower install
        - curl -k -L -o slimerjs.tar.bz2 http://download.slimerjs.org/v0.9/0.9.1/slimerjs-0.9.1-linux-x86_64.tar.bz2
        - tar -jxf slimerjs.tar.bz2
        - mv slimerjs-0.9.1 $HOME/slimerjs
    deployment:
      production:
        branch: master
        heroku:
          appname: my-production-herokuapp
      staging:
        branch: develop
        heroku:
          appname: my-staging-herokuapp
    notify:
      webhooks:
        - url: https://my-circleci2slack.herokuapp.com/

If all is good, you should see green lights in all the rows in circleCI:
![image](https://alyssaq.github.io/blog/images/SlimerJS-circleCI.png)