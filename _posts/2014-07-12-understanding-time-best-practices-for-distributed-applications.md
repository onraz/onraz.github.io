---
layout: post
title: 'Understanding Time: Best Practices for Distributed Applications'
date: 2014-07-12 15:32:26.000000000 +10:00
categories:
- Design
tags:
- best practice
- design
- time
---
![clock](/assets/1405161003_clock.png) 
When representing time in software, we need to be aware of the nature of time. 
Improper handling of time can easily create tricky issues that are hard to identify.

* This will become a table of contents (this text will be scraped).
{:toc}

## Key Concept: Time without location is meaningless

It took the genius of Albert Einstein to fully understand the true nature of time. 
Time can't exist without a location in space, as space and time are inter-connected. 
Simply put, if it is 9 AM in *Sydney*, it is not necessarily 9 AM *everywhere in the universe* 
(unless of course you have a broken watch). Without going into too much details of the physical properties of time, 
the important consequence of this concept is:

> To accurately represent or measure time in computer software, one must take into account the location where the time was measured.

There is no silver bullet or a technology framework that can solve all time related problems if we don't understand the above nature of time.

## Time is Relative and Contextual
When modelling time, we usually deal with two types of time data, `Civil Time` and `Physical Time`.

### Civil Time
Civil time is represented as a point on a Calendar or a local clock as agreed by civilian authorities. 
This is the type of time data we use in our day to day conversations. When capturing Civil Time, we need to be aware of the context in which it is specified. 
As an example, when we say *"Lets catch 9'o clock bus"* - a lot of information is implied by the context of the speaker. 
Such *implied or implicit context* is not always available in software design, and needs to be explicitly specified 
if the software will be used in more than one time zones.

> Therefore, we need to be mindful of the context and timezone when modelling time. 
> In addition, we also need to be aware of local transformations such as Day Light Saving Time (DST).

### Physical Time
Physical Time is represented as a point in the continuous universal timeline. 
This kind of time data can be adequately represented in Universal Time (UTC), 
which is calculated by reference to atomic clocks. Physical time is useful in storing calculations 
and measurements, as it is unambiguous and **context-free**.

## Best Practices
Time is a complex topic, however we can still follow some best practices to avoid some pitfalls that may occur in 
distributed client-server applications.

### Persist Globally, Display Locally
When you store time, store it in UTC and use server time. UTC time is DST agnostic, 
thus it is a good approach of storing time without confusion.

### Delayed/Late Conversion
Only apply timezone/DST in the Last possible moment when you display the time to client. 
Only apply timezone formats at the very end, when the time is being displayed at the client terminal/browser. 
When converting to timezone, remember that timezones may change, and an entire state or country may not be in the same timezone.

### Capture Context
When capturing Time as input, take into consideration how the client is specifying the time, 
and for what purpose. Understand the context in which a time related information is specified. 
Here are a few examples of different contexts where the meaning of time can vary:

Example Type | Example Usage | Contextual Meaning
-------------|---------------|-------------------
Event Time | The concert is going to start at 10pm | Event time is always relative to the location of where the event takes place
Contract Time | The car insurance is valid till 31st January next year | Depending on the insurance company's policies, the policy may expire at 5pm 31st January depending on where the policy is purchased from. Exact meaning needs to be clarified.
Recurring Time | The TV show airs every day at 9am | Everyday at 9am local time, regardless of DST
Floating Durations |Tech support available during Business Hours, Express Trains available during Peak Hours on Weekdays | Always based on local time and day of week, subjected to holidays etc.
Time interval or relative time | Contract is due for renewal every year, or we charge you phone fee on a monthly basis | Storing agreed interval from a reference point.

### Use the Universally Accepted ISO format for Web Services
When designing web based services that expose time 
(e.g. created date, modified data) it is best to keep it in a human readable ie. string form rather than a long number. 
It allows interoperability between different types of clients, and also allows humans to understand and debug the service. 
Standardised formats as [ISO 8601][1] should be considered.

> Prefer ISO date format in representing date/time for web services over Numeric time (e.g. Long 1341542232312) 
> which has limited readability, usability or portability.

ISO date format is the best choice for a date representation that is accurately/universally (e.g. W3C) understandable. 

[1]: http://en.wikipedia.org/wiki/ISO_8601
