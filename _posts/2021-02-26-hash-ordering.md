---
layout: post
title: Hash ordering and Hyrum's Law
---

A Google engineer named [Hyrum Wright](http://www.hyrumwright.org/) made an observation that came to
be known as [Hyrum's Law](https://www.hyrumslaw.com/):

> With a sufficient number of users of an API, it does not matter what you promise in the contract:
> all observable behaviors of your system will be depended on by somebody.

What are the consequences of Hyrum's Law? Well, it means that anyone trying to do a large-scale
migration is going to tear their hair out discovering that their seemingly simple migration is way
harder than expected, and that despite putting that giant warning in your docs not to depend on some
implementation-specifc behavior, your clients are going to depend on it anyway.

Can't you just break them? They're violating the spec!

![Lisa Simpson looking at a sign that says, "Keep out... or enter. I'm a sign, not a cop"](../images/sign-not-a-cop.jpeg)

No, you can't break them. Let's consider a concrete example -- hash iteration order.

## Hash iteration order

Generally, when you iterate over the keys and values in a hash table, the iteration order is
unspecified and implementation dependent. For example, the Javadoc for `java.util.HashMap` states,
"This class makes no guarantees as to the order of the map; in particular, it does not guarantee
that the order will remain constant over time." In practice, `HashMap` iteration order is stable for
several years but is sometimes perturbed by major Java version updates.

Because the iteration order remains stable for years at a time, users end up depending on it,
despite the warning in the Javadoc. Now let's say you're on the Java team at a large company with a
monorepo, and you're responsible for updating the monorepo from version N of Java to version N + 1.
There may be thousands of places where users depend on the hash iteration order.

## Example problems

Why would anyone want to depend on hash iteration order? Generally they don't, and the dependency is
accidental. We encountered a handful of patterns.

### Order-dependent tests

Unit testing frameworks such as JUnit previously did not specify the order in which tests are
executed, and users were supposed to write their tests so that each test was independent from all
others. This makes sense if the data structure underlying test discovery and execution is a hash
table.

If test execution order is stable in practice, users will end up depending on it. For example, they
might run some setup code in one unit test but not the others, and it will happen to work if that is
always the first test to be executed.

### Over-specified test assertions

Consider a unit test for the following method:

```java
public static Map<Integer, String> doSomething() {
    // some implementation here...
    return new HashMap<Integer, String>(contents);
}
```

Someone might write the unit test like so:

```java
@Test
public void doSomethingReturnsCorrectResult() {
    var actual = Foo.doSomething();
    assertThat(actual).containsElementInOrder(foo, bar, baz);
}
```

Clearly, that's incorrect -- the elements don't have to be in that order, but for now they are and
the test passes.

## Approaches

How do you deal with this so you can complete the version ugprade?

### File bugs

One approach might be to file thousands of bugs against the teams that own this broken code and ask
them to fix the problem. Their code is clearly incorrect, and it's a reasonable request for them to
fix it. The problem with this approach is that it doesn't fix the underlying problem, and the next
time you do a Java version upgrade, you're going to have to file thousands of bugs again.

### Specify the behavior

[JUnit took this approach](https://github.com/junit-team/junit4/wiki/Test-execution-order) when they
were affected by the change in hash iteration order from Java 6 to 7. Instead of trying to get all
their users to fix their tests, they simply specified test execution order in a way that matched the
Java 6 behavior.

### Defensive randomization

Hyrum's Law talks about _observable_ behaviors. If you make it impossible to observe the behavior,
no one can depend on it. So one option is to randomize the behavior. This is what we did. We[^1]
modified our JDK to randomize hash iteration order. We still had to file bugs against teams whose
tests broke as a result of this change, but once all those bugs were fixed, the problem was solved
permanently.

[^1]: Specifically, Kurt Kluever and Martin Buchholz

We didn't invent this idea. Python and Go were both doing this before us, and we modeled our
approach on Python's. Like Python, we used an environment variable to provide a random seed which is
used to randomize per process. That is, hash iteration order is fixed for a given invocation, but
changes from invocation to invocation. The environment variable can be used to fix the seed and
reproduce bugs.

## Conclusions

Hash iteration order is a great example of Hyrum's Law -- if the iteration order is stable in
practice, users will depend on it no matter what the documentation says. The best way to fix this is
to randomize the iteration order, making it impossible for users to depend on it. Alternatively, you
can specify the order if underspecifying it isn't buying you anything.

The people who have to deal with this problem might get frustrated that _people are doing it
wrong_!!! And yes, people are in fact doing it wrong. But even the best software engineers make
mistakes. Blaming them doesn't solve the problem; making it impossible to make the mistake does.

## Related work

Python had hash randomization as an opt-in option since at least 2010, and it was enabled by default
in Python 3.3 in 2012.

Go has had hash randomization on by default since go1 in 2012.

Gyori et al. had a paper at FSE 2016 called
"[NonDex: A Tool for Detecting and Debugging Wrong Assumptions on Java API Specifications.](https://dl.acm.org/doi/10.1145/2950290.2983932)"
It addressed this problem of underspecified APIs, and hash iteration order was one of the specific
issues it addressed.
