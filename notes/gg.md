**Notes courtesy Sama Ziki**

**gg**

- gg is a general framework for building burst-parallel cloud-functions , by building them on an abstraction of transient, functional containers, or thunks
- Thunk - some unit of execution which takes in input and creates an output
- We execute thunks using serverless functions
- All objects must be passed to the serverless function
- Content Addressable storage system is used to store the object and its hash and the hash is sent to the serverless function
- gg is cheap and faster than more cores or clusters
- gg can run on cores cluster or AWS Lambda

**Backend**

- Storage Engine
  - Simple interface to a content address storage to store objects and their hashes
- Execution Engine
  - Runs in conjunction with the storage engine
  - Execution engine implements a simple abstraction
    - A function that receives a thunk as the input and returns the hashes of its output objects (which can be either values or thunks)
- Coordinator
  - Implements the services offered by the gg such as job scheduling, memorization, failure recovery and straggler mitigation
  - As an optimization for such cases, the user can decide to run the execution engine in the &quot;long-lived&quot; mode, where each Lambda worker stays up until the job finishes and seeks out new thunks to execute
  - There are significant start up and tear down costs to serverless functions show invoking a chain of functions is great
  - Multiple thunks can be bundled as one and sent to the execution engine
- Failure Recovery and Straggler Mitigation
  - Occurs when coordinator fails
  - Allows the job to be picked up where it was left off through use of on disk cache entries to avoid redoing work that has already been done

**Overall**

- 14,000 lines of code
- Shows that bulk of the work is done by Amazon Lambda
- Contributions
  - Software Compilation
    - Systems project that runs on multiple applications which means they took a lot of care to take care of edge cases
    - Uses thunks
  - Unit Testing
  - gg is a distributed system compiler that can compile large code bases with low overhead
- Two lambda functions can directly communicate at a really high speed
  - Bandwidth is higher
  - No slow storage
  - More functionality
