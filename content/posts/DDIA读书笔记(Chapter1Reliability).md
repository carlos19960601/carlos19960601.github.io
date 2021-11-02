---
title: "DDIA读书笔记(Chapter1Reliability)"
date: 2021-11-02T23:59:17+08:00
draft: false
original: true
categories: 
  - 书籍
tags: 
  - 读书笔记
---

# Chapter 1 **Reliable, Scalable, and Maintainable Applications**

many applications commonly need functionality:

- Store data so that they, or another application, can find it again later (*databases*)
- Remember the result of an expensive operation, to speed up reads (*caches*)
- Allow users to search data by keyword or filter it in various ways (*search indexes*)
- Send a message to another process, to be handled asynchronously (*stream pro‐
cessing*)
- Periodically crunch a large amount of accumulated data (*batch processing*)

It  painfully obvious, but different applications have different requirements

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

continuing to work correctly, even when things go wrong.

misleading

it suggests that we could make a system tolerant of every possible kind of fault

that is not feasible, makes sense to say:

tolerating *certain types* of faults.

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