---
layout: page
title: About
permalink: /about/
---

## Contents
{:.no_toc}

* This will be replaced by the table of contents
{:toc}

## GitHub (2021 - present)

I am a Principal Researcher at GitHub. My interests are in developer tools, programming languages,
and software engineering. My work is at the border between research and practice -- applying
cutting-edge research to build tools that help developers be more productive and write better code.

## Google (2011-2021)

Previously, I was a Tech Lead/Manager at Google on the Java Team. I led a team of 9
engineers and worked on a number of tools for Java developers:

### Error Prone

[Error Prone](https://github.com/google/error-prone) is Google's static analysis framework for Java.  It is used as a bug detector, a large-scale refactoring tool, and a large-scale code analysis tool. Error Prone is integrated into Google's build system and code analysis platform and is thus used by every Java engineer at Google.  

Error Prone is used by many other companies, including Uber and Square.  Uber built their null checker, [NullAway](https://github.com/uber/NullAway), on Error Prone.

### Bazel

[Bazel](https://bazel.build/) is Google's build system. My team owned the Java compiler(s) used in
Bazel and the Java rules implementation. We disentangled the Java compiler from Bazel itself, took
ownership, and made many improvements to build speed via tools like
[Turbine](https://github.com/google/turbine), a header compiler for Java.

### google-java-format

[google-java-format](https://github.com/google/google-java-format) is Google's Java formatter.
Before google-java-format, there was no formatter which correctly implemented
[Google's Java Style Guide](https://google.github.io/styleguide/javaguide.html). We built one which
grew organically to be used by ~all Google Java developers.

Notably, google-java-format is
[not configurable](https://github.com/google/google-java-format/wiki/FAQ#i-just-need-to-configure-it-a-bit-differently-how),
which enabled us to deliver excellent formatting results with reduced engineering effort.

### Java 8+ Support for Android

Older Android devices do not support Java 8+ language features (e.g., lambdas) or APIs. We built a tool,
[Desugar](https://github.com/bazelbuild/bazel/tree/master/src/tools/android/java/com/google/devtools/build/android/desugar),
that rewrites Java 8+ bytecode into Java 7 bytecode and
[repackages Java 8+ libraries](https://github.com/google/desugar_jdk_libs) so that they can be used
on these older devices.

Initially Desugar was supported only for internal Google developers, but we
worked with [Android Studio to bring desugaring to all Android developers](https://developer.android.com/studio/write/java8-support).

---

I also worked on two machine learning for code efforts:

### DeepDelta

DeepDelta was our first attempt to automatically repair broken builds.  We
collected a giant dataset of all Java builds at Google and the diffs between them. We processed
these into "resolution sessions," sequences of builds that went from failed to successful, and
extracted the AST diffs that caused them to succeed. We then trained a neural machine translation model to
predict these diffs with 50% accuracy.

This was joint work among my team and two visiting
researchers, [Ali Mesbah](http://ece.ubc.ca/~amesbah/) from UBC and
[Andrew Rice](https://www.queens.cam.ac.uk/professor-andrew-rice) from Cambridge. 

Please see our [FSE paper](https://research.google/pubs/pub48350/) for details.

### Graph2Diff

Graph2Diff is an extension of our prior DeepDelta work. DeepDelta's diff DSL did not provide sufficient
information to turn the model output back into source code in all cases. Graph2Diff improved the
DSL, enabling us to generate actual textual diffs to repair the broken programs. In addition, and more importantly, it
applied a graph neural network approach to the build repair problem, doubling accuracy over
DeepDelta. Graph2Diff has been deployed and is currently delivering ML-powered build repairs to Google
developers.

Graph2Diff was a collaboration among my team, the ML for code team in Google Brain, and
visiting researcher [Andrew Rice](https://www.queens.cam.ac.uk/professor-andrew-rice) from
Cambridge.

Please see [our paper on ArXiv](https://arxiv.org/abs/1911.01205) for details.

## Tufts University (2004-2011)

I received my PhD in computer science from Tufts University. My advisor was
[Sam Guyer](http://www.cs.tufts.edu/~sguyer). My work focused on garbage collection and dynamic
analysis to detect memory problems.

## Publications

Please see my [Google Scholar page](https://scholar.google.com/citations?hl=en&user=E2AR0HUAAAAJ)
for a list of publications.

## Contact

* [LinkedIn](https://www.linkedin.com/in/eddie-aftandilian-772b267/)
* [Twitter](https://twitter.com/eaftandilian)
* [Email](mailto:aftandilian@gmail.com)