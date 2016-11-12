---
layout: post
title: Groovy traits bring us closer to the vision of DCI Architecture
date: 2014-07-16 20:37:47.000000000 +10:00
categories:
- Functional Programming
---

Since the addition of Traits in Groovy 2.3, it becomes easier to implement the vision of DCI Architecture. 
The trait construct replaces the previously available `@Mixin` transformation, making traits a first class 
construct in the language itself. In this article we will explore how a simple DCI application can be developed 
using Groovy Traits.

## What are Traits?
Traits are a structural construct of the language that allow composition of behaviours. 
Similar to the `interface` construct in Java, traits are used to define object types by specifying supported methods. 
But unlike `interfaces`, traits can be *partially implemented* - allowing traits to provide default implementations for types. 
Traits can be used to implement multiple inheritance in a controlled way, without running into the [diamond problem][1]. 
Groovy resolves multiple inheritance conflicts by letting the last declared trait's method to win.
Traits provide a powerful design alternative to Inheritance for reusing behaviours. 
As an example, the concept of an Account can be modelled using traits. 
This allows us to create a reusable unit of behaviour that can be applied to any objects. 
Following is an example of how such a trait is declared:

```groovy
trait Account {
	double balance = 0
	
	void increaseBalance(double amount) {
		balance += amount
	}
	
	void decreaseBalance(double amount) {
		balance -= amount
	}
}
```
The trait of account can be used to represent a Savings Account Object, it can also be applied to other objects e.g. a Student Object to represent the concept of a Student Account. For a simple example, we will just represent two account types using the Account trait:

```groovy
class SavingsAccount implements Account {
	SavingsAccount(balance) { 
		this.increaseBalance(balance) 
	}
	
	@Override String toString() { 
		"Savings: ${balance}" 
	}
}

class CheckingAccount implements Account {
	CheckingAccount(balance) { 
		this.increaseBalance(balance)  
	}
	
	@Override String toString() { 
		"Checking: ${balance}" 
	}
}
```

## Representing DCI Roles using Traits
DCI (Data, Context, Interaction) is a vision to capture the end user cognitive model of roles and interactions between them. 
The paradigm separates the domain model (*data*) from use cases (*context*) and Roles that objects play (*interaction*). 
This allows us to cleanly separate code for rapidly changing system behavior (*what the system does*) 
from code for slowly changing domain knowledge (*what the system is*).

DCI promotes the decoupling of a Role that an object plays from the object itself. 
The same object can play different roles depending on the context. 
In our example, the same Account object can play the role of money source or a money destination in different transactions. 
These roles can be represented by the following Traits:

```groovy
trait TransferMoneySource implements MoneySource {
	void withdraw(double amount, MoneyDestination dest) {
		if (getBalance() > amount) {
			this.decreaseBalance(amount)
			dest.deposit(amount)
			this.updateLog "Withdrawal of ${amount} performed"
		} else {
			throw new IllegalArgumentException("Insufficient Balance in Source")
		}
	}
}

trait TransferMoneyDestination implements MoneyDestination {
	public void deposit(double amount) {
		increaseBalance(amount)
	}
}
```
## Binding The Roles to Objects in a Use Case
The Context in DCI enacts the use-case by assigning roles to objects, and then the objects interact as their roles. 
Below, the first account plays the role of a Money Source, and the second account plays the role of a Money Destination. 
The Groovy `as` keyword binds the role trait to the object that plays the role. 
The objects in a use-case collaborate using only role methods.

```groovy
class WithdrawalContext {
	Account source, dest
	double amount
	
	def execute() {
		// Apply the role of a MoneySource to a source Account
		MoneySource moneySource = source as TransferMoneySource
		// Apply the role of a MoneyDestination to a destination Account
		MoneyDestination moneyDestination = dest as TransferMoneyDestination
		// Perform the usecase
		moneySource.withdraw(amount, moneyDestination)
	}
}
```

DCI allows the source code to reflect the run-time structure, as the network of interactions between Roles in the code is the same as the corresponding network of objects at run time. 
The following picture illustrates this network of interactions: 

![dci][2]

A sample application executing the use-case may look like:

```groovy
def savings = new SavingsAccount(50.0)
def checkings = new CheckingAccount(200.0)
...
new WithdrawalContext([ source :checkings, dest: savings, amount:100 ]).execute()
...
// Sample Output
Before Tranfer: Savings: 50.0, Checking: 200.0
Withdrawal of 100.0 performed
After Tranfer: Savings: 150.0, Checking: 100.0
```

To conclude, the dynamic nature Groovy traits provides an excellent tool for object composition and brings us closer to the DCI vision.

## Further Reading
All code can be found [here][3]. Read about the []DCI Vision][4] and further [DCI Resources][5].


[1]: http://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem
[2]: /assets/dci3.png
[3]: https://github.com/openraz/dci.git
[4]: http://www.artima.com/articles/dci_vision.html
[5]: http://fulloo.info/Documents/
