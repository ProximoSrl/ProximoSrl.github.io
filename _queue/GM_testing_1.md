---
layout: post
title: Always execute your code
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

One of the greatest risk in software development is letting untested code slip in production. Every developer knows that even if code was tested (automatically or manually) bugs can slip in production, this means that you should always execute your code and verify that it behaves as you expected before putting it in production.

## A real example

Suppose we have this code that represents an Id based on an underling String value.

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

Finally we have some services that manage saving and ordering Customers from DB with NHibernate and customers are always returned in a given order. To simplify discussion consider a service that keeps everything in memory.

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

Trivial code isn't it? You can manually test with a database with some test data, and since it is super simple you can let it goes into production but.... you spotted the bug? 

The bug can be really subtle, so let's try to write a simple unit test that exercise the code.

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

This test is green.

You deploy this in production.

You have a bug, some sort of time bomb. This test can highlight the bug.

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

Now examine the differencies, basically it is the same test as before, but now we have two customers with the same TotalOrderAmount, and the test fails with this message.

	System.ArgumentException: At least one object must implement IComparable.

And this is caused by the .ThenBy LINQ instruction, because it try to order objects based on Id property, but it is not a base type, and LINQ does not know how to know if an index is greater than another.

The solution is really trivial, just implement the IComparable interface for the StringId base class and everything is green.

## Lesson learned

This kind of problem is caused by letting slip in production code that was never executed, nor manually, nor with Unit Testing and it constitutes a time bomb in your system that can explode at every moment.

A possible solution is trying to cover all lines of code with Unit Test code coverage, but code coverage is never perfect. In the previous example if you run only the first unit test (the one that succeeds) you got 100% code coverage for CustomerService class, because the code that was reall not covered is inside the implementation of the ThenBy LINQ operator.

Gian Maria.
