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
<p>Programmers new to hibernate may face dilemmas trying to understand the subtle differences between hibernate's get and load methods for retrieving an entity. I remember my struggle to grasp the differences, and thought I would compile the information I got from various web sources into one single post to help out others facing similar issues. Here is a refresher on how these two methods can be used:</p>
<p>[java"]<br />
// Open the session<br />
Session session = sessionFactory.openSession();<br />
Transaction tx = session.beginTransaction();</p>
<p>// Load the Persistent entity using either Get or Load<br />
Item itemByLoad = (Item) session.load(Item.class, new Long(1234));<br />
//or should you use???<br />
Item itemByGet = (Item) session.get(Item.class, new Long(1234));</p>
<p>tx.commit();<br />
session.close();<br />
[/java]</p>
<p>Below we explore the various aspects of Load/Get operations in Hibernate to understand their differences.</p>
<h2>Javadoc</h2>
<ul>
<li><strong>Get</strong>: Return the actual instance of the given entity class with the given ID, or null if there is no such instance. (If the instance is already exists in the current Session, return that instance or proxy.)</li>
<li><strong>Load</strong>: Return a proxy instance of the given entity class with the given ID, assuming that the instance exists in the database.
<ul>
<li><em>You should not use this method to determine if an instance exists</em> (use get() instead).</li>
<li><em>Use this only to lazily retrieve an instance that you assume exists, where non-existence would be an actual error</em>.</li>
</ul>
</li>
</ul>
<p>&nbsp;</p>
<h2>Querying for an instance that doesn't exist in the database</h2>
<ul>
<li><strong>Get</strong>: Database hit occurs· Returns <strong>null</strong></li>
<li><strong>Load</strong>: <strong>No</strong> database hit occurs· Returns a <strong>proxy</strong>· Calling getter/setter on the proxy throws <strong>ObjectNotFoundException </strong><strong> </strong><span class="lang:default decode:true  crayon-inline">org.hibernate.ObjectNotFoundException:No row with the given identifier exists: [User#123]</span><strong> </strong></li>
</ul>
<p>&nbsp;</p>
<h2>Querying for an instance that exists in the database</h2>
<ul>
<li><strong>Get</strong>: Database hit occurs· Returns the actual instance, not proxy i.e. <em>no Lazy Loading optimization!</em></li>
<li><strong>Load</strong>: Will <strong>always</strong> return a <strong>proxy </strong>(<em>lazy loading optimization</em>)
<ul>
<li>Database hit will <strong>not</strong> occur till you use getter/setter on the returned proxy.</li>
<li><em>If you don't use getter/setter</em><em>on the proxy </em>load <strong>never</strong> generates a database hit, ie the object is <strong>never</strong> actually loaded from the database</li>
<li>The only exception to the above is when the object is already present in the persistence cache, it will return the cached instance instead of a proxy</li>
</ul>
</li>
</ul>
<p>&nbsp;</p>
<h2>Accessing properties from a detached entity</h2>
<p>An entity becomes detached when it is outside of an active Hibernate Session.</p>
<ul>
<li><strong>Get</strong>: · All non-lazy properties can be accessed <em>no Exception is thrown</em></li>
<li><strong>Load</strong>: If you didn't access any properties while the detached entity was in persistent within the persistent context, then your detached entity is basically a proxy placeholder – Calling any getter/setter will throw a LazyInitialization exceptione.g.</li>
</ul>
<p>[java]<br />
session.beginTransaction();<br />
User user=(User)session.load(User.class, new Long(1));<br />
session.getTransaction().commit();<br />
System.out.println(user.getPassword());<br />
[/java]<br />
<em>The above generates </em> <span class="lang:default decode:true  crayon-inline">org.hibernate.LazyInitializationException: could not initialize proxy - no Session</span></p>
<p>&nbsp;</p>
<h2>Typical Usage Guidelines</h2>
<ul>
<li><strong>Get</strong>: For the most part, you'll probably use the get method most often in your code. If you ever want to use the JavaBean that you are retrieving from the database after the database transaction has been committed, you'll want to use the get method, and quite frankly, that tends to be most of the time. For example, if you load a User instance in a Servlet, and you want to pass that instance to a Java Server Page for display purposes, you'd need to use the get method, otherwise, you'd have a LazyInitializationException in your JSP</li>
<li><strong>Load</strong>: On the other hand, if your goal is largely transactional, and you are only going to be accessing the JavaBean of interest within a single unit of work that will pretty much end once the transaction is committed, you'll want to use the load method.Furthermore, the load method may be the method of choice if you know, and are absolutely sure, that the entity you are searching for exists in the database with the primary key you are providing.</li>
</ul>
<p>&nbsp;</p>
<h2>Example</h2>
<p>The following illustrates the difference between Get/Load operation.</p>
<ul>
<li><strong>Get</strong>: TWO SELECTS AND ONE UPDATE ARE GENERATED</li>
<li><strong>Load</strong>: ONLY ONE SELECT AND UPDATE FOR USER ARE GENERATED</li>
</ul>
<pre class="lang:default decode:true">public class PetService {
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
}</pre>
<p>Also see my other post Hibernate Derived Properties</p>