---
layout: post
title: Hibernate Load Vs Get Differences
date: 2009-01-05 23:57:08.000000000 +11:00
categories:
- Hibernate
- java
tags:
- Hibernate
---
Programmers new to hibernate may face dilemmas trying to understand the subtle differences between hibernate's get and
load methods for retrieving an entity. I remember my struggle to grasp the differences, and thought I would compile the
 information I got from various web sources into one single post to help out others facing similar issues. 

Here is a refresher on how these two methods can be used:

```java
// Open the session
Session session = sessionFactory.openSession();
Transaction tx = session.beginTransaction();
// Load the Persistent entity using either Get or Load
Item itemByLoad = (Item) session.load(Item.class, new Long(1234));
//or should you use???
Item itemByGet = (Item) session.get(Item.class, new Long(1234));
tx.commit();
session.close();
```
Below we explore the various aspects of Load/Get operations in Hibernate to understand their differences.

## Javadoc

* **Get:** Return the actual instance of the given entity class with the given ID, or null if there is no such instance. 
  (If the instance is already exists in the current Session, return that instance or proxy.)
* **Load:** Return a proxy instance of the given entity class with the given ID, assuming that the instance exists 
in the database.

* *You should not use this method to determine if an instance exists* (use get() instead).
* *Use this only to lazily retrieve an instance that you assume exists, where non-existence would be an actual error*.


## Querying for an instance that doesn't exist in the database

* **Get:** Database hit occurs· Returns *null*
* **Load:** No database hit occurs· Returns a *proxy·*
 Calling getter/setter on the proxy throws **ObjectNotFoundException** 
 `org.hibernate.ObjectNotFoundException: No row with the given identifier exists: [User#123]` 


## Querying for an instance that exists in the database

* **Get:** Database hit occurs· Returns the actual instance, not proxy i.e. *no Lazy Loading optimization!*
* **Load:** Will always return a proxy (*lazy loading optimization*)

* Database hit will not occur till you use getter/setter on the returned proxy.
* If you don't use getter/setter on the proxy load never generates a database hit, ie the object is never actually loaded from the database
* The only exception to the above is when the object is already present in the persistence cache, it will return the cached instance instead of a proxy




## Accessing properties from a detached entity
An entity becomes detached when it is outside of an active Hibernate Session.

* **Get:** · All non-lazy properties can be accessed *no Exception is thrown*
* **Load:** If you didn't access any properties while the detached entity was in persistent within the persistent context, 
then your detached entity is basically a proxy placeholder – Calling any getter/setter will throw a LazyInitialization exception e.g.

```java
session.beginTransaction();
User user=(User)session.load(User.class, new Long(1));
session.getTransaction().commit();
System.out.println(user.getPassword());
```
The above generates `org.hibernate.LazyInitializationException: could not initialize proxy - no Session`

## Typical Usage Guidelines

* **Get:** For the most part, you'll probably use the get method most often in your code. If you ever want to use the JavaBean that you are retrieving from the database after the database transaction has been committed, you'll want to use the get method, and quite frankly, that tends to be most of the time. For example, if you load a User instance in a Servlet, and you want to pass that instance to a Java Server Page for display purposes, you'd need to use the get method, otherwise, you'd have a LazyInitializationException in your JSP
* **Load:** On the other hand, if your goal is largely transactional, and you are only going to be accessing the JavaBean of interest within a single unit of work that will pretty much end once the transaction is committed, you'll want to use the load method.Furthermore, the load method may be the method of choice if you know, and are absolutely sure, that the entity you are searching for exists in the database with the primary key you are providing.


## Example
The following illustrates the difference between Get/Load operation.

* **Get:** Two SELECTs and one UPDATE are generated
* **Load:** Only one SELECT and UPDATE for User are generated

```java
public class PetService {
	public void purchasePet(Long ownerUserId, Long petId) {
		Session session = getSessionFactory().openSession();
		Transaction tx = session.beginTransaction();

		User owner = session.get(User.class, ownerUserId);
		/* or, using load 
		 * User owner = session.load( User.class, ownerUserId); */

		Pet purchasedPet = session.get(Pet.class, petId);
		/* or using load Pet 
		 * purchasedPet = session.load( Pet.class, petId); */

		owner.setPet(purchasedPet);

		tx.commit();
		session.close();
	}
}
```

Also see my other post regarding Hibernate Derived Properties.
