---
layout: post
title: Functional Programming - Improving Readability of Code
date: 2014-06-03 11:59:05.000000000 +10:00
categories:
- Functional Programming
tags:
- functional programming
- java
---

<p>Coding is in many ways a form of communication. Just like any human language, a programming language allows us to express our intention in many different ways. A well written code can communicate its intention clearly and concisely to other developers. <em>Functional Programming</em> is a way of achieving this, and in the following example I'd like to illustrate how we can communicate better with Functional Programming constructs in Java 8.</p>
<h3>Identifying Prime Numbers</h3>
<p>Consider the rather simple task of printing the Prime numbers within a given List.<br />
Note that a Prime Number is a positive integer greater than 1 that is only divisible by 1 and itself.</p>
<p>The program to achieve this using a simple imperative style may look like this:</p>
<pre class="whitespace-before:1 whitespace-after:1 lang:java decode:true">boolean isPrime(int num) {
	boolean prime = true;
	if (num > 1) {
		for (int divisor = 2; divisor < num; divisor++) {
			if (num % divisor == 0) {
				prime = false;
				break;
			}
		}
	} else {
		prime = false;
	}
	return prime;
}

/**
 * Print prime numbers in a given list
 */
void printPrimes(List<Integer> values) {
	for (Integer num : values) {
		if (isPrime(num)) {
			System.out.println(num);
		}
	}
}</pre>
<p>We can write the above in a declarative, functional way as follows:</p>
<pre class="whitespace-before:1 whitespace-after:1  lang:java decode:true">boolean isPrime(int num) {
	return num > 1 &amp;&amp; IntStream.range(2, num)
				.noneMatch(divisor -> num % divisor == 0);
}

void printPrimes(List<Integer> values) {
	values.stream()
		.filter(Util::isPrime)
		.forEach(System.out::println);
}</pre>
<p>Even though its a rather simple example, notice the lack of control structures (e.g. for/if) in the second approach. It is more like story telling instead of providing rigorous instructions to solve a problem. This is different from the imperative style, where we specify the details of how the problem needs to be solved as well as the control structure of the program. In this light, the second approach has the following advantages:</p>
<ul>
<li>Since the details of how a task is performed is left out, the compiler/runtime can optimise how the tasks are carried out or even perform Lazy Evaluation</li>
<li>The cognitive load of understanding/following the control structures has been eliminated, thus improved clarity &amp; readability</li>
<li>The second approach doesn't require a variable, thus reduces mutable state - which is desirable in concurrency</li>
</ul>
<p>Functional Program improves program correctness with immutability and side-effect free functions, hence there is little need for tricky and expensive synchronisation for shared mutable state. These properties make Functional Programming a natural choice for designing highly concurrent and distributed systems.</p>
<p>Consider how easily the above program could be made concurrent and faster using Parallel Stream:</p>
<pre class="whitespace-before:1 whitespace-after:1  lang:java decode:true">
void printPrimes(List<Integer> values) {
	values.parallelStream()
		.filter(Util::isPrime)
		.forEach(System.out::println);
}</pre>
<p>This highlights the ability of Functional Programming to scale horizontally in a distributed environment, which is increasingly important  in the context of Big Data and multi-core CPU/GPU programming.</p>
<p>For the complete code example please see: <a href="https://github.com/openraz/java8-functional/blob/master/FunctionalJava8/src/examples/ImperativeDeclarativePrime.java" target="_blank">https://github.com/openraz/java8-functional/blob/master/FunctionalJava8/src/examples/ImperativeDeclarativePrime.java</a></p>
