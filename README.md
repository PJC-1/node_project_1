
Advanced Node Concepts
===================
> Learning about Node.js

Threads
-------------
>*What are threads?*
>
>When ever you run **programs** on a computer, something called a **process** is started up.
>
>A **process** is an instance of a computer program that is being executed.
>
>Within a single process, we can have **multiple** things called **threads**.
>
>You can think of a **thread** as a type of **to-do list**, that has some number of instructions that need to be executed by the **cpu** of you computer.
>
>The **thread** is given to the **cpu**, and the **cpu** will attempt to **run** every instruction on it, **one by one**.
>
>A **single process** can have **multiple threads** inside of it.
>
>An important aspect of **threads** are **scheduling**, which refers to you **operating system's** ability to decide which **thread** to process at any given instance in time.
>
>You need to remember that your computer has a *limited* amount of resources available to it and your **cpu** can only process so many *instructions* per *second*.
>
>This starts to become very relevant when we start to get many active **processes** and **threads** on our computer.
>
>The **operating systems scheduler** has to look at the different **threads** that are asking to be processed, and figure out how to do some amount of work on each of them while making sure that they don't have to wait too long to be processed.
>
>We want to make sure that *urgent threads* don't have to wait too long to be executed.
>
>There are a couple different strategies that are used to improve the rate at which these threads can be processed.
>
>The first being: *Adding more CPU Cores to our machine.*, if we have more than one core inside of our  **CPU** then we can easily process multiple threads at the same time.
>
>The second is: *Closely examine the work that is being done by each thread and allow our operating system scheduler to detect big pauses in processing time due to expensive input and output operations*
>
>*what is the event loop?*
>
>When we start up a node program on our computer, node automatically creates one **thread** and then executes all of our code *inside* of that one single thread.
>
>Inside that single thread is something called the **event loop**, you can think of the **event loop** as being like a *control structure* that decides what our one thread should be doing at any given point in time.
>
>This event loop is the absolute core of every program that you and I run and every program that you and I run has exactly **one** event loop.
>
>Understanding how the event loop works is *extremely* important because a lot of performance concerns about node eventually boil down to how the event loop behaves.
>
>**fake code example to illustrate the event loop process**
>
>```
>// node myFile.js
>
>const pendingTimers = [];
>const pendingOSTasks = [];
>const pendingOperations = [];
>
>// New timers, tasks, operations are recorded from myFile running
>myFile.runContents();
>
>function shouldContinue() {
>  // Check one: Any pending setTimeout, setInterval, setImmediate?
>  // Check two: Any pending OS tasks? (Like server listening to port)
>  // Check three: Any pending long running operations? (Like fs module)
>  return pendingTimers.length || pendingOSTasks.length || pendingOperations;
>}
>
>// Entire body executes in one 'tick'
>while(shouldContinue()) {
>  // 1) Node looks at pendingTimers and sees if any functions
>  // are ready to be called. setTimeout, setInterval
>
>  // 2) Node looks at pendingOSTasks and pendingOperations
>  // and calls relevant callbacks
>
>  // 3) Pause execution. Continue when...
>  //  - a new pendingOSTasks is done
>  //  - a new pendingOperation is done
>  //  - a timer is about to complete
>
>  // 4) Look at pendingTimers. Call any setImmediate
>
>  // 5) Handle any 'close' events
>}
>
>// exit back to terminal
>
>```
>
>*is the event loop single threaded?*
>
>The node *event loop* is single threaded, but some of the functions that included inside of the node *standard library* are run outside of the event, and outside of that single thread (not single threaded).
>
>Basically, the event loop uses a single thread but a lot of the code that you and I write does not actually execute inside that thread entirely.
>
>**Questions about threadpools?**
>
>Q: *Can we use the threadpool for javascript code or can only nodeJS functions use it?*
>A: We can write custom JS that uses the thread pool.
>
>Q: *What functions in node std library use the threadpool?*
>A: All 'fs' module functions. Some crypto stuff. Depends on OS (windows vs unix based).
>
>Q: *How does this threadpool stuff fit into the event loop?*
>A: Tasks running in the threadpool are the 'pendingOperations' in our code example.
>
>**Questions about OS Async features?**
>
>Q: *What functions in node std library use the OS's async features?*
>A:  Almost everything around networking for all OS's. Some other stuff is OS specific.
>
>Q: *How does this OS async stuff fit into the event loop?*
>A: Tasks using the underlying OS are reflected in our 'pendingOSTasks' array.
>
>**Interesting Threadpool Example**
>
>**Example Code**:
>```
>const https = require('https');
>const crypto = require('crypto');
>const fs = require('fs');
>
>const start = Date.now();
>
>function doRequest() {
>  https
>  .request('https://www.google.com', res => {
>    res.on('data', () => {});
>    res.on('end', () => {
>      console.log(Date.now() - start);
>    });
>  })
>  .end();
>}
>
>function doHash() {
>  crypto.pbkdf2('a', 'b', 100000, 512, 'sha512', () => {
>    console.log('Hash:', Date.now() - start);
>  });
>}
>
>doRequest();
>
>fs.readFile('multitask.js', 'utf8', () => {
>  console.log('FS:', Date.now() - start);
>});
>
>doHash();
>doHash();
>doHash();
>doHash();
>
>```
>
>**Example Output**:
>```
>$ node multitask.js
>311
>Hash: 1877
>FS: 1878
>Hash: 1888
>Hash: 1891
>Hash: 1901
>```
>
>**Output Explination**:
>- First we see the benchmark from the ```http``` module.
>- Then we see one console log from the hashing function.
>- After we see the ```file system``` module call.
>- Then we see the ```3``` remaining hashing function calls.
>
>- There is no way that reading a file off of the hard drive can possibly take ```2``` seconds.
>- If you comment out all of the hashing function calls and run the file.
>**Example Code**:
>```
>const https = require('https');
>const crypto = require('crypto');
>const fs = require('fs');
>
>const start = Date.now();
>
>function doRequest() {
>  https
>  .request('https://www.google.com', res => {
>    res.on('data', () => {});
>    res.on('end', () => {
>      console.log(Date.now() - start);
>    });
>  })
>  .end();
>}
>
>function doHash() {
>  crypto.pbkdf2('a', 'b', 100000, 512, 'sha512', () => {
>    console.log('Hash:', Date.now() - start);
>  });
>}
>
>doRequest();
>
>fs.readFile('multitask.js', 'utf8', () => {
>  console.log('FS:', Date.now() - start);
>});
>```
>
>**Example Output**:
>```
>$ node multitask.js
>FS: 19
>185
>```
>
>- It takes ```19``` milliseconds to complete reading off the harddrive, this means that we are seeing some very intersting behavior in the implementation with the hashing function calls since it's taking ```~2``` seconds to complete the ```file system``` read method.
>
>**Why do we always see exactly one hash console log before the result of the file system?**
>
>- Once all the function calls (```fs.readFile``` and the ```4``` ```doHash``` function calls) are properly allocated to the first ```4``` threads in the **thread pool**.
>- When ```fs``` module call is loaded into thread ```1```, thread ```1``` started to go through the process of the ```fs.readFile``` method, where node goes out to get some information about the file, accesses the hard drive and returns the information, then node goes out and accesses the hard drive again to stream the file contents back to the application, where node returns the file contents to us.
>- During that first *phase* where ```thread 1``` goes out to the hard drive to get some information about the specified file, the *thread* will basically move on to the next transaction in line (```pbkdf2``` call number ```4```). So ```thread 1``` temporarily forgets about that file system call and starts calculating the hash.
>- ```thread 2``` will complete and be ready to accept more work, it will see that there's still a pending **file system** call that needs to be worked on. ```thread 2``` will look to see if it has gotten any information back from the hard drive. That information/statistics come back into ```thread 2``` and then continues to wrk on the **file system** call. Makes another follow up request to the hard drive to get the actual file contents. ```thread 2``` processes them and we then see that console log appear.
>- This is why we alway see one hash get completed before the file system module call.
>
>**Why do we always see the HTTP request complete first?**
>
>- *Note*, both the ```http``` request and the ```file system``` call are both **asynchronous**, it some amount of time for both of them to complete.
>- Node makes use of a **thread pool** for some very specific function calls. In particular, almost everything inside of the ```fs``` module makes use of this **thread pool**.
>- The crypto module function ```pbkdf2``` also makes use of the **thread pool** as well.
>-  However, the ```https``` module does not use the *thread pool*, instead it reaches out directly to the ```operating system``` and leverages the operating system to do all that networking work for us.
>- If we look at the times it took to complete the different operations, we see that the ```https``` call resolved right away, but we had to wait much longer for all the other function calls for some reason.
>

Performance
-------------
>*Improving Node Performance*
>- Use Node in ```Cluster Mode```.
>- Use ```Worker Threads```.
>
>*Note*:
>- It is **Recommended** to *Use Node in 'Cluster' Mode*, for improving performance of your application.
>- It is considered **Experimental** to *Use Worker Threads*.
>
>**Clustering**
>
>**Cluster Manager** is responsible for monitoring the health of individual instances of our application that we're going to launch at the same time on our computer. This is regarding multiple instances on ```one``` computer. The *cluster manager* itself doesn't actually execute any application code. The *cluster manager* isn't really responsible for handling incoming requests or fetching data from the database or doing anything like that. Instead, the *cluster manager* is responsible for monitoring the health of each of the individual instances.
>
>The *Cluster Manager* can:
>- Start instances.
>- Stop an instance.
>- Restart an instance.
>- Send an instance data.
>- Do other kind of administrative tasks.
>
>It is up to the individual instances of the server to actually process incoming requests and do things such as access the database, handle authentication, or serve up static files.
>
>**Worker Instances**
>
>*Worker Instances* are actually responsible for processing incoming requests.
>
>To create **worker instances** the *cluster manager* is going to require in the *cluster module* from the node *standard library*. There is one particular function on the **cluster module** called ```fork()``` and whenever we call that ```fork``` function from within the *cluster manager* node internally goes back to our index.js file and it executes it a second time, but it executes it that second time in a slightly different mode. Basically the index.js file is being executed multiple times by node. The very first time it's going to *produce* the **cluster manager** and then every time after that it's going to be producing our **worker instances**.
>
>**Use-case where Clustering in Node can be helpful?**
>
>If you have some routes inside of your app that usually take a while to process, but you have other routes that are very quick then by using *clustering* you can start up multiple instances of your server that more evenly address all the incoming *requests* that are coming into your application and have more predictable response times.
>
>**Where Clustering does NOT work out so well**
>
>Using ```ab``` (*Apache Benchmark*) to measure performance.
>
>The following command will test a single request to ```localhost:3000/```.
>```
>$ ab -c 1 -n 1 localhost:3000/
>```
>
>**ab output**:
>```
>This is ApacheBench, Version 2.3 <$Revision: 1807734 $>
>Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
>Licensed to The Apache Software Foundation, http://www.apache.org/
>Benchmarking localhost (be patient).....done
>
>Server Software:
>Server Hostname:  localhost
>Server Port:  3000
>Document Path:  /
>Document Length:  8 bytes
>Concurrency Level:  1
>Time taken for tests: 0.975 seconds
>Complete requests:  1
>Failed requests:  0
>Total transferred:  206 bytes
>HTML transferred: 8 bytes
>Requests per second:  1.03 [#/sec] (mean)
>Time per request: 974.912 [ms] (mean)
>Time per request: 974.912 [ms] (mean, across all concurrent requests)
>Transfer rate:  0.21 [Kbytes/sec] received
>
>Connection Times (ms)
>min  mean[+/-sd] median max
>Connect:  0  0 0.0  0 0
>Processing: 975  975 0.0  975 975
>Waiting:  974  974 0.0  974 974
>Total:  975  975 0.0  975 975
>```
>
>The following command will make ```2``` requests at the **exact** same time to our ```1``` child inside of our cluster, where that ```1``` child only has ```1``` thread available :
>```
>$ ab -c 2 -n 2 localhost:3000/
>```
>
>**ab output**:
>```
>This is ApacheBench, Version 2.3 <$Revision: 1807734 $>
>Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
>Licensed to The Apache Software Foundation, http://www.apache.org/
>Benchmarking localhost (be patient).....done
>
>Server Software:
>Server Hostname:  localhost
>Server Port:  3000
>Document Path:  /
>Document Length:  8 bytes
>Concurrency Level:  2
>Time taken for tests: 1.918 seconds
>Complete requests:  2
>Failed requests:  0
>Total transferred:  412 bytes
>HTML transferred: 16 bytes
>Requests per second:  1.04 [#/sec] (mean)
>Time per request: 1917.762 [ms] (mean)
>Time per request: 958.881 [ms] (mean, across all concurrent requests)
>Transfer rate:  0.21 [Kbytes/sec] received
>Connection Times (ms)
>min  mean[+/-sd] median max
>Connect:  0  0 0.0  0 0
>Processing: 974 1446 667.4 1918  1918
>Waiting:  974 1445 667.5 1917  1917
>Total:  974 1446 667.4 1918  1918
>
>Percentage of the requests served within a certain time (ms)
>50% 1918
>66% 1918
>75% 1918
>80% 1918
>90% 1918
>95% 1918
>98% 1918
>99% 1918
>100% 1918 (longest request)
>```
>It takes ```~2``` seconds. Both requests are accepted at the same time, but it can only process ```1``` at a time.
>
>Now, we can try the same ```ab``` command, but instead try it will the ```2``` **children** running:
>```
>$ ab -c 2 -n 2 localhost:3000/
>```
>
>**ap output**:
>```
>This is ApacheBench, Version 2.3 <$Revision: 1807734 $>
>Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
>Licensed to The Apache Software Foundation, http://www.apache.org/  
>Benchmarking localhost (be patient).....done
>
>Server Software:
>Server Hostname:  localhost
>Server Port:  3000
>Document Path:  /
>Document Length:  8 bytes
>Concurrency Level:  2
>Time taken for tests: 0.928 seconds
>Complete requests:  2
>Failed requests:  0
>Total transferred:  412 bytes
>HTML transferred: 16 bytes
>Requests per second:  2.16 [#/sec] (mean)
>Time per request: 928.056 [ms] (mean)
>Time per request: 464.028 [ms] (mean, across all concurrent requests)
>Transfer rate:  0.43 [Kbytes/sec] received
>Connection Times (ms)
>min  mean[+/-sd] median max
>Connect:  0  0 0.0  0 0
>Processing: 926  927 1.1  928 928
>Waiting:  925  926 1.3  927 927
>Total:  926  927 1.1  928 928
>Percentage of the requests served within a certain time (ms)
>50%  928
>66%  928
>75%  928
>80%  928
>90%  928
>95%  928
>98%  928
>99%  928
>100%  928 (longest request)
>```
>
>From the output we can see that both requests are nearly processed in parallel, where ```1``` request was processed slightly faster than the other. Each being processed by one of the ```2``` children in the cluster. In this use-case we definitely a performance benefit.
>
>We can now try *increasing* the number **children** in the cluster to ```6``` and make an ```ab``` request of ```6``` concurrent requests to ```localhost:3000/```:
>```
>$ ab -c 6 -n 6 localhost:3000/
>```
>
> **ab output**:
> ```
> This is ApacheBench, Version 2.3 <$Revision: 1807734 $>
>Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
>Licensed to The Apache Software Foundation, http://www.apache.org/
>Benchmarking localhost (be patient).....done
>
>Server Software:
>Server Hostname:  localhost
>Server Port:  3000
>Document Path:  /
>Document Length:  8 bytes
>Concurrency Level:  6
>Time taken for tests: 2.819 seconds
>Complete requests:  6
>Failed requests:  0
>Total transferred:  1236 bytes
>HTML transferred: 48 bytes
>Requests per second:  2.13 [#/sec] (mean)
>Time per request: 2819.378 [ms] (mean)
>Time per request: 469.896 [ms] (mean, across all concurrent requests)
>Transfer rate:  0.43 [Kbytes/sec] received
>Connection Times (ms)
>min  mean[+/-sd] median max
>Connect:  0  0 0.0  0 0
>Processing:  2784 2804  14.9 2811  2819
>Waiting: 2782 2802  14.6 2810  2817
>Total: 2784 2804  14.9 2811  2819
>Percentage of the requests served within a certain time (ms)
>50% 2811
>66% 2811
>75% 2819
>80% 2819
>90% 2819
>95% 2819
>98% 2819
>99% 2819
>100% 2819 (longest request)
> ```
>  
> When we **increase** the number of children in the cluster to ```6```, we see that each request took ```~3``` seconds to complete.
>
>Depending on laptop/desktop being used, there is an absolute upper limit to the computer's ability to process incoming requests and do some amount of work. So when we run our code and we do ```6``` at the same time, that means that in those ```6``` separate **threads** that are running ```6``` separate **children** we are bouncing between every hash function called at the exact same time and the **CPU** is trying to do a little bit of work on all the different requests all at the same time.
>
>The result is that the code was **not** executed six times faster, instead the result is that it took significantly longer to eventually get a response. The overall performance suffered because the **CPU** was trying to bounce around and process all the incoming requests at exactly the same time.
>
>
>Let now try bring down the number of children to ```2``` and execute the ```ab``` benchmark with ```6``` concurrent tests:
>
>**Example Command**:
>```
>$ ab -c 6 -n 6 localhost:3000/
>```
>
>**Example Output**:
>```
>This is ApacheBench, Version 2.3 <$Revision: 1807734 $>
>Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
>Licensed to The Apache Software Foundation, http://www.apache.org/
>Benchmarking localhost (be patient).....done
>
>Server Software:
>Server Hostname:  localhost
>Server Port:  3000
>Document Path:  /
>Document Length:  8 bytes
>Concurrency Level:  6
>Time taken for tests: 2.910 seconds
>Complete requests:  6
>Failed requests:  0
>Total transferred:  1236 bytes
>HTML transferred: 48 bytes
>Requests per second:  1.99 [#/sec] (mean)
>Time per request: 2900.743 [ms] (mean)
>Time per request: 501.624 [ms] (mean, across all concurrent requests)
>Transfer rate:  0.40 [Kbytes/sec] received
>Connection Times (ms)
>min  mean[+/-sd] median max
>Connect:  0  0 0.1  0 0
>Processing: 999 1999 891.1 2004  2900
>Waiting:  997 1999 891.8 2004  2900
>Total: 1000 2000 891.0 2005  2900
>Percentage of the requests served within a certain time (ms)
>50% 2005
>66% 2005
>75% 2900
>80% 2900
>90% 2900
>95% 2900
>98% 2900
>99% 2900
>100% 2900 (longest request)
>```
>
>You will notice that the *longest* request took ```~3``` seconds, which is basically the same as when we had ```6``` children in the cluster, but the shortest request was almost a whole second shorter.
>
>Generally you will want to match your number of children in your cluster to either the number of **physical cores** or **logical cores** that you have.
>


Node.js Production Process Manager
-------------
>
>[Unitech pm2 github](https://github.com/Unitech/pm2)
>
>[pm2 official website](https://pm2.io/doc/en/runtime/overview/)
>


Caching and MongoDB
-------------
>
>Caching is a very easy way to dramatically improve the read performance of an express application.
>
>Inside MongoDB, there is an **index**, which is an efficient data structure for looking up sets of records inside of that collection.
>
>A **full collection scan** is a collection scan where we have to look at every single record inside of a given collection. Which is an **extremely** expensive operation.
>
>**Cache Server**
>
>Any time *mongoose* issues a query it's going to first go over to the **cache server**. The **cache server** is going to check to see if that exact query has ever been issued before, if it hasn't then the server will take query and send it over to *mongo DB* and *mongo* will execute the query. The results of that query will go back to the *cache server*, the server will **store** the result of that query.
>
>The **cache server** will now know that any time it receives that query it will get that stored response.
>
>It's going to maintain a record between queries that are issued and responses that come back from those queries.
>
>The **cache server** will then take that response and send it back to *mongoose*, where *mongoose* will give that data to *express* and it eventually ends up inside of our application.
>
>Now, anytime that same exact query is issued again *mongoose* is going to send the same query over to the *cache server*, but this time if the cache server sees that the query has already been issued once  before it's not going to send the query onto *mongo DB*. Instead it's going to take the response to that query that it got the last time and immediately send it back over to *mongoose*.
>
>So there's no indices here or full table scan being done. Nothing is being sent to *mongo*.
>
>On the **cache server** we can imagine that there might be something similar to a data store inside there.
>
>It might be a key value *data store* where all the keys are some type of query that has already been issued before and the value might be the result of that query.
>
>An example would be the *key* are the *IDs* of the *records* that we've already looked up before. So imagine that *mongoose* issues a new query where ti tries to find a blog post with an *ID* of ```'123'```.  This query is going to flow into the *cache server*, where the *cache server* is going to check to see if it has a result for any query that was looking for an *ID* of ```'123'```. And if it does not exists, this query is then taken and sent on to the *mongo DB* instance. *Mongo DB* will execute this query, get a response and send it back.
>
>This result is sent back over to the *cache server*, the *cache server* takes the result and immediately sends it back to mongoose.
>
>Immediately right after that, the *cache server* will also take the **query** that was issued, and add that onto it's collection of queries that have been issued. And then it's going to take the results of that query and store it against that query.
>
>The caching layer is only used for reading data. Writing data is going to always go over to the *mongo DB* instance. We need to make sure that anytime we write some amount of data we clear any data stored on the *cache server* that is related to the record that we just wrote or updated.
>

**Redis**
>
>Our caching server is going to be an instance of something called **Redis**.
>
>**Redis** is an **in-memory** data store.
>
>You can think of it as essentially a tiny *database* that runs in the *memory* of your machine and allows you to *read* and *write* data very quickly.
>
>**Redis** is a *data store* that operates only in *memory* and so that means that once it gets *turned off*, *restarted*, or anything like that all the data that sits inside of there is instantly **deleted** and **wiped**.
>
>So, in practive we only use **redis** for data that we feel kind of **OK** having suddenly *disappear* into thin air because a user can lose power or the users machine can be restarted.
>
>**Redis** is *very fast* for reading and writing data. All of these qualities come together to make it fantastic decision for handling something like a caching layer, where we want to make sure that it's very quick for reading data.
>
>To interact with this *redis server* we're going to be running on our local machines, we're going to use a library called ```redis``` (**npm package**), which is the *node* implementation of the **redis** library.
>
>Use ```brew install redis``` if you currently have **brew** already installed on your machine.
>
>Run ```brew services start redis``` to launch redis. You will get the following output in the terminal:
```==>  Successfully started `redis` (label: homebrew.mxcl.redis)```
>
>To confirm that *redis* is working, you can run:
```redis-cli ping```
You will receive the following response:
```
$ redis-cli ping
PONG
```
>
>
>
