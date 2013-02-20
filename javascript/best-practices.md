# JavaScript Coding Standards and Best Practices

## Introduction:

This is the JavaScript best practices and standards guide for PULSE, the internal agency responsible for the EA SPORTS family of websites.

Although this guide is primarily centered around front-end JavaScript, most of the practices here are equally applicable to back-end JavaScript in node.js.

Author:  
Steve Kwan, Project Lead, EASPORTS.com  
Electronic Arts  
[mail@stevekwan.com](mailto:mail@stevekwan.com)  
[http://www.stevekwan.com/](http://www.stevekwan.com/)

## If You're A Javascript Noob, Read These:

### [JavaScript: The Good Parts, by Douglas Crockford](http://www.amazon.ca/JavaScript-Good-Parts-Douglas-Crockford/dp/0596517742)
The good news is, this book is very short.  The bad news is, it's very dense and will likely require a lot of thinking and re-reading.  But The Good Parts is, in my mind, the definitive guide on how an engineer should approach JavaScript.  If you take nothing else aware from this article, please take away this: read The Good Parts.

### [Constructors considered mildly confusing, by Joost Diepenmaat](http://joost.zeekat.nl/constructors-considered-mildly-confusing.html)
If you've ever been confused by how constructors and the prototype property work in JavaScript, this article does a very detailed job of explaining how they work.

### [jQuery API](http://api.jquery.com/)
jQuery is essential for the modern JavaScript developer.  I recommend you consult their API docs to learn all the library has to offer.

## If You're A Javascript Ninja, Read These:

### [The Surprisingly Elegant JavaScript Type Model, by Kannan Vijayan](http://vijayan.ca/blog/2012/02/21/javascript-type-model/)
JavaScript's type model is deceptively complicated, but this article does the best job I've ever seen of explaining it.

### [JavaScript Object vs Function experiment, by Steve Kwan](https://github.com/stevekwan/experiments/blob/master/javascript/object-vs-function.html)
If you liked Kannan's article, you might like this.  It's a really trippy demonstration of the relationship between the Object and Function objects.

### [Partial Application in JavaScript, by Ben Alman](http://benalman.com/news/2012/09/partial-application-in-javascript/)
Currying and partial function application are important, but advanced, concepts within functional languages like JavaScript.  When you're ready to truly master these concepts, this article is the best I've seen.  I learned a ton reading it.

### [JavaScript constructor vs prototype experiment, by Steve Kwan](https://github.com/stevekwan/experiments/blob/master/javascript/constructor-vs-prototype.html)
If you _really_ want to understand constructor vs prototype, here's an in-depth example I came up with.

## Common Javascript Gotchas:

Before we get started on the best practices, let's talk about some of the JavaScript 101 problems that noobs to the language encounter.  If you're relatively unfamiliar with JavaScript, I highly recommend you read this section - it'll save you a lot of pain down the road.

### The `var` keyword: what exactly does it do?
`var` declares a variable.  But...you don't need it.  Both these will work:

    var myFunction = function()
    {
        var foo = 'Hello'; // Declares foo, scoped to myFunction
        bar = 'Hello';     // Declares bar...in global scope?
    };

But you should never, EVER use the latter.  See the comment for the reason why.

If you ever forget the var keyword, your variable will be declared...but it will be scoped __globally__ (to the `window` object), rather than to the function it was declared in.

This is extremely bizarre behaviour, and really a stupid design decision in the JavaScript language.  There are no pragmatic situations where you would want local variables to be declared globally.  So remember to __always__ use `var` in your declarations.

### Why are there so many different ways to declare a function?  What are the differences?
There are three common ways to define a function in JavaScript:

    myFunction = function(arg1, arg2) {};     // NEVER do this!
    function myFunction(arg1, arg2) {};       // This is OK, but...
    var myFunction = function(arg1, arg2) {}; // This is best!

The first option is bad because it declares a function like a variable, but without the `var` keyword.  Read the above point to understand why that's bad...it gets created as a global function!

The second option is better, and is properly scoped, but it leads to certain syntax problems once you get into closures.

The third option is almost always the best, causes the fewest problems, and is syntactically consistent with the rest of your variables.

### The `this` keyword: how does it behave?
`this` in JavaScript does __not__ behave the way you would expect.  Its behaviour is very, very different from other languages.

If you are new to JavaScript, I suggest avoiding the `this` keyword until you get comfortable with the basics of the language.  But when you are comfortable, here's an explanation...

In more sane languages, `this` gets you a pointer to the current object.  But in JavaScript, it means something quite different: it gets you a pointer to the __calling context__ of the given function.

This will make much more sense with an example:

    var myFunction = function()
    {
        console.log(this);
    };
    var someObject = new Object();      // Create an object and initialize it to something empty
    someObject.myFunction = myFunction; // Give someObject a property
    someObject.myFunction();            // Logs someObject
    myFunction();                       // Logs...Window?

That's bizarre, isn't it?  Depending on how you call a function, its `this` pointer changes.

In the first example, because `myFunction` was a property of `someObject`, the `this` pointer referred to `someObject`.

But in the second example, because `myFunction` wasn't a property of anything, the `this` pointer defaults to the "global" object, which is `window`.

The important take-away here is that __`this` points to whatever is "left of the dot."__

As you can imagine, this causes a ton of confusion - particularly for new JavaScript developers.  My recommendation is to avoid writing code in such a way that this becomes a problem.  In fact, I try to avoid using `this` for anything but the obvious.

### WTF are .constructor and .prototype?

If you're asking this question, it means you're getting knee-deep into JavaScript OOP.  Good for you!

The first thing you need to know is that JavaScript does NOT use classical OOP.  It uses something called prototypal OOP.  This is very, very different.  If you really want to know how JavaScript OOP works, you need to read [Constructors considered mildly confusing, by Joost Diepenmaat](http://joost.zeekat.nl/constructors-considered-mildly-confusing.html).  Joost does a better job of explaining it than I ever will.

But for the lazy, I'll summarize: JavaScript does not have any classes.  You don't create a class and spawn new objects off of it like in other languages.  Instead, you create a new object, and set its `prototype` property to point to the old one.

When you refer to an object's property, if that property doesn't exist JavaScript will look at the prototype object...and its prototype, and its prototype, all the way up until it hits the `Object` object.

So unlike the class/object model you see in other languages, JavaScript relies on a series of "parent pointers."

`constructor` is a pointer to the function that gets called when you use the `new` keyword.  If you want to learn more about the prototype/constructor relationship, read [The Surprisingly Elegant JavaScript Type Model, by Kannan Vijayan](http://vijayan.ca/blog/2012/02/21/javascript-type-model/).  But be prepared to be confused.

### WTF is a closure?
Closures are a concept that appear in pure functional languages like JavaScript, but they have started to trickle their way into other languages like PHP and C#.  For those unfamiliar, "closure" is a language feature that ensures variables never get destroyed if they are still required.

This explanation is not particularly meaningful in and of itself, so here's an example:

    // When someone clicks a button, show a message.
    var setup = function()
    {
        var clickMessage = "Hi there!";
        $('button').click
        (
            function()
            {
                window.alert(clickMessage);
            }
        );
    };
    setup();

In a language without closures such as C/C++, you'd likely get an error when that click handler fires because `clickMessage` will be undefined.  It'll have fallen out of scope long ago.

But in a language with closures (like JavaScript), the engine _knows_ that `clickMessage` is still required, so it keeps it around.  When a user clicks a button, they'll see, "Hi there!"

As you can imagine, closures are particularly useful when dealing with event handling, because an event handler often gets executed long after the calling function falls out of scope.

Because of closures, we can go one step further and do something cool like this!

    (function()
    {
        var clickMessage = "Hi there!";
        $('button').click
        (
            function()
            {
                window.alert(clickMessage);
            }
        );
    })();

In the above example, we don't even need to give the function a name!  Instead, we execute it once with the () at the end, and forget about it.  Nobody can ever reference the function again, but _it still exists_.  And if someone clicks that button, it will still work!

### Why does JavaScript have so many different ways to do the same thing?
Because JavaScript is a poorly designed language in a lot of ways.  Your best guide to muddle through it is to read [JavaScript: The Good Parts, by Douglas Crockford](http://www.amazon.ca/JavaScript-Good-Parts-Douglas-Crockford/dp/0596517742).  He clearly outlines which pieces of the language you should ignore.

## Best Practices We Follow:

### Ensure your site still works without JavaScript.
<!--
* In particular, links...progressive enhancement URL?
-->

### Use the Module Pattern to encapsulate.
<!--
* Provides scope
* Provides public/private support
* By default, works best for singletons (but can be used for true OOP)
-->

### When using the Module Pattern, keep your JavaScript public object as clean as possible.
<!--
* Avoid polluting the global namespace.
-->

### Namespace your JavaScript if you need to refer to it elsewhere.

### Anonymously scope JavaScript if you’re never going to call it elsewhere.

### Put your JavaScript right before the `<body>` tag.

### Avoid causing document reflows.

### Optimize the big things.
<!--
* Document reflows
* Events that get fired ALL THE TIME (eg on resizing)
-->

### Be sure to `unbind()` before binding.
<!--
* Not strictly required, but a good defensive coding practices to prevent events from stacking up.
* jQuery event functions stack, they don't replace.
-->

### Prevent flash of unstyled content (FOUC).
<!--
* Wait until all content on the page has been loaded (can be detrimental to the UX)
* Put some of the "styling" scripts in the <head> (be wary that this can be a VERY bad practice...)
* CSS3 or JS fade-ins
-->

### Avoid chains where you can't detect `null`/`undefined` mid-chain.

## Bad Practices We Avoid:

### Treating JavaScript like a classical OOP language.
<!--
* Prototypal, not classical...read The Good Parts for more info
* Classical OOP results in ugly and complicated JavaScript
* Don't create crazy object structures in JS
    * Rely on the DOM as your object model, or
    * Rely on JSON REST results as your object model.
-->

### Inlining the crap out of functions and object literals.

### Excessive optimization.
<!--
* Caching selectors, especially long-term (at most cache for only a function's lifetime)
* Going nuts with minification
-->

### Optimizations that try to "outsmart the browser."
<!--
* Inevitably go out of vogue quickly, eg. domain sharding
-->

## "Best" Practices We Disagree With:
<!--
Many of these come from: [JS adolescence](http://james.padolsey.com/javascript/js-adolescence/)
-->

### Putting all `var` declarations at the top.

### Obsessing over indentation, tabs vs spaces, and squiggly bracket placement.

### Obsessing over strict equality.

### Caching selectors for long periods of time.

### Combined `var` declarations.

### Excessive chaining at the expense of readibility.