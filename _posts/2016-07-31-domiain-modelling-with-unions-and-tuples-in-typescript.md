---
layout: post
title:  "Domain Modelling with Unions and Tuples in TypeScript"
date:   2016-07-31
---

Excitingly, in the last few days a great new feature has made it into TypeScript - narrowing of literal union types - which adds a powerful tool for domain modelling to the TypeScript toolbox.

So, what does narrowing of literal union types give us? In [my last post](/2016/7/25/discriminated-union-types-in-typescript) I showed how discriminated union types allowed us to write safer code. The following code will error at compile time because we have missing clauses in our `switch` statement:

<br/>
<script src="https://gist.github.com/mtinning/dd7cc1e4ae183ca6deffdc0ae4e7f090.js">
</script>
<br/>

Now that the same thing can be done with literal union types, our type model can be simplified. We define the `RoomType` type to be the union of string literals describing our room types. Our code now looks like this:

<br/>
<script src="https://gist.github.com/mtinning/3648a8ffd322ec27ee6f404622e104f7.js">
</script>
<br/>

In the above code, I've kept the error in to demonstrate that the compiler will let you know if there is a case you're not handling. The code performs exactly the same job as previously - the difference is that instead of making use of a discriminant property on interfaces, we can use string literals directly and the compiler will check that we are handling all of the cases in the literal union type. The working code looks like this:

<br/>
<script src="https://gist.github.com/mtinning/5e5dc9268ce98a12ae849c3a621484df.js">
</script>
<br/>

Literal union types provide us with a very terse way of defining a property that can take a limited set of values, such as the room types here.

Another tool available to us in TypeScript is the tuple. This provides a minimal way of defining a type that has several properties. It is basically an array with a fixed length, where we know up front exactly what elements and of what types are going to be in the array. We don't get property names or methods like we get in `class`es, but tuples are extremely convenient.

Let's expand our domain model a bit then. We want to define our room to have a `RoomType` and to contain a `Person`. We might then want to explain what these folk are doing in a room. Here's what that might look like:

<br/>
<script src="https://gist.github.com/mtinning/b0cafdf58449e1c51d5a0e4bb4feac10.js">
</script>
<br/>

Now this is OK, but what is up with that syntax? How are we supposed to know that `room[0]` is the room type and `room[1]` is the person? Fortunately, _array destructuring_ comes to the rescue. This allows us to declare our function argument in such a way that we can access the members of the tuple directly from the method. Now we can use `roomType` and `people` in a much more readable way:

<br/>
<script src="https://gist.github.com/mtinning/c95dd7868c8e88832b460dbec4745bdb.js">
</script>
<br/>

Great, this looks a lot better - the function is a lot more readable, it's a lot more obvious what it's doing.

I don't know about you, but my the rooms in my house don't just contain people - they also have clutter in them. Sometimes I like to put the clutter in cupboards. Since clutter can either be in or out of a cupboard, for the sake of this post I'm going to call both of these `Item`s. So my room can contain many items. Here's what my domain model looks like now, in all it's glory:

<br/>
<script src="https://gist.github.com/mtinning/9361e3638350ea93bfb1a060f284ec81.js">
</script>
<br/>

Now I can write something like the following to list all of the clutter in a room, in and out of cupboards. This makes use of the _discriminant property_ of the `Item` type to ensure that all types of items are covered by the `switch` statement ([see my other post](/2016/7/25/discriminated-union-types-in-typescript) for an explanation on this). Another feature of TypeScript which it uses is the `Array` _spread syntax_ - [see this deep dive](https://basarat.gitbooks.io/typescript/content/docs/spread-operator.html) for an explanation on how it works.

<br/>
<script src="https://gist.github.com/mtinning/03a34e7cde584d6cfa9e19bc1de2fcb1.js">
</script>
<br/>

I really like how readable domain models like this are. They're so compact that there's very little to obfuscate the meaning of what you're trying to get across. This is true of functional programming approaches in general - since the whole state is out in the open it is usually clear to see what is going on.

As well as readability, domain models built using unions, tuples and literals also make it very easy to write immutable code. I plan to post on this subject very soon!
