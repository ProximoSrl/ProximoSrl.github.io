---
layout: post
title: Always execute your code before moving it in production
description: "Why testing and running your code is fundamental in software development"
tags: [testing, csharp]
author: gianmariaricci
image:
  feature: anvil.jpg
  credit: Morguefile
  creditlink: http://www.morguefile.com/
comments: true
share: true
---

## Code that was never executed in production

One of the greatest risk in software development is **letting untested code slip in production**. The general rule is: *you should always execute your code and verify that it behaves as you expected before moving to production*. Clearly we should prefer automated testing over manual testing, but at least we need to be sure that every part of the code was at least executed once before promote it to production.

## A real example

Suppose we have this code that represents an Id based on a simple string.

{% highlight csharp %}

	public class StringId
	{
	    public String Value { get; set; }
	
	    public static implicit operator String(StringId id)
	    {
	        return id.Value;
	    }
	
	    public override bool Equals(object obj)
	    {
	       //....
	    }
	}

{% endhighlight %}

And we have also a Customer class based on this type of Id

{% highlight csharp %}

    public class CustomerId : StringId
    {
        public CustomerId(String rawValue)
        {
            this.Value = rawValue;
        }
    }

    public class Customer
    {
        public CustomerId Id { get; set; }

        public String Name { get; set; }

        public String Surname { get; set; }

        public Int32 Age { get; set; }

        public Double TotalOrderAmount { get; set; }
    }

{% endhighlight %}

Finally we have some services that handle accessing data in Sql Database with NHibernate and we have a function that return all customers. To simplify this discussion lets consider a service that keeps data in memory.

{% highlight csharp %}

    public class CustomerService
    {

        public Dictionary<CustomerId, Customer> _inMemoryRepo = new Dictionary<CustomerId, Customer>();

        public void AddCustomer(Customer customer)
        {
            _inMemoryRepo[customer.Id] = customer;      
        }

        public IEnumerable<Customer> GetAllCustomers()
        {
            return _inMemoryRepo.Values
               .OrderBy(c => c.TotalOrderAmount)
               .ThenBy(c => c.Id);
        }
    }

{% endhighlight %}

Trivial code isn't it? Since it is super simple probably you think that there is no need for testing. If you believe that the above code is simple and correct, you missed a bug. 

Let's try to write a simple unit test that exercise the code.

{% highlight csharp %}

[TestMethod]
public void verify_get_all_customers()
{
    CustomerService sut = new CustomerService();
    sut.AddCustomer(new Customer() { Id = new CustomerId("c1"), TotalOrderAmount = 10.0d });
    sut.AddCustomer(new Customer() { Id = new CustomerId("c2"), TotalOrderAmount = 5.0d });
    sut.AddCustomer(new Customer() { Id = new CustomerId("c3"), TotalOrderAmount = 16.0d });

    var result = sut.GetAllCustomers();
    result.Select(c => c.Id.Value)
        .Should()
        .Have.SameSequenceAs(new [] { "c2", "c1", "c3" });
}

{% endhighlight %}

This test is oversimplified, but it represent a similar test that you can write if the service access database trough repository interface. 

**This test is green, but you have a bug!**

The real problem shows up only with certain test data. This new test can highlight the bug.

{% highlight csharp %}

        [TestMethod]
        public void verify_get_all_customers_with_same_amount()
        {
            CustomerService sut = new CustomerService();
            sut.AddCustomer(new Customer() { Id = new CustomerId("c1"), TotalOrderAmount = 16.0d });
            sut.AddCustomer(new Customer() { Id = new CustomerId("c2"), TotalOrderAmount = 10.0d });
            sut.AddCustomer(new Customer() { Id = new CustomerId("c3"), TotalOrderAmount = 16.0d });

            var result = sut.GetAllCustomers();
            result.Select(c => c.Id.Value)
                .Should()
                .Have.SameSequenceAs(new[] { "c2", "c1", "c3" });
        }

{% endhighlight %}

Now examine the differencies: basically it is the same test as before, but **now we have two customers with the same TotalOrderAmount**, and the test fails with this message.

	System.ArgumentException: At least one object must implement IComparable.

And this is caused by the .ThenBy LINQ instruction. This instruction *tries to order objects based on Id property, but since it is a complex type, LINQ does not know how to order based on CustomerId class*. The previous test never exercised the condition specified by the .ThenBy

The solution is really trivial, just implement the IComparable interface for the StringId base class and everything is green. The real problem happens when *this error is discovered in production and you need to do some quick fix*.

## Lesson learned

This kind of problem is caused by letting slip in production code that was never executed, nor manually, nor with Unit Testing. This kind of code constitutes a time bomb in your system that can explode in every moment.

A possible solution is trying to cover all lines of code with Unit Test code coverage, **but code coverage is no perfect. In the previous example if you run only the first unit test (the one that succeeds) you got 100% code coverage for CustomerService class**. This happens because the code that was really not covered is inside the implementation of the ThenBy LINQ operator.

This can be a classic example on how difficult is writing Unit Tests that are able to discover every bug in your code.

Gian Maria.
