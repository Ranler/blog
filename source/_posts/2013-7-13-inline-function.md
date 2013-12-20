layout: post
title: Inline Function
date: 2013-7-13
categories: C&CPP
---

### Inline Function in C

Inline functions only make sense when you put them in a ".h" file.
The whole concept is about making the function definition visible to all callers.

```.c
// .h
inline void foo() {...}
```

If you want to make function definition in source file(".c"), you need to do is make the prototype static with:

```.c
// .c
inline static void foo {...}
```

But if you just want to have functions put in place in your "lib.o" compilation unit, forget about all that inline or static and let the compiler do that for you. gcc does that if you switch on optimization:

<blockquote>
    `-finline-small-functions` Integrate functions into their callers when their body is smaller than expected function call code (so overall size of program gets smaller). The compiler heuristically decides which functions are simple enough to be worth integrating in this way.

     Enabled at level `-O2`.
</blockquote>

Reference From: [How do I use but not expose an inline function in a c99 library?](http://stackoverflow.com/questions/5208381/how-do-i-use-but-not-expose-an-inline-function-in-a-c99-library)

### Inline Funtion in CPP


