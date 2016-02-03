---
title: What's Dependency Inversion got to do with Dependency Injection
updated: 2016-01-28 22:10
---

Dependency injection is prevalent throughout many frameworks and libraries these days, and rightly so. The design pattern helps decouple classes, improves testing and reduces high level modules being dependant on lower level modules.

While dependency injection ties in well with inversion of control and inversion of control solves the dependency inversion principle, their ideas can blur into one and another which can be confusing when learning the topic.

In order to help understand these patterns and principles it's easier to segregate them and understanding their sole purpose and what they try to solve and then it becomes clearer how they all fit together.

## Dependency Inversion Principle

This principal is the last of the holy S.O.L.I.D principles and is based on 2 rules:

- High-level modules should not depend on low-level modules. Both should depend on abstractions.

- Abstractions should not depend on details. Details should depend on abstractions.

In short this means that the internal functionality of a class should not be dependant on concrete implementations of classes but that they should be dependant on an abstraction of those concrete classes instead. For example if we had a class that was dependant on a postcode service (service to retrieve an address based on a postcode) and we wanted to implement it without adhering to the dependency Inversion principle then it might look something like the following.

```cs
public class HouseAggregate
{
    private UKPostcodeService ukPostcodeService;

    public HouseAggregate() {}
    
    public string GetAddress(string postcode)
    {
        this.ukPostcodeService = new UKPostcodeService();
        return this.ukPostcodeService.GetAddress(postcode);
    }
}
```

As you can see our aggregate is completely dependant on the UKPostcodeService. This is restrictive as we can't ever change the type of our post code service without modifying the code. If we wanted to use an Australian postcode service instead we would have to rewrite our aggregate in order to do so. It also makes it difficult to unit test our aggregate as we can't moq out our postcode service. These are the issues that this principles tries to solve by advising that all high level modules (the House aggregate) should not be dependant on low level modules (the UK postcode service).

## Inversion of Control

Now that we know what the issues are, the Inversion of Control design pattern is how we go about solving them and implement the guidelines that the Dependency Inversion Principle recommend. Given our postcode service example, in order for our low level module (UKPostcodeService) to be dependant on an abstraction we need give it an interface (IPostcodeService) and use the interface as the dependency.

```cs
public class HouseAggregate
{
    private IPostcodeService postcodeService;

    public HouseAggregate() {}
    
    public string GetAddress(string postcode)
    {
        this.postcodeService = new UKPostcodeService();
        return this.PostcodeService.GetAddress(postcode);
    }
}
```

This has inverted the flow of control as our House Aggregate is now not dependant on a concrete implementation of a low-level module (UKPostcodeService) but instead dependant on an abstraction (IPostcodeService). 

## Dependency Injection

Even though we have adhered to the Dependency Inversion Principle, we can go one step further and remove the need for the low level module to be instantiated within the dependency class, and this is where the Dependency Injection design pattern comes in.

Dependency Injection is the process of decoupling classes and moving the class instantiation process out of the dependant class and into a separate Builder object<sup>[1](#footnote-one)</sup>. The Builder object is responsible for instantiating (injecting) a class based on a given interface or type. The injection process can happen either through the constructor parameters of a class (constructor injection) or through the setter handler on a property (setter injection).

If we were to apply constructor injection to our Inversion of Control example then the class constructor would accept the IPostcodeService interface and the Builder object would be responsible for injecting the correct concrete class as soon as our House Aggregate was instantiated.

```cs
public class HouseAggregate
{
    private IPostcodeService postcodeService;
    
    public HouseAggregate(IPostcodeService postcodeService) 
    {
        this.postcodeService = postcodeService;
    }
    
    public string GetAddress(string postcode)
    {
        return this.postcodeService.GetAddress(postcode);
    }
}
```

This allows us to easily swap out the concrete implementation of our postcode service based on the business logic or test case scenarios, it decouples the high level modules from our low level modules and makes introducing new concrete implementations of our classes easy, with minimum changes to our code. All while adhering to the Inversion of Control design pattern and following the Dependency Inversion Principle.

___
<a name="footnote-one"><sup>1</sup></a> The task of the Builder object is a complex one involving reflection and is ideally left to a framework or library such as Ninject, Unity or Autofac. 