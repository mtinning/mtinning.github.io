---
layout: post
title:  "Discriminated Unions in TypeScript"
date:   2016-07-25
comments: true
description: Digging into discriminated unions in TypeScript - what are they? Why are they useful? And how can they help us write safer code?
---

I was pretty late to jump on the TypeScript bandwagon, but have recently been taking a closer look at it. This was largely inspired by a golden nugget of knowledge that I found at June's [.NET South West Meetup](http://www.meetup.com/dotnetsouthwest/) - TypeScript now has discriminated union types!

<br/>
<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/mttinning">@mttinning</a> <a href="https://twitter.com/dotnetsouthwest">@dotnetsouthwest</a> Awesome isn&#39;t it. Here&#39;s the pull request from Anders himself - <a href="https://t.co/8ea3sv7JYs">https://t.co/8ea3sv7JYs</a></p>&mdash; Joseph Woodward (@joe_mighty) <a href="https://twitter.com/joe_mighty/status/743574655601082368">June 16, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8">
</script>
<br/>

I've enjoyed using discriminated union types ever since I got my head around them looking at F#, which features them extensively. They're a terse, robust alternative to OO for modelling domains. See [this easily digested post](https://medium.com/@cscalfani/goodbye-object-oriented-programming-a59cda4c0e53#.9jhnv6kxu) by Charles Scalfani for reasons why finding alternatives to OO might be a good idea... In short, deep OO hierarchies can be brittle, limiting and too tightly coupled to implementations.

> The problem with object-oriented languages is they’ve got all this implicit environment that they carry around with them. You wanted a banana but what you got was a gorilla holding the banana and the entire jungle.

So what's the alternative when you have a domain in need of a model? Scalfani makes a useful distinction: where OO languages tend towards **_Categorical Hierarchies_** - a square _is_ a quadrilateral _is_ a shape - these types of hierarchies simply do not model the real world very effectively - we don't find many deep hierarchies of categories in the real world. One alternative is **_Containment Hierarchies_**.

A Containment Hierarchy is one where types are defined by their contents. I might decide to model my house with a containment hierarchy - my house contains rooms, including a kitchen, living room, bedrooms etc.. My kitchen contains units. One unit contains drawers. One draw contains cutlery. That cutlery consists of knives, forks, and spoons. There is no need for a categorical hierarchy here at all.

So how does all of this relate to Discriminated Unions in TypeScript? First, let's see how we might model the rooms in my house in TypeScript. Introducing the union type:

<br/>
<script src="https://gist.github.com/mtinning/bc99692b1b098be3a699ea29f151116a.js">
</script>
<br/>

Here, we define each room to be an interface with a property name, which has a different value for each interface. This is called the _discriminant property_ of the interface - the reason will become clear shortly.

We then define the `Room` type. This is the _union_ of each of the interface types - `Kitchen`, `DiningRoom`, `LivingRoom`, `Bedroom`, `Bathroom`. This means that a value that is declared to be of this type can be _any one_ of those interface types _but no other type_. This point is worth emphasising - a value of type `Room` can only be one of the given interface types - as it will allow TypeScript to do some seriously clever stuff!

Now I've got my natty little domain model, I want to do something useful with it! I might, for instance, want to know what I'm doing if I'm in one of those rooms. Now I haven't been spending enough time in my `Bedroom`, `"Sleeping"`, so I bash out some code but get distracted half way through, leaving the following _unsafe_ code:

<br/>
<script src="https://gist.github.com/mtinning/edf90d922f29d778441ba12291cf4f1a.js">
</script>
<br/>

The first `console.log` will compile just fine, and work as expected. In the second case, we get a compile error because I haven't defined a room that looks like `{ name: "study" }` - again, so far so good. In the third case, however, we get some unsafe behaviour - `console.log` results in `undefined`, even though the argument is correct.

Can we do better? Yes - by making the most of discriminated unions.

Languages like F# and Haskell have an [extremely powerful type system](http://programmers.stackexchange.com/questions/279316/what-exactly-makes-the-haskell-type-system-so-revered-vs-say-java) and use algebraic data types to find errors in code that looks like what we have above. This is one of the things that makes working in these languages such a joy.

In TypeScript, we need a small extra step in order to ensure our error is picked up by the compiler: we make user of the [`never` type](https://github.com/Microsoft/TypeScript/issues/3076). In type theory, this is known as a [bottom type](https://en.wikipedia.org/wiki/Bottom_type) - a type which has no values. We can make use of this property to force TypeScript to recognise that we haven't implemented all of the clauses in our `switch` as follows:

<br/>
<script src="https://gist.github.com/mtinning/dd7cc1e4ae183ca6deffdc0ae4e7f090.js">
</script>
<br/>

Great! The TypeScript compiler is now smart enough to figure out that we will reach `default` handler in the `switch`. The reason for this is that we are switching on the discriminant property that I mentioned earlier. TypeScript is able to build up a map of all of the values that this property can take thanks to our union type definition. Since it will not allow an assignment to a value declared to be of type `never`, we get a compile error - exactly what we want.

I've woken up a bit now, so let's fix that:

<br/>
<script src="https://gist.github.com/mtinning/3ec7440c72e4ca53db16ee8dad033bf3.js">
</script>
<br/>

Brilliant, I can now rest happy in the knowledge that all of my cases are handled!

The introduction of this kind of modelling into TypeScript is a huge plus for the language, and one that makes me want to use it every day when building domain models. The definition of types like this is slightly cumbersome at the moment, but I'm looking forward to more improvements on this front, such as [this one](https://github.com/Microsoft/TypeScript/pull/9407)  introducing narrowing of literal union types, which should allow much more terse definition of unions.

I'm loving digging into TypeScript - I think Microsoft and everyone else involved have done a great job with this language! Long may it blaze a trail for JavaScript to follow. I hope to follow up this post with plenty more on the joys of domain modelling with discriminated unions, as well as other TypeScript features like the eminently useful tuple.
