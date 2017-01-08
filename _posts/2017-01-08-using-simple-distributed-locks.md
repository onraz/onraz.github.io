---
layout: post
published: false
title: Using Simple Distributed Locks
categories: ''
tags: ''
---

DynamoDb offer simple set of primitives that makes it possible to implement a clustered lock.

- Strongly consistent reads - All nodes get consistent view of the lock within a Region, unless AZ replication fails due to network issue. Establishes a “happens-before” relationship that a consistent read reflects all successful writes that happened before that read.

- Conditional writes - We can attempt to write the lock if it doesn’t exist.

This algorithm trades lock expiration robustness for ease of use. Instead of a robust consensus algorithm, where all nodes are asked to confirm it doesn’t hold the lock, we simply put a TTL on the lock item.

The critical section of the code must finish before the expiry of the Lock.

Prune expired lock
Create Item unless exists
Yield to critical section inside of timeout
Delete Item



