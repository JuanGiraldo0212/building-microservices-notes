# Chapter 11: Microservices at Scale

---

## Prevent failure or deal with it gracefully?

We can also spend a bit less of our time trying to stop the inevitable, and a bit more of
our time dealing with it gracefully. I’m amazed at how many organizations put processes and controls in place to try to stop failure from occurring, but put little to no
thought into actually making it easier to recover from failure in the first place.

---

## Which 3 requirements do you need to understand to handle failure?

-  Response time/latency
-  Availability
-  Durability of data

---

## What question do you have to make about response time or latency? and why is important to ask such question?

How long should various operations take? It can be useful here to measure this
with different numbers of users to understand how increasing load will impact
the response time.

---

## When it comes to "Avaiability", what's important to consider? (3 questions)

Can you expect a service to be down? Is this considered a 24/7 service? Some
people like to look at periods of acceptable downtime when measuring availability, but how useful is this to someone calling your service? I should either be able
to rely on your service responding or not. Measuring periods of downtime is
really more useful from a historical reporting angle.

---

## When talking about Durability of data, What 2 questions are important to answer? and what's important to consider?

How much data loss is acceptable? How long should data be kept for? This is
highly likely to change on a case-by-case basis. For example, you might choose to
keep user session logs for a year or less to save space, but your financial transaction records might need to be kept for many years.

---

## When and how should you "degrade functionality" in your microservices?

What we need to do is understand the impact of each outage, and work out how to
properly degrade functionality. If the shopping cart service is unavailable, we’re probably in a lot of trouble, but we could still show the web page with the listing. Perhaps
we just hide the shopping cart or replace it with an icon saying “Be Back Soon!”

---

## What do you have to ask yourself when dealing with multiple microservices that depend on multiple downstream collaborator? (Resilience) (2)

But for every customer-facing
interface that uses multiple microservices, or every microservice that depends on
multiple downstream collaborators, you need to ask yourself, “What happens if this is
down?” and know what to do.

---

## Systems that act slow or systems that fail fast? Which is the worst?

When you get down to it, we discovered the hard way that systems
that just act slow are much harder to deal with than systems that just fail fast. In a
distributed system, latency kills.

---

## What three (3) fixes did the Sam's team implement in the ads website project where he was the technical lead?

We ended up implementing three fixes to avoid this happening again: getting our timeouts right,
implementing bulkheads to separate out different connection pools, and implementing a circuit breaker to avoid sending calls to an unhealthy system in the first place.

---

## Explain Chaos Monkey

The most famous of these programs is the Chaos Monkey, which during certain
hours of the day will turn off random machines. Knowing that this can and will happen in production means that the developers who create the systems really have to be
prepared for it.

---

## Explain the trade-offs when deciding "Timeouts" (3)

Wait too long to decide that a call has failed, and you can slow the whole system
down. Time out too quickly, and you’ll consider a call that might have worked as
failed. Have no timeouts at all, and a downstream system being down could hang
your whole system.

---

## Where do you have to put timeouts? and what do you have to do with them?

Put timeouts on all out-of-process calls, and pick a default timeout for everything.
Log when timeouts occur, look at what happens, and change them accordingly

---

## Explain "Circuit Breakers"

With a circuit breaker, after a certain number of requests to the downstream resource
have failed, the circuit breaker is blown. All further requests fail fast while the circuit
breaker is in its blown state. After a certain period of time, the client sends a few
requests through to see if the downstream service has recovered, and if it gets enough
healthy responses it resets the circuit breaker. You can see an overview of this process
in Figure 11-2.

![image](https://user-images.githubusercontent.com/1868409/92310004-12dfeb00-ef78-11ea-8a72-e2350a00e4d6.png)

---

## What do you have to do with a request when a circuit breaker is blown (open)? (synchronous and asynchronous operations)

While the circuit breaker is blown, you have some options. One is to queue up the
requests and retry them later on. For some use cases, this might be appropriate, especially if you’re carrying out some work as part of a asynchronous job. If this call is
being made as part of a synchronous call chain, however, it is probably better to fail
fast. This could mean propagating an error up the call chain, or a more subtle degrading of functionality.

---

## Explain Bulkheads

We should have used different connection pools for each downstream connection. That way, if one connection pool gets exhausted, the other connections aren’t
impacted, as we see in Figure 11-3. This would ensure that if a downstream service
started behaving slowly in the future, only that one connection pool would be impac‐
ted, allowing other calls to proceed as normal.

![image](https://user-images.githubusercontent.com/1868409/92310219-30ae4f80-ef7a-11ea-9284-10c24c66b99a.png)

---

## How can we combine circuit breakers with bulk-heads?

We can think of our circuit breakers as an automatic mechanism to seal a bulkhead,
to not only protect the consumer from the downstream problem, but also to potentially protect the downstream service from more calls that may be having an adverse impact.

---

## Why is important to keep isolation in our services?

The more one service depends on another being up, the more the health of one
impacts the ability of the other to do its job. If we can use integration techniques that
allow a downstream server to be offline, upstream services are less likely to be affected by outages, planned or unplanned

---

## What's an Idempotent operation?

In idempotent operations, the outcome doesn’t change after the first application, even
if the operation is subsequently applied multiple times. If operations are idempotent,
we can repeat the call multiple times without adverse impact. This is very useful when
we want to replay messages that we aren’t sure have been processed, a common way
of recovering from error.

---

## When does using Idempotent operations work well?

This mechanism works just as well with event-based collaboration, and can be especially useful if you have multiple instances of the same type of service subscribing to
events. Even if we store which events have been processed, with some forms of asynchronous message delivery there may be small windows where two workers can see
the same message. By processing the events in an idempotent manner, we ensure this
won’t cause us any issues.

---

## Why is good to move microservices into their own hosts?

As the microservices are independent
processes that communicate over the network, it should be an easy task to then move
them onto their own hosts to improve throughput and scaling. This can also increase
the resiliency of the system, as a single host outage will impact a reduced number of
microservices.

---

## What's important to consider when looking at SLA? (SLA stands for...?)

If you’re using an underlying service provider, it is important to know if a service-level agreement (SLA) is offered and plan
accordingly. If you need to ensure your services are down for no more than four
hours every quarter, but your hosting provider can only guarantee a downtime of
eight hours per quarter, you have to either change the SLA, or come up with an alternative solution.

---

## Explain VLAN

One mitigation is to have all the instances of the microservice inside a single
VLAN, as we see in Figure 11-5. A VLAN is a virtual local area network, that is isolated in such a way that requests from outside it can come only via a router, and in
this case our router is also our SSL-terminating load balancer. The only communication to the microservice from outside the VLAN comes over HTTPS, but internally
everything is HTTP.

![image](https://user-images.githubusercontent.com/1868409/92339525-05685500-f08d-11ea-8fca-f72cdcc28bfb.png)

---

## Explain what Jeff Dean said in its presentation: "“Challenges in Building Large-Scale Information Retrieval Systems" (2)

The architecture that gets you started may not be the architecture that keeps you
going when your system has to handle very different volumes of load. As Jeff Dean
said in his presentation “Challenges in Building Large-Scale Information Retrieval
Systems” (WSDM 2009 conference), you should “design for ~10× growth, but plan to
rewrite before ~100×.” At certain points, you need to do something pretty radical to
support the next level of growth.

---

## Why is preparing our systems for a massive usage from the very beginning a bad idea?

We need to be able to rapidly
experiment, and understand what capabilities we need to build. If we tried building
for massive scale up front, we’d end up front-loading a huge amount of work to prepare for load that may never come, while diverting effort away from more important
activities, like understanding if anyone will want to actually use our product.

---

## How can we scale reads in relational databases? How is such setup called?

In a relational database management system (RDBMS) like MySQL or Postgres, data
can be copied from a primary node to one or more replicas. This is often done to
ensure that a copy of our data is kept safe, but we can also use it to distribute our
reads. A service could direct all writes to the single primary node, but distribute reads
to one or more read replicas, as we see in Figure 11-6. The replication from the primary database to the replicas happens at some point after the write. This means that
with this technique reads may sometimes see stale data until the replication has completed. Eventually the reads will see the consistent data. Such a setup is called eventually consistent, and if you can handle the temporary inconsistency it is a fairly easy
and common way to help scale systems

![image](https://user-images.githubusercontent.com/1868409/92339830-69d7e400-f08e-11ea-9961-0b324caad525.png)

---

## What approach do we have when scaling for writes?

One approach is to use
sharding. With sharding, you have multiple database nodes. You take a piece of data
to be written, apply some hashing function to the key of the data, and based on the
result of the function learn where to send the data. To pick a very simplistic (and
actually bad) example, imagine that customer records A–M go to one database
instance, and N–Z another.

---

## In short words, Explain CQRS

The Command-Query Responsibility Segregation (CQRS) pattern refers to an alter‐
nate model for storing and querying information. With normal databases, we use one
system for performing modifications to data and querying the data. With CQRS, part
of the system deals with commands, which capture requests to modify state, while
another part of the system deals with queries.
