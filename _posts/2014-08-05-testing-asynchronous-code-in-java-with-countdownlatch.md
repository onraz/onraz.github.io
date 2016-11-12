---
layout: post
title: Testing Asynchronous Code in Java with CountDownLatch
date: 2014-08-05 16:42:20.000000000 +10:00
categories:
- java
tags:
- concurrency
- java
- testing
---

Asynchronous Event driven applications are becoming ever more common, and testing the 
correctness of these applications can be tricky. However, there are some techniques and 
tools available to aid testing asynchronous code - one such tool is a CountDownLatch.

## Executing Concurrent Tasks

Suppose we have an application that runs simple tasks. 
These tasks can take varying amount of time to finish. 
Upon the completion of a task, a task can be marked as executed.

```java
class Task implements Runnable {
	private String taskId;
	private boolean executed;

	public Task(String taskId) {
		this.taskId = taskId;
	}

	@Override
	public void run() {
		System.out.println("Performed task " + taskId);
		executed = true;
	}

	public boolean hasExecuted() {
		return executed;
	}
}
```

The Tasks are run by a TaskRunner object, which uses an Executor Service to run tasks concurrently:

```java
class TaskRunner {
	ExecutorService executor = Executors.newCachedThreadPool();

	public void executeTasks(List<Task> tasks) {
		for (Task task : tasks) {
			executor.submit(task);
		}
	}

	public void tearDown() {
		executor.shutdown();
	}
}
```

## Basic Test Case
If we have to write a very basic test case for the task runner, it may look like:

```java
@Test
public void testExecution() throws InterruptedException {
	// Generate Sample Tasks
	List<Task> tasks = new ArrayList<>();
	for (int i = 0; i < 10; i++) {
		tasks.add(new Task("Task " + i));
	}

	taskRunner.executeTasks(tasks);

	// Give the tasks sufficient time to finish
	Thread.sleep(2000);

	for (Task task : tasks) {
		assertTrue(task.hasExecuted());
	}
}
```

In the above test we had to a `Thread.sleep()` of 2 seconds because the tasks may not have finished before we reach assert statements.
However, the tasks may take more or less than 2 seconds to finish.
There are at least two problems with this approach:

* The test is unreliable, as running the tests on a faster or slower machine or build agent influences the result of the test.
* These kind of tests also make the build slower when unit tests are run as part of the build. This goes against the principles of [Continuous Integration][1]

We can circumvent these concerns with a `CountDownLatch`.

## What is a CountDownLatch?
A CountDownLatch is a construct that allows one or more threads to wait until a set of operations being performed in other thread completes.

*  The latch is initialised with a *Count*, a positive integer e.g. 2
*  The thread that calls` latch.await()` will block until the *Count* reaches to *Zero*
*  All other threads are required to decrement the Count by calling `latch.countDown()`
*  Once the *Count* reaches Zero, the awaiting thread resumes execution

![Countdown Latch][3]

Once a latch reaches Zero, it can no longer be used, a brand new latch needs to be created. 
However, a CyclicBarrier may be more suited for such requirements.
The following is a simple usecase of how to use a CountDownLatch:

```java
CountDownLatch latch = new CountDownLatch(3);
ExecutorService executor = Executors.newCachedThreadPool();

// submit three tasks
for (int i = 0; i < 3; i++) {
	executor.submit(new Runnable() {
		@Override
		public void run() {
			// do long running task here
			System.out.println("Performing long task...");
			// when task finished, countDown
			latch.countDown();
		}
	});
}

// wait until task is finished
latch.await();
System.out.println("All tasks are done!! ");

executor.shutdown();
```

## Better Tests using Latch
`TestRunner` test we discussed previously using `Thread.sleep(...)` can be written using a `CountDownLatch`. 
Note how the test calls `latch.await()` to wait for all tasks to finish before it can verify the assertions.

```java
@Test
public void testExecutionLatch() throws InterruptedException {
	CountDownLatch latch = new CountDownLatch(10);
	List<Task> tasks = new ArrayList<>();
	for (int i = 0; i < 10; i++) {
		// Create latched tasks that countdowns the latch when it finishes
		tasks.add(new Task("Task " + i) {
			@Override
			public void run() {
				super.run();
				latch.countDown();
			}
		});
	}

	taskRunner.executeTasks(tasks);

	// wait for all tasks to finish
	latch.await();

	for (Task task : tasks) {
		assertTrue(task.hasExecuted());
	}
}
```

This gist can be found [here][2].

#### Tip
If a `Task` doesn't finish due to a bug in our `TestRunner` implementation, then test may forever block. 
Thus, its a good idea to impose a timeout on our tests either by introducing timeout parameter in the 
`@Test` annotation e.g. `@Test(timeout=2000)` or simply specifying a timeout in the await 
`latch.await(2, TimeUnit.SECONDS);`.


[1]: http://en.wikipedia.org/wiki/Continuous_integration#Keep_the_build_fast
[2]: https://gist.github.com/openraz/21f6bae97795ea145ea0
[3]: /assets/tasks.png
