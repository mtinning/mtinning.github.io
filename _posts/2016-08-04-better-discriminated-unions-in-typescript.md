---
layout: post
title:  "Better Discriminated Unions in TypeScript with Strict Null Checks"
date:   2016-08-04
comments: false
description: TypeScript 2.0 introduces non-nullable types with strict null checks. This is an awesomely powerful tool for writing safe and correct code - and in addition, it makes our discriminated unions a whole lot better!
---

I've recently been trying out TypeScript 2.0 beta and discovered the extremely useful (you might say essential) `--strictNullChecks` compiler switch. This switch makes all type non-nullable by default, so the compiler will produce an error if a variable is assigned a null value unless _explicitly_ declared as nullable. You can find more information about it on the [pull request](https://github.com/Microsoft/TypeScript/pull/7140) (than man Anders Hejlsberg again!). I wanted to write a quick post here as non-nullable types are a great way to make our discriminated unions even better!

You may have seen in [my previous post about discriminated unions in TypeScript](/2016/07/25/discriminated-unions-in-typescript.html) that we can make use of the `never` type when we switch on discriminant properties to force the compiler to error with something like the following:

<br/>
<script src="https://gist.github.com/mtinning/2bd73049dc07c706f68acfa0d61b099a.js">
</script>
<br/>

Here, because the switch will fall through to the assignment to `_never` in two cases, we get the compile error `Type '"bedroom" | "bathroom"' is not assignable to type 'never'.` This is really useful, but messy - why should we have to assign to a `never` type in order to force the compiler to recognise our daft mistake? The reason is that in the above code, a variable of type `RoomType` can still take the value `undefined`. In the case that the value doesn't match any of the `case`s in the `switch`, the function will return `undefined`, which is still valid.

This brings us to _non-nullable types_. What is a non-nullable type? It's simply a type which is not allowed to take a value of `null` or `undefined`. When we use strict null checks with the example below, the first variable is non-nullable, which means that it's value can never be null or undefined. The other examples _are_ nullable, so they can assume a null value.

<br/>
<script src="https://gist.github.com/mtinning/3c4caab42707bedcc662ad5ab364548a.js">
</script>
<br/>

We can make use of strict `null`/`undefined` checking in our example above to force the compiler to realise that we're missing something, without needing an assignment to a `never` type. Here's what our switch looks like now:

<br/>
<script src="https://gist.github.com/mtinning/6b62f47e3ac64bd402abbec3f1399266.js">
</script>
<br/>

Ahhhh... Doesn't that look better? I can just feel the weight of ugly code lifting from my shoulders! Now I can make this code compile by re-introducing the remaining two `case`s. Unfortunately the compile error that we receive isn't quite as useful in this case - the compiler informs us that `Function lacks ending return statement and return type does not include 'undefined'.`. This is still enough to make us realise that not all paths through our function return a value of type `string`, so we can go back and address this.

We can also make the above compile by declaring the return type to be a union of `string | undefined`, but I don't think we want to do that! Strict null checks are an extremely useful tool for ensuring that code is _correct_, which is one of the core aims of the functional programming paradigm, so I would encourage everyone using TypeScript to use it wherever possible.

This has been a very quick post but I hope it has been useful! If you want to dig in to non-nullable types more then check out the [pull request](https://github.com/Microsoft/TypeScript/pull/7140), which has some great details, or even better give it a shot yourself! I'm getting more and more excited about the possibilities that the new TypeScript language features open up for writing clean, concise and safe code.
