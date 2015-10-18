title: Publish your node module to NPM
date: 2014-01-23 18:59 
tags: 
- nodejs
- npm
categories: 
- tech
---

## Goal:
You have a node module and want in on npm registry for others to use by simply running: 

	npm install <your_module_name>

## Prerequisites
1. Install [nodejs](http://nodejs.org)
2. At least the following 3 files:
	- package.json (run `npm init` to create one and follow the prompts)
	- README.md
	- < module >.js
	- < module-test >.js (optional but recommended. [mocha](http://visionmedia.github.io/mocha/) or [jasmine](http://pivotal.github.io/jasmine/) is good)

## Steps:

1) **Register yourself on npm or login**   
You can do this via the [npmjs](https://npmjs.org/signup) site or running

	npm add-user

and providing a username & email.

2) **Publish your node module to npm**      

	npm publish <your_module_name>

**Bam!** You're done.    
Note: It will fail if the node module name is already taken.

## Other housekeeping
- - - 
### Update your published node module 
Increment the version number in your `package.json`. Run

	npm publish 

Your node module is now updated!

### Uninstall node module

	npm uninstall <module_name>

### Example
My mini stats node module: [github.com/alyssaq/stats-analysis](http://github.com/alyssaq/stats-analysis)