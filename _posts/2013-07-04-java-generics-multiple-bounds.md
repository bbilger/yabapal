---
id: 167
title: 'Java Generics - Multiple Bounds'
date: 2013-07-04T21:48:10+00:00
author: Björn Bilger
permalink: /2013/07/04/java-generics-multiple-bounds/
categories:
  - Java
tags:
  - Generics
  - Java
  - Multiple Bounds
---
Even though I have written and read a lot of Java code during the last years, I recently read an interesting article about a feature I did not know about and also haven't seen yet: Multiple Bounds.

Generics in Java are a quite essential feature and most use them in order to avoid compiler warnings. If you are using them for more than just parameterizing - for example - list objects, then things might get a little tricky. Still it's worth the effort since you can reduce code complexity and duplicity - I really loathe the latter one.

One of those tricky things with generics is to make sure that a type implements two or more interfaces. If the classes implementing those interfaces are your own then this is not too much of a problem since you can form a new interface which implements all required interfaces. If the classes are not your own ones then you'll have a problem to avoid code duplicity.

<!--more-->

Let's say we want to have a method that appends a newline character to either a StringBuilder or StringBuffer. Both classes implement CharSequence and Appendable but without multiple bounds you won't have a chance to support both with one method and you'll end up duplicating your code.

``` java
public void finishLine(StringBuffer str) {
  int len = str.length();
  char newline = '\n';
  if (len == 0 || str.charAt(len-1) != newline) {
    str.append(newline);
  }
}
public void finishLine(StringBuilder str) {
  int len = str.length();
  char newline = '\n';
  if (len == 0 || str.charAt(len-1) != newline) {
    str.append(newline);
  }
}
```

Even though you could reduce code duplicity slightly with some refactoring, this is not necessary at all. Check this example using multiple bounds:

``` java
public <T extends Appendable & CharSequence> void finishLine(T str) {
  int len = str.length();
  char newline = '\n';
  if (len == 0 || str.charAt(len-1) != newline) {
    try {
      str.append(newline);
    } catch (IOException e) {
      throw new RuntimeException(e);
    }
  }
}
```

(BTW: no idea why they designed Appendable's append method to throw an IOException...)

There's just one rule for those bounds:

> Only one of the bounds can be a class and this one must be specified first.

So those examples won't compile:

  * `<T extends Interface1 & Class>` (use: `<T extends Class & InterfaceX & ... & InterfaceN>`, instead)
  * `<T extends Class1 & Class2 & ...>`  (this simply does not work)

With this neat *trick* it is possible to

  1. Avoid code duplication.
  2. Reduce code complexity.
  3. Avoid intermediate interfaces, if the classes are your own.
  4. Avoid implicit requirements like "a caller has to call two or more methods consecutively on an interface". Avoiding this is somewhat essential in order to design good interfaces!

Hope this helps!
