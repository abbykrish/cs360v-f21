* Serverless
  * Lets say you have a function (that talks to a server-side
  database) that needs to be triggered every time a user visits a web
  page. 
  * The normal way to do this is to run a web server on the server side,
 use a framework like Ruby on Rails to implement your logic, and
 present the results in a web page to the user
 * This requires that:
     * You need a physical machine (or an EC2-like instance) to run your
     web server on
     * You need to run the whole operating system + web server stack
     on this machine
     * If the traffic increases, you might have to do the whole thing
     on another machine
* Problems:
    * Servers are underutilized
    * Hard to scale
    * Managing an entire OS for a service is hard
* Now, containers help with this problem by only requiring developers
to create containers instead of full VMs, but the other problems remain:
     * You need to run/rent a cluster to run your containers on
     * If all you wanted was to run a single function in response to a
     user request, it still seems like overkill
* Serverless is the next stop along the road to making all of this
more efficient and convenient
* With Serverless,
    * You focus on the application logic
    * You don’t care which server or instance it is deployed on (it is
    invisible to you) 
    * The infrastructure automatically scales for you, running as many
      instances of your function as required
    * Running a function could *potentially* be faster than
    instantiating and running a new container
    * You only pay for the time the function is executing, not the
    time the stack is sitting around un-utilized 
* Each Serveless function is triggered by an event that you configure
* Key difference between different paradigms:
  * Traditional: manage servers + server applications
  * VMs: manage VMs + server applications + physical/virtual cluster
  (optional) 
  * Containers: manage containers + server applications (inside
  containers) + physical/virtual cluster (optional) 
  * Serverless: manage functions (nothing else, not even server
  applications) 
* Serverless provided by all major providers now: Amazon web services,
Microsoft Azure, IBM OpenWhisk, Google Cloud Platform
* Amazon Serverless is called Lambda
  * Amazon free tier provides 1 million lambda requests per month!
* Not all applications fit the Serverless Paradigm:
  * The function that is triggered cannot have its own state (it may
  be running on a different machine or instance each time). 
  * All its state needs to be external. This is the same problem that
  containers face, but more extreme
  * If a function is invoked twice in the same container process, the
  second invocation has access to state left behind by the first. Not
  so for serverless functions
  * If your function is long-running and frequently triggered, it
  would actually be more cost-effective just to provision an always-on
  server on it
  * Multiple copies of the function needs to be able to run in
  parallel (more on this below)
* Good example of Serverless function: authentication
  * Send username + key to backend, authenticate, get back response
  * If this is triggered at a low frequency, say once an hour, doesn’t
  make sense to have a server run just on authentication
* Why don’t we simply run multiple services in a single server instance?
  * We are coupling together different services on a single hardware
  instance 
  * This makes it harder to scale
  * For example, if you have service A and service B on a machine, and
  service B alone gets a lot more traffic, do you now replicate both A
  and B onto a new machine? 
* The client application or website has to be modified to use serverless
  * It is now tracking the session and the requests sent out to various serverless services
  * The client application thus becomes more complex
* How is Serverless implemented on the backend?
  * Using containers!
  * The infrastructure provider creates and runs containers
* Developers have to write serverless functions carefully:
  * Have to assume multiple instances of the function are running
  * The infrastructure may do this if there is demand
  * For example, code that acquires a big lock each time the function
  is run will be extremely bad for this architecture
* API Gateway
  * This allows a function to be triggered in response to an incoming
  HTTP request
* Function startup times on AWS Lambda:
    *10-100 ms for a Python or Javascript function
    *>100 ms (sometimes several seconds) for a Java function (need to start
 the JVM)

* Reading
  * [Serverless Framework](https://serverless.com/framework/docs/)
  * [Serverless Architectures](https://martinfowler.com/articles/serverless.html)

