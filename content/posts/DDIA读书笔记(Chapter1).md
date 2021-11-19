---
title: "DDIA读书笔记(Chapter1)"
date: 2021-11-02T23:59:17+08:00
draft: false
original: true
categories: 
  - 书籍
tags: 
  - 读书笔记
---

# Reliable, Scalable, and Maintainable Applications

many applications commonly need functionality:

- Store data so that they, or another application, can find it again later (*databases*)
- Remember the result of an expensive operation, to speed up reads (*caches*)
- Allow users to search data by keyword or filter it in various ways (*search indexes*)
- Send a message to another process, to be handled asynchronously (*stream pro‐
cessing*)
- Periodically crunch a large amount of accumulated data (*batch processing*)

It  painfully obvious, but different applications have different requirements

<!--more-->

**Data Systems**

- no longer neatly fit into traditional categories
    - Redis as message queues
    - Kafka as database
- single tool can no longer meet all of its data processing and storage needs

different tools are stitched together using application code.

![](/DDIA/figure1-1.png)

You are now not only an application developer, but also a data system designer.

If you are designing a data system or service, a lot of tricky questions arise.

- insure that the data remains correct and complete
- provide consistently good performance
- scale to handle an increase in load
- good API for the service

many factors that may influence the design of a data system

- skills and experience of the people involved
- legacy system dependencies
- the time‐scale for delivery
- tolerance of different kinds of risk,regulatory constraints
- etc

## **Reliability**

typical expectations include:

- The application performs the function that the user expected.
- It can tolerate the user making mistakes or using the software in unexpected
ways.
- Its performance is good enough for the required use case, under the expected
load and data volume.
- The system prevents any unauthorized access and abuse.

roughly meaning

> continuing to work correctly, even when things go wrong.

misleading

> it suggests that we could make a system tolerant of every possible kind of fault

that is not feasible, makes sense to say:

> tolerating *certain types* of faults.

a fault is not the same as a failure

- A *fault* is usually defined as one component of the system deviating from its spec
- a *failure* is when the system as a whole stops providing the required service to the user

it is usually best to design fault-tolerance mechanisms that prevent faults from causing failures.

*increase* the rate of faults by triggering them deliberately — The Netflix *Chaos Monkey*

### **Hardware Faults**

- Hard disks crash
- RAM becomes faulty
- the power grid has a blackout
- unplugs the wrong network cable

add redundancy to the individual hardware components, but multi-machine redundancy was only
required by a small number of applications for which high availability was absolutely
essential.

as data volumes and applications’ computing demands have increased,more applications have begun using larger numbers of machines,which proportionally increases the rate of hardware faults.Hence there is a move toward systems that can tolerate the loss of entire machines, by
using software fault-tolerance techniques in preference or in addition to hardware
redundancy.

### **Software Errors**

- A software bug that causes every instance of an application server to crash when
given a particular bad input.
- A runaway process that uses up some shared resource
- A service that the system depends on that slows down, becomes unresponsive, or
starts returning corrupted responses.
- Cascading failures, where a small fault in one component triggers a fault in
another component, which in turn triggers further faults

There is no quick solution to the problem of systematic faults in software.Lots of
small things can help:

- carefully thinking about assumptions and interactions
- thorough testing
- process isolation
- allowing processes to crash and restart
- measuring, monitoring, and analyzing system behavior in production

### **Human Errors**

humans are known to be unreliable

How do we make our systems reliable, in spite of unreliable humans? The best sys‐
tems combine several approaches:

- Design systems in a way that minimizes opportunities for error
- Decouple the places where people make the most mistakes from the places where
they can cause failures.
- Test thoroughly at all levels, from unit tests to whole-system integration tests and
manual tests
- Allow quick and easy recovery from human errors, to minimize the impact in the
case of a failure.
- Set up detailed and clear monitoring, such as performance metrics and error
rates. In other engineering disciplines this is referred to as *telemetry*.
- Implement good management practices and training

## Scalability

*Scalability* is the term we use to describe a system’s ability to cope with increased load.

### Describing Load

The best choice of *load parameters* depends on the architecture of your system

- may be requests per second
- the ratio of reads to writes in a database
- the number of simultaneously active users
- the hit rate on a cache
- something else

let’s consider Twitter as an example.Two of Twitter’s main operations are:

- *Post tweet:* A user can publish a new message to their followers (4.6k requests/sec on aver‐
age, over 12k requests/sec at peak).
- *Home timeline:* A user can view tweets posted by the people they follow (300k requests/sec).

Twitter’s scaling challenge is not primarily due to tweet volume,but due to *fan-out.*There are broadly two ways of implementing these two operations:

1. inserts the new tweet into a global collection of tweets,look up all the people they follow,find all the tweets for each of those users,and merge them

    ```sql
    SELECT tweets.*, users.* FROM tweets
    JOIN users ON tweets.sender_id = users.id JOIN follows ON follows.followee_id = users.id WHERE follows.follower_id = current_user
    ```

2. Maintain a cache for each user’s home timeline

![](/DDIA/figure1-2.png)

![](/DDIA/figure1-3.png)

最初，twitter采用的是第1种方式，但是home timeline的负载日趋上升，所以他们切换到第2种方式，第2种方式比第1种方式好，因为写比读少2个数量级。这种方式在写的时候做了更多工作，读的时候就很简单。

### **Describing Performance**

- *throughput*
- *response time*
- **Latency**
- *percentiles*
- *service level agreements* (SLAs)
- *tail latency amplification*

calculate percentiles

- [forward decay](http://dimacs.rutgers.edu/~graham/pubs/papers/fwddecay.pdf)
- [t-digest](https://github.com/tdunning/t-digest)
- [HdrHistogram](http://www.hdrhistogram.org/)

### **Approaches for Coping with Load**

- *scaling up*
- *scaling out*
- there is no such thing as a generic, one-size-fits-all scalable architecture
- In an early-stage startup more important to iterate quickly on product features than it is to scale

## **Maintainability**

the majority of the cost of software is not in its initial development, but in its ongoing maintenance

- fixing bugs
- keeping its systems operational
- investigating failures
- adapting it to new platforms
- modifying it for new use cases
- repaying technical debt
- adding new features

many people dislike maintenance of so-called *legacy* systems.it is difficult to give general recommendations for dealing with them.

design principles for software systems:

- *Operability:* Make it easy for operations teams to keep the system running smoothly.
- *Simplicity:* Make it easy for new engineers to understand the system, by removing as much
complexity as possible from the system. (Note this is not the same as simplicity
of the user interface.)
- *Evolvability:* Make it easy for engineers to make changes to the system in the future, adapting
it for unanticipated use cases as requirements change. Also known as *extensibil‐
ity*, *modifiability*, or *plasticity*.

### **Operability: Making Life Easy for Operations**

A good operations team typically is responsible for the following:

- Monitoring the health of the system and quickly restoring service if it goes into a
bad state
- Tracking down the cause of problems, such as system failures or degraded per‐
formance
- Keeping software and platforms up to date, including security patches
- Keeping tabs on how different systems affect each other, so that a problematic
change can be avoided before it causes damage
- Anticipating future problems and solving them before they occur (e.g., capacity
planning)
- Establishing good practices and tools for deployment, configuration manage‐
ment, and more
- Performing complex maintenance tasks, such as moving an application from one
platform to another
- Maintaining the security of the system as configuration changes are made
- Defining processes that make operations predictable and help keep the produc‐
tion environment stable
- Preserving the organization’s knowledge about the system, even as individual
people come and go

Data systems can do various things to make routine tasks easy:

- Providing visibility into the runtime behavior and internals of the system, with
good monitoring
- Providing good support for automation and integration with standard tools
- Avoiding dependency on individual machines (allowing machines to be taken
down for maintenance while the system as a whole continues running uninter‐
rupted)
- Providing good documentation and an easy-to-understand operational model
(“If I do X, Y will happen”)
- Providing good default behavior, but also giving administrators the freedom to
override defaults when needed
- Self-healing where appropriate, but also giving administrators manual control
over the system state when needed
- Exhibiting predictable behavior, minimizing surprises

### **Simplicity: Managing Complexity**

various possible symptoms of complexity:

- explosion of the state space
- tight coupling of modules
- tangled dependencies
- inconsistent naming and terminology
- hacks aimed at solving performance problems
- special-casing to work around issues elsewhere
- and many more

Making a system simpler does not necessarily mean reducing its functionality

One of the best tools we have for removing accidental complexity is *abstraction.*

### **Evolvability: Making Change Easy**

- *Agile*
- refactor
- simplicity & abstractions

## **Summary**

*Reliability* means making systems work correctly, even when faults occur.

*Scalability* means having strategies for keeping performance good, even when load
increases.

*Maintainability* has many facets, but in essence it’s about making life better for the
engineering and operations teams who need to work with the system.