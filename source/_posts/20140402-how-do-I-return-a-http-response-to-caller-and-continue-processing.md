title: How do I return a HTTP response to caller and continue processing
date: 2014-04-02 13:33 
tags: 
- multithreading
- api
- queue
- python
categories: 
- tech
---

## Scenario
I have an analysis service that is invoked by posting data via its API.   
Since the processing could be long-running, it is best to return a response to the caller and continue with the data.  

## Goal
Return a 200 HTTP response to the caller upon receiving valid data.   
After returning the response, continue processing the data and post the results to another endpoint.

## Potential solutions
* Multithreading
* Multiprocessing
* Message Queues such as [celery](http://www.celeryproject.org/), [python-rq](http://python-rq.org/) or [ironMQ](http://www.iron.io/mq)

## Python multithreading
Between [multithreading and multiprocessing](#multithreading_multiprocessing), using threads was sufficient for my requirements. I ended up using Python's [threading](https://docs.python.org/2/library/threading.html) library.

Below is the [code gist](https://gist.github.com/alyssaq/9928442) to handle POST requests and continue processing with threads.
Tested with `python 2.7` and `python 3.3` 

<pre><code class="language-python">
#!/usr/bin/env python
# -*- coding: utf-8 -*-

""" Sample multithreading with bottle.py
Requirements: requests, bottle

To run: 
$ python app.py

To post data, open another command shell and type:
$ curl -X POST -H "Content-Type: application/json" -d '{"data":"test"}' http://127.0.0.1:8080/process -D-

"""
import bottle
import json
import requests
from threading import Thread

POSTBACK_URL = 'http://127.0.0.1:8080/postprocessed'

@bottle.post('/postprocessed')
def postprocessed_handle():
  print('Received data at postprocessed')
  return bottle.HTTPResponse(status=200, body="Complete postprocessed")

def process_data(data):
  # do processing...
  result = json.dumps(data)

  print('Finished processing')
  requests.post(POSTBACK_URL, data=result, 
    headers={'Content-Type':'application/json'})
               
@bottle.post('/process')
def process_handle():
  data = bottle.request.json.get('data', False)
  print('Received data at process: ' + data)

  if not data:
    return bottle.HTTPResponse(status=400, body="Missing data")

  #Spawn thread to process the data
  t = Thread(target=process_data, args=(data, ))
  t.start()

  #Immediately return a 200 response to the caller
  return bottle.HTTPResponse(status=200, body="Complete process")

if __name__ == '__main__':
  bottle.run()
</code></pre>

## Terminologies
### <a name="multithreading_multiprocessing"></a> multithreading vs multiprocessing
* A thread is the smallest sequence of instructions within a process
* A process is an instance of a computer program that is being executed.

| Threads        | Processes           | 
| ------------- |-------------| 
| Share address space      | Separate address space | 
| Share process state, memory <br>and other resources      | Carries more state information      | 
| thread lifetime, context switching and <br>synchronisation costs are lower |  slower context switching between <br>processes than threads 
| Exist as subsets of a process      | Typically independent. Insulated <br>from each other by the OS      | 
| An error in a thread can kill all the <br> other threads in the same process       | An error in a process <br>may not affect another process      |