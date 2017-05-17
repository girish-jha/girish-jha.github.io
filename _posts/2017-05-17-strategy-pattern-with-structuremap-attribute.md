---
layout: post
title: Using StuctureMap for implementing stratagy pattern 
permalink: StructuremapAttributeForStrategyPatten/
description: Some Description
date: 2017-05-17 09:17:33 +05:30
tags: "some tags here"
---

We all know and love startagy pattern. Startagy pattern helps us instantiating the right implementation of interface at the runtime based on the scenario.
A general way to use startagy pattern in our code will be to create the stratagy class and inject the stratagy class instead of the dependency class.
In the constructor we call the `GetInstance` method of stratagy call to get the right instance.

```csharp
class DependentClass : IDependentClass
{
    private readonly IDependency dependency;
    public ClassName(IStratagy stratagy)
    {
        dependency = stratagy.GetInstance();
    }
}

class Stratagy2 : IStratagy
{
    public SomeType GetInstance()
    {
        if(Session.IsUserLoggedIn)
            return obj1;
        else return obj2;
    }
}

class MyRegistry : Registry
{
    public MyRegistry()
    {
        For<IStratagy>().Use<Stratagy>().Named("String");
        For<IStratagy>().Add<Stratagy2>().Named("String2");
                
        For<IDependentClass>().Use<DependentClass>().Ctor<IStratagy>().Is(_ => _.GetInstance("String2"));
    }
}

```

This is very  clean way of  injecting dependency. But it has it's own downside. The constructor parameter is not clear about the dependecies of the class.
Moreover testing of the class will be little bit clumsy.

To solve these we can take help of `StructureMapSttribute`. With the help of `StructuremapAttribute` we will create an aiitibute and decorate the attribute on the constructor parameter with and instanceKey.

```csharp
    public class InjectStratagyAttribute : StructureMapAttribute
    {
        private IContainer Container => (IContainer)HttpContext.Current.Items["container"];
        public string Key { get; set; }
        public override void Alter(IConfiguredInstance instance, ParameterInfo parameter)
        {
            instance.Dependencies.AddForConstructorParameter(parameter, Container.GetInstance<IStratagy>(Key).GetInstance()); 
        }
    }
```

Now to use this we will decorate the attribute on the constructor parameter:

```csharp
    public class DependentClass
    {
         private readonly IDependency _value;

        public DependentClass([InjectStratagy(Key = "String2")] IDependency value)
        {
            _value = value;
        }

       
    }
```



