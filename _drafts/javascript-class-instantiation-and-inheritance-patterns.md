---
layout: post
title: JavaScript Class Instantiation & Inheritance Patterns
---

[underscore-extend]: http://underscorejs.org/#extend

<!-- post -->
At first glance, JavaScript may not seem like an object oriented programming (OOP) language, but it does have powerful object oriented programming capabilities. OOP features such as inheritance, polymorphism, and encapsulation are all enabled in JavaScript. However, being a very flexible language, there are several different ways to define classes and use them. Here's an overview of all the different JavaScript class instantiation and inheritance patterns and how to implement them.

<!--excerpt-->

<!--legend-->
## Sections ##
- <div><a href="#functional"        class="jump">Functional</a></div>
- <div><a href="#functional-shared" class="jump">Functional Shared</a></div>
- <div><a href="#prototypal"        class="jump">Prototypal</a></div>
- <div><a href="#pseudoclassical"   class="jump">Pseudoclassical</a></div>

<!-- script -->
<script>
  $(function() {
    $('.jump').click(function(e) {
      e.preventDefault(); // prevent default behavior - "jump" to anchor
      $('html, body').animate({
        scrollTop: $($(this).attr('href')).offset().top - 90
      }, 750);
    });

  });
</script>

<h2 id="functional">Functional</h2>

What is a **class**? In plain english, a class is a set of characteristics that define a collection of things. Knowing that something belongs to a certain class should indicate that certain properties are exhibited by that thing. In OOP, the "thing" would be referred to as an **object**, which is an **instance** of a class. The simplest way to create objects that all exhibit the same properties, is through a function.

Consider the function `makeAnimal` that takes a `name` and a `color` and returns animal objects. Every invocation of `makeAnimal` will create an instance that has name and color properties. This function is a **constructor** in the sense that it creates objects of a particular class.

```javascript
var makeAnimal = function(name, color) {
  var animal = {};
  animal.name = name;
  animal.color = color;
  return animal
}

var bob = makeAnimal('bob', 'blue'); // bob.name === 'bob'
var yo  = makeAnimal('yo',  'yellow'); // yo.color === 'yellow'
```

Methods (functions associated with a class) can also be added to each instance via the constructor function.

```javascript
var makeAnimal = function(name, color) {
  var animal = {};
  animal.name = name;
  animal.color = color;
  animal.growl = function() {
    console.log('grrr');
  }
  return animal
}

var bob = makeAnimal('bob', 'blue'); // bob.name === 'bob'

bob.growl() // logs 'grrr'
```

However, not all animals are capable of growling, so it may not be appropriate to add that method on all animals. This is a good reason to subclass. Subclassing under the functional pattern is very straight forward. I can create a function that makes bears, using my first definition of `makeAnimal` above:

```javascript
var makeAnimal = function(name, color) {
  var animal = {};
  animal.name = name;
  animal.color = color;
  return animal;
}

var makeBear = function(name, color, type) {
  var bear = makeAnimal(name, color);
  bear.type = type;
  bear.growl = function() {
    console.log('grrr');
  }
  return bear;
}

var yogi = makeBear('yogi', 'brown', 'thief');

yogi.growl(); // logs 'grrr'
```

This, in essense, is inheritance under the functional pattern. You would be correct to point out that this does not provide certain capabilities offered by inheritance in other OOP languages. For instance, there is no way to know that bears made by `makeBear` are animals without examining the code. This code is also not very efficient. Every instance of bear (and animal, for that matter) holds properties that point to its own function definitions.

```javascript
var yogi = makeBear('yogi', 'brown', 'thief');
var pooh = makeBear('winnie', 'yellow', 'honey');

yogi.growl(); // logs 'grrr'
yogi.growl === pooh.growl; // false
```

If my app required the creation of thousands of bears, there would be a lot of redundant memory costs associated with this pattern.

There are benefits to using this pattern though - this code is very transparent and easy to understand. Depending on the expected use of this function, the redundant memory costs may be perfectly acceptable.

<h2 id="functional-shared">Functional Shared</h2>

A slightly improved version of the functional pattern involves eliminating that redundant memory cost mentioned above. Instead of defining a function on each instance of the class, the methods can be defined in some form of _method holder_ for each instance to point at.

Using the same example as above:

```javascript
var makeAnimal = function(name, color) { // no change to this
  var animal = {};
  animal.name = name;
  animal.color = color;
  return animal;
}

var bearMethods = { // the holder of all methods that bears should have
  growl: function() {
    console.log('grrr');
  }
}

var makeBear = function(name, color, type) {
  var bear = makeAnimal(name, color);
  bear.type = type;
  bear.growl = bearMethods.growl; // point at same function as bearMethods.growl
  return bear;
}

var yogi = makeBear('yogi', 'brown', 'thief');
var pooh = makeBear('winnie', 'yellow', 'honey');

yogi.growl(); // logs 'grrr'
yogi.growl === pooh.growl; // true
```

If more methods are needed for bears, simply define them  within `bearMethods`. And within `makeBear`, a function such as [extend] [underscore-extend] can be used to extend all the methods onto the bear instance.

Inheritance would work the same way as the plain Functional pattern. A subclass of bears would call `makeBear` to create a bear (which _extends_ all methods from `bearMethods`) and then decorate the instance with additional properties and methods specific to the subclass (for example, via a method holder called `polarBearMethods`).

<h2 id="prototypal">Prototypal</h2>


<h2 id="pseudoclassical">Pseudoclassical</h2>
