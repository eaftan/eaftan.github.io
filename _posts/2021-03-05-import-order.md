---
layout: post
title: "Changing import order: What could go wrong?"
---

One of our team's responsibilities at Google was to make style decisions and enforce them for Java.
That doesn't mean that we made these decisions unilaterally. There was an email list where anyone
could send a proposed change to the style guide, and there would be a healthy debate, and then a
group of "Java stylists" would make the actual decisions. The "Java stylists" came from across
Google, not just the Java team.

Until 2016, the [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html)
specified a pretty complicated set of rules for ordering import statements:

> Import statements are divided into the following groups, in this order, with each group separated
> by a single blank line:
>
> 1. All static imports in a single group
> 1. `com.google` imports (only if this source file is in the `com.google` package space)
> 1. Third-party imports, one group per top-level package, in ASCII sort order for example:
>    `android`, `com`, `junit`, `org`, `sun`
> 1. `java` imports
> 1. `javax` imports
>
> Within a group there are no blank lines, and the imported names appear in ASCII sort order.

It turned out to be impossible to implement these rules correctly in all the various editors people
used at Google. Furthermore, what's the point of such a complex ordering? Is this helping anyone, or
just causing friction?

We decided to simplify the rule to the following:

> Imports are ordered as follows:
>
> 1. All static imports in a single block.
> 1. All non-static imports in a single block.
>
> If there are both static and non-static imports, a single blank line separates the two blocks.
> There are no other blank lines between import statements.
>
> Within each block the imported names appear in ASCII sort order.

OK, so we changed the rule, great. Now, what do we do about the existing Java files in the depot? We
had two options:

- **Don't change existing files and let a file's imports get re-sorted the next time someone touches
  the file.** This would be less work for us, but irritating for our users because they would get
  linter errors even if they hadn't touched the imports.
- **Re-sort all the imports in all the Java files in the depot ourselves.** This seemed pretty
  doable. Re-sorting imports is obviously semantics preserving, so it should be totally safe. The
  edit to each file is independent of every other edit. And actually doing the re-sorting requires
  only a single file (compared to refactorings where you need a classpath to be able to resolve
  symbols, etc.).

We decided to re-sort all the imports ourselves. I argued against this, since the problem would
eventually resolve itself as developers edited every file in the depot. Greg Kick, who made most of
these changes, and Liam Miller-Cushon, who built the tooling and helped with some of the changes,
convinced me otherwise.

## What could go wrong?

Re-sorting imports is one of the safest changes you can make to a Java source file. However,
[Hyrum's Law](https://hyrumslaw.com) predicts that if you make even the safest change at large
enough scale, something will go wrong. So what went wrong when we did this?

1. **Scale limitations in our tools.** This was one of the largest large-scale changes ever made at
   Google, and we ran into tool limitations. For example, we hit the limit on the size of a commit.
   It was easy enough to work around these limitations by splitting the commit into smaller chunks
   that the tools could handle.

1. **Merge conflicts.** You could probably predict this one. In these cases, we reverted the
   proposed change, regenerated it, and tried again.

1. **Golden file tests.** Suppose you work on a tool that generates Java source code. A typical way
   to write tests is to compare the actual output with golden source files. Often the golden source
   files are not actual `.java` files but either inline strings in the test cases or text files with
   some other extension like `.expected`. These tools typically depend on the Java formatter to
   massage the output into something that complies with the style guide. So when you change the
   formatter to output the new sort order, turns out you break all these golden file tests.

1. Actual breakages! It turns out that you actually can break a compilation by re-sorting the
   imports. Consider the following source files:

   z/Z.java

   ```java
   package z;

   public class Z {}
   ```

   a/A.java

   ```java
   package a;

   import z.Z;
   import a.A.B.C;

   public class A {
       class B extends Z {
           class C {}
       }
   }
   ```

   `a/A.java` is a bit weird as it imports the symbol `a.A.B.C` from the same file, but that's legal
   and it compiles successfully. But if you flip the order of the imports, compilation fails!

   ```shell
   $ javac a/A.java
   a/A.java:7: error: cannot find symbol
       class B extends Z {
                       ^
     symbol:   class Z
     location: class A
   1 error
   ```

   Why does this happen? It's a [compiler bug](https://bugs.openjdk.java.net/browse/JDK-7101822).
   javac attempts to resolve the imports in order. It first attempts to resolve `a.A.B.C`, which
   triggers symbol completion for `class B extends Z`. `Z` hasn't been imported yet, so compilation
   fails. FWIW, this was fixed in Java 9.

## Conclusions

Even trivial changes can be difficult at large enough scale. This ended up taking a few months as a
background task.

But it's pretty neat that this was possible. Google's monorepo makes the easy things hard, and the
hard things possible.
