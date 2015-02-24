---
layout: post
title: Test context
description: "Why test fixture is not so simple to setup"
tags: [testing, csharp]
author: gianmariaricci
image:
  feature: anvil.jpg
  credit: Morguefile
  creditlink: http://www.morguefile.com/
comments: true
share: true
---

## The mytical perfect Unit Test Fixture

After years of unit testing I'm really convinced that the real complexity in writing a good Unit Test is **setting up a strong [test fixture](http://xunitpatterns.com/test%20fixture%20-%20xUnit.html)**. Here is an example: in *Jarvis* we have a test that runs fine in R# test runner, in Nunit GUI test runner, in Team City build test runner, but is red when it runs inside *Visual Studio Test Runner* with Nunit Test adapter.

After a brief investigation we discovered that the cause was a different handling of System.Threading.Thread.CurrentPrincipal object within the various test runners. Here is our situation

## What is happening inside VS?

Here is a stupid test that confirmed our suspicions.

{% highlight csharp %}

[TestFixture]
public class verify_principal
{
    [Test]
    public void verify_principal_is_empty()
    {
        Assert.That(System.Threading.Thread.CurrentPrincipal.Identity.Name, Is.Null.Or.Empty);
    }
}

{% endhighlight %}

This simple test verify that **the name of the identity associated to the thread used to run the test is null or empty**. This test runs green with every test runner, except with Nunit test adapter in Visual Studio. 

When test is executed in Visual Studio, CurrentPrincipal identity is a valid Windows Identity and **it's equal to your current Windows user**, and this makes some of our test fails when executed inside visual studio.

## Fixture Fixture Fixture

The test that failed is this one

{% highlight csharp %}

[Test]
public void raising_events_without_user_context_should_throw()
{
	var router = new AggregateRootEventRouter<SampleAggregate.State>();
	var ex = Assert.Throws<InvalidPrincipalException>(() =>
		{
			router.Dispatch(new SampleAggregateCreated(new SampleAggregateId(1)));
		});
}

{% endhighlight %}

This test verifies that: *if you try to raise a Domain Event without user context, the engine should throw an exception*. Clearly this test fails with Visual Studio Nunit test adapter, because CurrentPrincipal.Identity is equal to current user but it runs perfectly with other test runners.

Actually we can blame VS Test Runner for this behaviour, but **the real cause of the [erratic test](http://xunitpatterns.com/Erratic%20Test.html) is due to a not perfect test fixture**. If the test should test **how our engine behave if no user context is set, we should not rely on the fact that some test runner actually runs the test without an identity set in CurrentPrincipal**. 

If you believe that the real culprit is VS Test Runner, consider what happened if, before this text executes, anohter test set a valid Principal in Thread.CurrentPrincipal property and does not restore it to original value. You will end with a failure in all test runner, but the failur will occour only if the other test is executed before this one. What you have is

1. You run all test, this test is red
2. You run the test alone, test is green (because the test that changes CurrentPrincipal is not run)
3. You run all test again, test is red :|
4. You change the name of some test and the test returns green (maybe because the test that changes CurrentPrincipal is run after the erratic test

This is called [erratic test](http://xunitpatterns.com/Erratic%20Test.html), because it is a **test that can fail due to other external conditions**. Such kind of tests bring real pain when they fails, because you need to understand if the test fails because the underling assumption is wrong (you have a bug) or if it's an external situation that makes it fail (the bug is in the test).

The above test is better refactored in this way:

{% highlight csharp %}

[Test]
public void raising_events_without_user_context_should_throw()
{
    var old_identity = System.Threading.Thread.CurrentPrincipal;
    System.Threading.Thread.CurrentPrincipal = new GenericPrincipal(new GenericIdentity(""), new String[] {});
	var router = new AggregateRootEventRouter<SampleAggregate.State>();
	var ex = Assert.Throws<InvalidPrincipalException>(() =>
		{
			router.Dispatch(new SampleAggregateCreated(new SampleAggregateId(1)));
		});
    System.Threading.Thread.CurrentPrincipal = old_identity;
}

{% endhighlight %}

This test is superior for a number of reasons

- It explicitly clears the user context before the test.
- The test is explicitly telling you that the engine is using System.Threading.Thread.CurrentPrincipal to check current user.
- This test is stable and runs correctly in every test runner.
- This test is stable even if a previous test set a valid CurrentPrincipal in current Thread for some other reason.

## Lesson learned

To create stable and reproducible tests, **you need to be sure that in the Fixture Set Up phase of the test, the system is the desired state for the test to start**. A typical error is believeing that desired fixture was already setup for you by some other "entity" like the test runner or other tests.

For every test, you should always ask yourself if you correctly setup a good Fixture or if the test is based to some assumption that does not depends on the test itself (runner, previous tests, etc).
Gian Maria.
