---
layout: post
title: Using StructureMap for implementing stratagy pattern 
#permalink: StructureMapAttributeForStrategyPattern/
description: Some Description
date: 2017-05-17 09:17:33 +05:30
tags: "some tags here"
---

We all know and love strategy pattern. Strategy pattern helps us instantiating the right implementation of interface at the runtime based on the state or input parameters provided.
A general way to use strategy pattern in our code will be to create the strategy class and inject the stratagy class instead of the dependency class.
In the constructor we call the `GetInstance` method of stratagy object to get the right instance.

```csharp
class DependentClass : IDependentClass
{
    private readonly IDependency _dependency;
    public DependentClass(IStratagy stratagy)
    {
        _dependency = stratagy.GetInstance();
    }
}

class Stratagy2 : IStratagy
{
    public SomeType GetDependency()
    {
        if(Session.IsUserLoggedIn)
            return obj1;
        else return obj2;
    }
}
```
If we are using StructureMap, the registry for this setup will look something like this:

```csharp
class MyRegistry : Registry
{
    public MyRegistry()
    {
        For<IStratagy>().Use<Stratagy>().Named("Strategy1");
        For<IStratagy>().Add<Stratagy2>().Named("Strategy2");
                
        For<IDependentClass>().Use<DependentClass>()
            .Ctor<IStratagy>().Is(_ => _.GetInstance("Strategy2"));
    }
}
```

This is very  clean way of  injecting dependency. But it has it's own downsides. The constructor parameter is not clearly telling about the dependencies of the class.
Moreover testing of the class will be little bit clumsy.

## Solution

### Through Registry changes

We can solve this by injecting the actual dependency and in the StructureMap registry we can configure how to inject the required dependency by calling the strategy.

So our `DependentClass` will look simply like this:

```csharp
class DependentClass : IDependentClass
{
    private readonly IDependency _dependency;
    public DependentClass(IDependency dependency)
    {
        _dependency = dependency;
    }
}
```

In the registry we will have to get the instance of the actual dependency by calling the strategy and inject it to the dependent class.

```csharp
class MyRegistry : Registry
{
    public MyRegistry()
    {
        For<IStratagy>().Use<Stratagy>().Named("Strategy1");
        For<IStratagy>().Add<Stratagy2>().Named("Strategy2");
                
        For<IDependentClass>().Use<DependentClass>()
            .Ctor<IDependency>().Is(_ => _.GetInstance("Strategy2").GetDependency());
    }
}
```
As the lambda provided in the `.Is(_ => _.GetInstance("Strategy2").GetDependency())` will be called at the runtime, the `GetDependency` method will called only at the runtime and your correct object will be injected in the `DependentClass`.

### Through StructureMapAttribute

`StructureMapAttribute` is an effective way to provide the registration convention. More about it can be read [here](http://structuremap.github.io/registration/attributes/).

We can take help of `StructureMapAttribute`. We will create an attribute and decorate the attribute on the constructor parameter with and instanceKey.

I am making use of `HttpContext.Current.Items` assuming we are wring code for a web application and making use of `nestedContainers`. 
Our attribute will look like this:
```csharp
public class InjectStratagyAttribute : StructureMapAttribute
{
    private IContainer _container => (IContainer)HttpContext.Current.Items["container"];
    public string Key { get; set; }
    public override void Alter(IConfiguredInstance instance, ParameterInfo parameter)
    {
        instance.Dependencies.AddForConstructorParameter(parameter, _container.GetInstance<IStratagy>(Key).GetDependency()); 
    }
}
```
Notice that we are using a public property `Key` in our attribute. This will be assigned by the `DependentClass` to get the right instance of `IStratagy` interface.
The registration for `IStratagy` should be done in normal fashion. 

Now decorate the attribute on the constructor parameter: 
```csharp
public class DependentClass
{
    private readonly IDependency _value;
    public DependentClass([InjectStratagy(Key = "Stratagy2")] IDependency value)
    {
        _value = value;
    }
}

```

With this, whenever we try to get the instance of dependent class, StructureMap will call the `Alter` method of our `InjectStratagyAttribute` before calling the constructor and update rest of the behaviour will follow according to the code we write.

_It's important to note here that using classes and methods of a library(`StructureMapAttribute` in this case) inside our code violates the Dependency Inversion principle and makes it tough if you decide to change the library in future. That being said, `StructureMapAttribute` is a very powerful tool to manage(by bringing one off changes to) the object resgistration convention._



