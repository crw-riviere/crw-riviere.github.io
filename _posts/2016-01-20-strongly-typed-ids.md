---
title: Strongly identifying with Strongly Typed Id’s
updated: 2016-01-20 22:16
keywords: Strongly Typed ID, C#, Static Typing, Strongly Typed ID's
description: How to strongly type ID's
---

## Refactoring gone wild

Within any project that involves a repository you're going to be interacting with identifiers, these identifiers are more than likely represented as value types in the form of integers or guid’s and their sole purpose is to identify an entity or aggregate within your domain. 

Within the lifespan of an aggregate multiple identifiers could be created, moved, associated and unassociated with multiple other aggregates and entities within your domain. In between all of this interaction the last thing you would want is for one of your entities to have a case of mistaken identity. All it takes is a little bit of refactoring and a sprinkle of human error and such a scenario can become a reality.

Take for example our Apartment aggregate and simple example:

```cs
public class Apartment : AggregateRoot
{
    public int Id { get; set; }
    public int AddressId { get; set; }
    public int AgentId { get; set; }
    public int AgencyId { get; set; }
    // More apartment properties...
}    
```

Our Apartment model has 3 commonly named identifier properties each represented via a 32 bit integer. The Apartment aggregate exists in a property real estate domain whereby an apartment can have its agent changed multiple times throughout the its timeline. And the method for this action looks like the following:

```cs
public void SetAgent(Agent newAgent)
{
    this.AgentId = newAgent.Id;
    this.AgencyId = newAgent.AgencyId;  
}
```

Given the weakly typed nature of our identifiers it would not be far fetched for our SetAgent() method to be incorrectly refactored (be it due to unfamiliarity of the task, domain, language or just human error) into the following:

```cs
public void SetAgent(Agent newAgent)
{
    // this.AgentId = newAgent.Id;
    this.AgentId = newAgent.AddressId;
    this.AgencyId = newAgent.AgencyId;    
}
```

Following this refactoring task we have accidently assigned the Address Id to our Agent Id and seeing as all our identifiers are integers the compiler is more than happy to let this build and the runtime is also more than happy to let this pass at execution. 

All it takes is a one call to our SetAgent() method and we have a corrupted application state as our Appartment aggregate has the wrong Agent assigned to it. If we were lucky then the Address Id that we assigned would not correlate to any Agent Id within our domain and we would receive a NullReferenceException, if we were not so lucky this issue would go by unnoticed for a while making our domain more corrupt. 

## The Answer

This could've all been prevented if we had used the power of our favourite statically typed language and created Strongly Typed Id’s. All a strongly typed Id is, is an identifier encapsulated in the type of our entity:

```cs
public class AgentId
{
    private readonly int id;
    
    public AgentId(int id)
    {
        this.id = id;
    }
    
    public int Value 
    { 
        get { return this.id; }
    }          
}    
```

Now instead of passing around our identifiers, we pass around our strongly typed identifiers which helps prevents the nightmare of mistakenly assigning identifiers to the wrong entity.

```cs
public class Apartment : AggregateRoot
{
    public int Id { get; set; }
    public AddressId AddressId { get; set; }
    public AgentId AgentId { get; set; }
    public AgencyId AgencyId { get; set; }
    // More apartment properties...
}  
```

While this won’t prevent or solve all your refactoring bugs, it does introduce type safety to your identifiers and improve code readability within your domain. If we wanted to take this further we could also introduce a Generic Id Type, although this simple example should hopefully allow you to see the benefits of Strongly Typed Identifiers and how to get started.    


