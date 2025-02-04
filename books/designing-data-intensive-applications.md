<!-- from Designing Data-Intensive Applications by Martin Kleppmann -->

# Chapter 1: Reliable, Scalable, and Maintainable Applications

*Data-intensive* applications are applications that aren't *compute-intensive*, but do have a large volume of potentially complex data which changes often. They are built from:
  - *Databases* to store data for later
  - *Caches* to remember expensive operations and speed up reads
  - *Search indexes* to allow search
  - *Stream processing* to send messages to other processes, which are handled asynchronously
  - *Batch processing* to crunch large amounts of data

Three main concerns for data systems are:
  - *Reliability*, or working correctly even with adversity and error
  - *Scalability*, or the ability to grow
  - *Maintainability*, or the ability for engineers to productively work with a system

## Reliability

*Fault-tolerance* means resilience to certain kinds of faults - but not total system failure. Faults can be triggered directly to test error handling (i.e. Chaos Monkey). 

Redundancy can help prevent hardware faults. AWS prioritizes flexibility and elasticity over single-machine reliability (... citation needed?). There's tons of different types of software faults, obviously. Human (or "operator" failures) cause issues for ISPs (configuration errors from operators are the leading source of faults). They can be obviated by non-prod sandbox environments, well-designed abstractions and APIs, testing, quick configuration roll-backs, monitoring, and good management.

## Scalability

Scalability is a function of *load* - like - requests per second, read/write ratios, simultaneously active users, hit rates on caches, etc.

In the case of Twitter, users post tweets to a global tweet database, and view tweets from people they follow from that database. The native relational database approach doesn't work quite as well when the number of tweets reaches hundreds of billions, so they switched to a mode where each user gets a "home timeline cache" - and when a tweet is posted, it is posted only to their followers' home timeline caches. If a user has 30 million followers, this is an expensive approach. So they moved to a hybrid approach: celebrities are exempted from "home timeline cache fan-out".

Batch processing systems (like Hadoop) use *throughput* as an important performance metric - i.e. records-processed-per-second. Online systems use response time. Median response time is a good metric (*not* mean).

*Elastic* systems can automatically increase compute resources when load increases are detected (auto-scaling).

Distributed *stateless* operations are straightforward, but *stateful* operations are more complicated - so until recently, databases were generally one-node and scaled-up-not-scaled-out.

## Maintainability

Maintainability consists of:

  - *Operatability*, or how easy it is to keep a system running smoothly
  - *Simplicity*, or how easy it is for a new engineer to learn a system
  - *Evolvability*, or how interchangeable the system's parts are (or how easy they are to change)

*"Good operations can often work around the limitations of bad software, but good software cannot run reliability with bad operations".*

Good operations looks like:

  - Monitoring system health and quickly restoring service
  - Tracking down causes of problems like system failures or degraded performance
  - Keeping servers and applications patched and up-to-date
  - Keeping tabs on how systems affect *each other*
  - Capacity planning (and other forms of future problem anticipation)
  - Automated deployments and configuration management
  - Complex maintenance tasks like moving applications across different OSes
  - Maintaining security of systems when configuration changes are made
  - Defining production change processes
  - _Preserving an organization's knowledge about a system even if the good engineers leave_
  - Avoiding dependency on individual machines, so machines can actually be taken down and/or patched
  - Good documentation 
  - No surprises
  - Self-healing, with the ability for administrators to override default behavior
 
# Chapter 2: Data Models and Query Languages

# todo