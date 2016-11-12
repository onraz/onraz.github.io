---
layout: post
title: Hibernate Derived Properties
date: 2008-12-10 23:28:15.000000000 +11:00
categories:
- Hibernate
- java
tags:
- Hibernate
- java
---
<p>Couple of days back I had an interesting design discussion with some fellow mates about how to handle a bean property that is derived from other properties of the bean and is also persisted using hibernate. The issue might seem a bit trivial/counter-intuitive at first as a derived property doesn't need to be persisted - as it can always be generated from the other properties.</p>
<p>However, this situation may occur in some legacy systems - and we might face a design issue in writing a good way of handling the property. I will be discussing a few of the ways it can be dealt with - looking at the strength and weaknesses of each approach.</p>
<p>It is common for legacy databases to have a first name, last name and full name columns in the table that represents customers or persons.</p>
<p>For the purposes of this discussion, we would also assume that there are different legacy systems that feed person data in to this database, some may export both first/last and full name, while some may export either first/last names or full name.</p>
<p>Our job is to design a java web application that allows CRUD operations on the person database.</p>
<p>Let’s say for the front end, we only let the user modify the first and last names and their name is generated from the first and last names - which gets displayed in other pages. The user can never enter their full name using the java front end.</p>
<p>Thus when designing a java CRUD web application for this table we can start off with a bean like below. Let us also assume that we have setup hibernate to construct person beans from the database rows - and in our configuration we have asked hibernate to use property access, ie via getters/setters of the bean instead of directly accessing fields using reflection.</p>
<p>[java]<br />
class Person {<br />
private String firstName;<br />
private String lastName;<br />
private String fullName;<br />
public String getFirstName() {<br />
return firstName;<br />
}<br />
public void setFirstName(String firstName) {<br />
this.firstName = firstName;<br />
updateName();<br />
}<br />
public String getLastName() {<br />
return lastName;<br />
}<br />
public void setLastName(String lastName) {<br />
this.lastName = lastName;<br />
updateName();<br />
}<br />
public String getFullName() {<br />
return this.fullName;<br />
}<br />
public void setFullName(String fullName) {<br />
this.fullName = fullName;<br />
}<br />
private void updateName() {<br />
setFullName( this.firstName + " " + this.lastName );<br />
}<br />
}<br />
[/java]</p>
<p>As you can see this leads to a leaky implementation, the client of the bean can easily call setFullName(...) and the integrity of the object might get broken.</p>
<p>So a better approach is to make setFullName(...) private so that other developers don't get confused nor can they accidently execute it. Some comments on top of setFullName(...) will be handy too.</p>
<p>[java]<br />
class Person {<br />
/** Never call this - only used internally */<br />
private void setFullName(String fullName) {<br />
this.fullName = fullName;<br />
}<br />
}<br />
[/java]</p>
<p>Note that this works because hibernate can access private getters / setters using reflection.</p>
<p>But now we realize we have another problem - depending on the exact sequence hibernate calls these getters / setters we may end up in an invalid state.</p>
<p>For instance, consider this sequence when hibernate loads an object:</p>
<p>1. hibernate : <span class="lang:default decode:true  crayon-inline ">person.setName("Raz Shahriar")</span><br />
2. hibernate : <span class="lang:default decode:true  crayon-inline ">person.setFirstName("");</span><br />
3. hibernate : <span class="lang:default decode:true  crayon-inline ">person.setLastName("");</span></p>
<p>So we don't want to get into inconsistent states neither from the hibernate end nor from the frontend/controller end.</p>
<p>Another attempt might be to modify the getter of the full name in this way:</p>
<p>[java]<br />
@SuppressWarnings("unused")<br />
Class Person {<br />
// simple first and last name setters<br />
public void setFirstName(String firstName) {<br />
this.firstName = firstName;<br />
}<br />
public void setLastName(String lastName) {<br />
this.lastName = lastName;<br />
}<br />
/** Never call this - only used internally */<br />
private void setFullName(String fullName) {<br />
// this is kept because hibernate might need its presence when loading getters/setters using reflection<br />
}<br />
// full name is derived and saved<br />
public String getFullName() {<br />
return this.firstName + this.lastName;<br />
}<br />
}<br />
[/java]</p>
<p>This means that we don't need to call any <span style="font-family: 'courier new', courier;">updateName()</span> method each time first name / last name are changed - as full name becomes completely dynamic always derived from the first/last names.</p>
<p>This solution is simple and sound, and gets the job done quite well. However, as a side-effect we see that every time a person object is loaded, it might become dirty straight away if the value stored in the database for the name field differs from the getter of the name field.</p>
<p>There might be certain scenarios where we would be loading person objects as part of an object graph (e.g. Project might have a map of person or Timesheet might load a reference to person) - and when persisting the project object we might not want to persist the person object.</p>
<p>Although there are many solutions to stop that unwanted side-effect, the easiest and the most obvious solution is to <em>disable cascading</em> for the person map in the hibernate mapping of project - so this concern can be ignored.</p>
<p>There could be other java based approaches as well, e.g. <em>using a factory to</em> create the person object which ensures the person name is in a consistent state - but these might create additional restrictions on how clients use the person object.</p>
<h2>Hibernate Forumula</h2>
<p>Now lets consider similar approaches using hibernate. One might also apply this logic in person hibernate mapping.</p>
<p>[xml]<br />
<!-- Note: you can also use concat(firstname, ' ', lastname) for the formula. --><br />
[/xml]</p>
<p>[java]<br />
class Person {<br />
// only when first and last name are changed then update the name field<br />
public void setFirstName(String firstName) {<br />
this.firstName = firstName;<br />
updateName();<br />
}<br />
public void setLastName(String lastName) {<br />
this.lastName = lastName;<br />
updateName();<br />
}<br />
private void updateName() {<br />
setFullName( this.firstName + " " + this.lastName );<br />
}<br />
public String getFullName() {<br />
return this.fullName;<br />
}<br />
private void setFullName(String fullName) {<br />
this.fullName = fullName;<br />
}<br />
}<br />
[/java]</p>
<p>This way the name property is always loaded as derived from first and last names. One might want to use SQL case statements in the formula if the concatenation needs some special logic for the order of the concatenation. A minor efficiency present in the solution is that name is only updated when it needs to be.</p>
<h2>Alternative Approach</h2>
<p>Another approach one might take is to take advantage of LifeCylcle interface for persistent domain objects instead of changing the hibernate mapping.</p>
<p>[java]<br />
class Person implements LifeCycle {<br />
public void setFirstName(String firstName) {<br />
this.firstName = firstName;<br />
updateName();<br />
}<br />
public void setLastName(String lastName) {<br />
this.lastName = lastName;<br />
updateName();<br />
}<br />
private void updateName() {<br />
setFullName( this.firstName + " " + this.lastName );<br />
}<br />
public String getFullName() {<br />
return this.fullName;<br />
}<br />
private void setFullName(String fullName) {<br />
this.fullName = fullName;<br />
}<br />
// called right AFTER the person is loaded and all fields are initialised<br />
public void onLoad(..) {<br />
updateName ();<br />
}<br />
// called right BEFORE the person is saved<br />
public void onSave(..){<br />
// any other logic for saving or assertions<br />
}<br />
}<br />
[/java]</p>
<p>Although this adds more power to the domain object (may be more than it should have) - but it also gives the uncomfortable feeling of coupling the domain object with a repository level design/technology - suppose you want move to a JPA implementation - then you pretty much have to change your domain object to make things work.</p>
<p>Arguably, a better alternative to using the LifeCycle interface is <em>to use a hibernate onLoad listener</em> - that executes <span style="font-family: 'courier new', courier;">updateName()</span> upon loading the person object. That way the domain object is decoupled from a repository level interface.</p>
<p>I'm pretty sure there are many other ways of doing the same thing - and most of these are minor variations of each other - so pick an approach that you think is best for your usecase.</p>