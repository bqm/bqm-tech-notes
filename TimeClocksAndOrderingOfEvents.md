Time, Clocks, and the Ordering of Events in a Distributed System
=================================================================

| Source | http://amturing.acm.org/p558-lamport.pdf                                       |
|--------|------------------------------------------------------------------------------------------------|
| Author | Leslie Lamport                                                                                 |
| Status | In Progress (80%)                                                                                |
| Why?   | Classic paper on distributed systems. |
| My 2 cents| To be added

Introduction
------------

* Concept of time is fundamental to our way of thinking.
* However, we cannot rely on this concept in distributed systems.
* Distributed system is a collection of distinct processes which are spatially separated.
* In distributed systems, it is sometimes impossible to tell which event occured first.
* The best that we can achieve is a partially ordering of the events. 

Partial ordering
----------------

* We call this partial ordering 'happened before'.
* We can't rely on physical clocks.
* We assume that events in a single process form a total ordering.
* Definition of 'happened before', denoted a -> b:
  - If a & b are 2 events in the same process & a comes before b, a -> b.
  - if a is the sending of a message by one process & b is the receipt of the same message by another process, a -> b.
  - If a -> b, b -> c, a -> c.
* Two events a & b are concurrents if a -/-> b & b -/-> a.
* we assume that a -/-> a

Logical clocks
--------------

* A clock is a way of assigning a number to an event.
* Each process P\_i has an associated clock C\_i which define a number for each event a.
* Clock condition: for any events a & b, a -> b => C(a) < C(b)
* Therefore we have 2 conditions:
  - IR1. If a & b are events in a process P\_i, C\_i(a) < C\_i(b)
  - IR2. If a is sending a message by a process P\_i & b receiving by a process P\_j, C\_i(a) < C\_j(b)

* Meeting IR1: Each process P\_i increments C\_i between 2 successive events.
* Meeting IR2: If event a is sending message by P_i, we call T = C\_i(a), process P\_j must set C\_j(b) > T.

Ordering the events totally
---------------------------

* We can create an arbitrary total ordering called =>, such that:
  - a => b if and only if, C\_i(a) < C\_j(b) or C\_i(a) = C\_j(b)
* Ordering => is not unique, however it's useful anyway to create one.
* Example: A system composed of fixed collection of processes which share a single resource.
  - A process which has been granted the resource must release it before it can be granted to another process.
  - Different requests for the resource must be granted in the order in which they are made.
  - If every process which is granted the resource eventually releases it, then every request is eventually granted.
* Answer: essentially, => allows us to track which process asked for the resource first & when they released the resource.
* There is no central authority, each process independently follow the rules.
* If one process goes down, everything falls apart.

Anomalous behavior
-----------------

* Let's take a set of events E. If a & b are events of E, the total ordering can say b => a when in reality, a was done before.
* Essentially, the unknown cases can be resolved incorrectly.
* There might be a bigger set E' which contains events that could have allowed us to decide correctly that a -> b.
* 2 ways to prevent this:
  - Manually letting the processes take into account external events.
  - Strong clock condition: For any event a & b in E, if a -> b then C(a) < C(b) where -> is the relation 'happened before' in E'.

Physical clocks
---------------

* Let C\_i(t) denote the reading of C\_i at physical time t.
* In order for C\_i to be a physical clock, we assume that P1 & P2 hold:
  - P1. There exists a constant K << 1 such that for all i, |dC\_i(t) / dt - 1| < K
  - P2. For all i,j: |C\_i(t) - C\_j(t)| < epsilon
* Question: How to ensure that P2 holds?
  - IR1'. For each i, if P\_i does not receive a message at physical time t, then C\_i differentiable at t and dC\_i(t)/dt > 0.
  - IR2'. (a) If P\_i sends a message m at physical time t, then m contains a timestamp Tm = C\_(t). (b) Upon receiving a message m at time t', process P\_j sets C\_j(t') equal to maximum (C\_j(t' - 0), Tm + um) where um is the minimum delay that the message must have took.




