---
title: What is CQRS + Event Sourcing and why is it so Awesome
updated: 2015-12-21 17:34
keywords: CQRS, Event Sourcing, C#, Awesome, Command Query Responsibility Segregation, CQS
description: What is CQRS plus Event Sourcing and what makes it Awesome.
---

CQRS and Event Sourcing are both design patterns in their own right, although they are often used together to create a highly powerful framework that comes with a wealth of features that can increase performance, responsiveness, scalability and security. We’re going to look at what these design patterns are and how they could forever change the way you look at and think of software architecture!

## CQRS

The Command Query Responsibility Segregation (CQRS) pattern is based on the Command-Query Separation (CQS) pattern which states that a method may either be a command or a query but never both.

A command in this context is a method that performs an action and does not return a value:

```cs
public void SetHousePriceCommand()
{
    // logic to set house price...    
}    
```

And a query is a method that simply returns a value:

```cs
public int GetHouseCountQuery()
{
    int houseCount = // logic to retrieve house count... 
    return houseCount;   
}    
```


CQRS expands on CQS by segregating the model that updates the data from the model that returns the data, in effect having a command model and query model. As opposed to a CRUD application where by the model to update an entity could be the same as the model that is retrieved to view an entity:

```cs
public class HouseModel
{
    public int Id { get; set; }
    public decimal Price { get; set; }
    public int Bathrooms { get; set; }
    public int Bedrooms { get; set; }
}
```

If we go ahead and apply CQRS to our CRUD app we would segregate our house model into a command and query model respectively based on a scenario in the domain. 

So given a scenario where we want to update a house price, we could have a Update House Price Command Model that is sent to our House repository:

```cs
public class UpdateHousePriceCommand
{
    public int HouseId { get; set; }
    public decimal Price { get; set; }
}
```

And a House Query Model that is returned from our repository:

```cs
public class GetAllHouseInfoQuery
{
    public int HouseId { get; set; }
    public decimal Price { get; set; }
    public int Bathrooms { get; set; }
    public int Bedrooms { get; set; }
}
```

By segregating our house models we never use the same model to update and retrieve an entity.

## Event Sourcing

The Event Sourcing pattern is the process of capturing all changes to an aggregate as a series of events. As opposed to a CRUD application whereby the current state is updated and queried with no regard for how the state came to be in the first place.

All manipulations to an aggregate are persisted through a series of event objects. Whenever an aggregate is then retrieved from its store, in order to get its current state we first retrieve an empty aggregate, take each of the events in chronological order and apply them one by one until we have the current most recent state of our aggregate.

Given our simple CRUD app with an SQL storage base, the process of creating a house, adjusting its price, changing the real estate agent of the house and adjusting the price once again might consist of the following steps:

- Send a create request to our repository to create a new house entity
- Send an update request to our repository to update the price of our house entity
- Send an update request to our repository to update the real estate agent of our house entity
- Send an update request to our repository to update the price of our house entity

If we were then to recreate our domain using the Event Sourcing pattern we would have a separate event object for each scenario that would result in a state change of our aggregate, here is what a serialized event stream of our aggreagate might look like:

```
HouseCreatedEvent 
{
    HouseId : 1
    Price : 5,000,000
    BathroomCount : 4
    BedroomCount : 6
    AgentId : 1   
}
```

```
HousePriceChangedEvent 
{
    HouseId : 1
    Price : 5,250,000
}
```


```
HouseAgentChangedEvent 
{
    HouseId : 1
    AgentId : 2  
}
```

```
HousePriceChangedEvent 
{
    HouseId : 1
    Price : 5,350,000
}
```

These events would be stored against an Aggregate Id and the Aggregate would be rehydrated with the events whenever it is retrieved from the store.


## Awesome

Now that we have a basic understanding of how these 2 patterns work, let’s take a look at what wealth of advantages they can offer us.

Using the CQRS pattern we can vastly improve performance and scalability as models can now be optimised based on the domain and infrastructure. For example given our simple CRUD application, the house model could be the same for updating and retrieving a house entity which can be costly if we want to only return a few fields from our entity as opposed to returning a model with tens of fields.

By applying CQRS to our domain we can now split and optimise the models based on what information we wish to create, update and read. So if we simply wish to return the name and price of a house we can have a tailor-made de-normalized model in our data store for such a demand. And if we want to update the number of rooms within the house then we can create a model specifically for such a scenario. 

With the Event Sourcing pattern we have a guarantee that the state of our application and aggregates are correct. For example,  within our CRUD application we might have the application state capture that the price of a house is £1,200,000. We have no confidence or assurance that this is the correct state, incorrect state, whether this figure was derived from months of fluctuation in housing prices or that the house was put straight on the market with this figure. We must simply put all our faith into the application state and the store that holds this value and trust that this house price is correct. Although with Event Sourcing we have a paper trail of how the application state came to be, by chronologically replaying each event on an entity we can see how the current application state arose and have proof that it is the correct state.

As well as a paper trail we have real-time auditing in our domain, as soon as an aggregate state is changed we are immediately able to report it, due to the event-driven nature we can identify when an event occurs and act upon it. We can also query for events that have occurred in the past across a whole domain and apply some analytics by viewing the trail of events from a user that led up to the event and view the events that followed. We could choose to time travel back in time by rehydrating all our aggregates up to a certain timestamp in their events and continue the domain flow to see how aggregates would respond given different scenarios. 

## Not so awesome

While CQRS + Event Sourcing offer many benefits and total control over the history of your domain, it also comes with just as many disadvantages, the main one being the architectural overhead that comes with having separate read and write layers. This overhead adds additional complexity to your system and also involves careful consideration when making changes to your domain as all modifications to an event must be backwards compatible, otherwise rehydration of an aggregate will result in an incorrect state. 

A common drawback of CQRS is eventual consistency whereby in order to confirm that a command has been successfully processed you must read the query model and confirm that the results are correct, this can be achieved through a number of methods such as continuous polling, websockets or webhooks. Some issues that arise because of this involve adding additional architectural overhead to your UI layer and an inconsistent user experience if the UI is not designed with eventual consistency in mind.

## Conclusion

Given the advantages of CQRS + ES and its drawbacks, the real benefits comes from knowing when to use it and how to use it. If your domain has no need for model reporting and your infrastructure has no desire for eventual consistency then CQRS + ES isn't worth the overhead. Although if you can see how your domain would benefit from a guarantee of the application state, how real-time auditing could enhance your business logic and how your infrastructure could thrive with optimized read times from your data store then CQRS + Event Sourcing would be a strong contender for your project. 
