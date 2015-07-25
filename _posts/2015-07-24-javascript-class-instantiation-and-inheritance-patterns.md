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
- <div><a href="#summary"           class="jump">Summary</a></div>

<!-- script -->
<script>
  $(function() {
    $('.jump').click(function(e) {
      e.preventDefault(); // prevent default behavior - "jump" to anchor
      $('html, body').animate({
        scrollTop: $($(this).attr('href')).offset().top - 90
      }, 'slow');
    });

  });
</script>

<h2 id="functional">Functional</h2>

A class is a set of characteristics that define a collection of things. Knowing that something belongs to a certain class should indicate that certain properties are exhibited by that thing. In OOP, the "thing" would be referred to as an **object**, which is an **instance** of a class. The simplest way to create objects that all exhibit the same properties, is through a function.

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

However, not all animals are capable of growling, so it may not be appropriate to add that method on all animals. This is a good reason to subclass. Subclassing under the functional pattern is very straight forward. I can create a function that makes bears, employing `makeAnimal` in the process:

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

This, in essense, is inheritance under the functional pattern. You would be correct to point out that this does not provide certain capabilities offered by inheritance in other OOP languages. For instance, there is no way to know that bears made by `makeBear` are animals without examining the code. This code is also not very efficient. Every instance of bear (and animal, for that matter) holds properties that point to its own function definitions. There is no convenient method to alter the functionality of the entire class.

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

If more methods are needed for bears, simply define them  within `bearMethods`, and use a function such as [extend] [underscore-extend] within `makeBear` to extend all the methods onto the bear instance.

Inheritance would work the same way as the plain Functional pattern. A subclass of bears would invoke `makeBear` to create a bear (which _extends_ all methods from `bearMethods`) and then decorate the instance with additional properties and methods specific to the subclass (for example, extending it with methods from a method holder called `polarBearMethods`).

Aside from the potential memory cost savings, this pattern does not offer many other benefits over the first functional pattern. There is still no convenient way to alter the methods for all instances of a class. Altering the `bearMethods` object will only point the properties at new function objects. The bears that have already been instantiated will not receive the update, unless explicitly updated with these new method definitions.

<h2 id="prototypal">Prototypal</h2>

The functional patterns do not create proper objects in the classical sense. They do not allow inheritance, since the instances of the "class" are very much their own object with access to only their own properties. The prototypal pattern allows for this behavior through an object's **prototype**.

Consider this example using pokemon under the functional shared pattern:

```javascript
// functional shared pattern
var pokemonMethods = {
  growl: function() {
    console.log( this.name + '! ' + this.name + '!');
  }
}

var makePokemon = function(name, level, type) {
  var pokemon = {};
  pokemon.name = name;
  pokemon.level = level;
  pokemon.type = type;
  pokemon.growl = pokemonMethods.growl;
  return pokemon;
}

var zeni = makePokemon('squirtle', 15, 'water');
zeni.growl(); // logs 'squirtle! squirtle!'

pokemonMethods.fight = function() {
  console.log( this.name + 'used tackle!');
}

zeni.fight(); // fight is undefined!
```

A simple modification to this code transforms this into the prototypal instantiation pattern:

```javascript
// prototypal pattern
var pokemonMethods = {
  growl: function() {
    console.log( this.name + '! ' + this.name + '!');
  }
}

var makePokemon = function(name, level, type) {
  var pokemon = Object.create(pokemonMethods); // *
  pokemon.name = name;
  pokemon.level = level;
  pokemon.type = type;
  // pokemon.growl = pokemonMethods.growl; // *
  return pokemon;
}

var zeni = makePokemon('squirtle', 15, 'water');
zeni.growl(); // logs 'squirtle! squirtle!'

pokemonMethods.fight = function() {
  console.log( this.name + ' used tackle!');
}

zeni.fight(); // logs 'squirtle used tackle!''
```

Note the very important difference within the `makePokemon` function. Instead of creating an empty object in the first line of `makePokemon`, `Object.create` is called with `pokemonMethods` as an argument. `Object.create` (introduced with ES5) is a method that returns a new object that will delegates failed property lookups to the object passed in to the function. For this reason, extending methods from the method holder to the instance is no longer necessary. Calling `zeni.growl()` will do the following:

1. Search `zeni` for the method `growl`
2. Fail method lookup on `zeni`
3. Search `pokemonMethods` for the method `growl`
4. Successful method lookup
5. Invoke `growl`, with `this` referring to `zeni`

The most powerful aspect of this pattern is that new methods declared on `pokemonMethods` (the `prototype`) will be available to `zeni` without any additional action. Modifications to `pokemonMethods` will affect all those that inherit from it, so long as they rely on `pokemonMethods` for lookups. If a `growl` method got defined on `zeni`, it would effectively **mask** the `growl` method on the prototype.

```javascript
zeni.growl = function() {
  console.log('squirrrrrrtttlleeeeee!!!');
}

zeni.growl(); // logs 'squirrrrrrtttlleeeeee!!!'
```

Inheritance under this pattern is also possible.

```javascript
var pokemonMethods = {
  growl: function() {
    console.log( this.name + '! ' + this.name + '!');
  }
  fight: function() {
    console.log( this.name + ' used tackle!');
  }
}

var makePokemon = function(name, level, type) {
  var pokemon = Object.create(pokemonMethods); // *
  pokemon.name = name;
  pokemon.level = level;
  pokemon.type = type;
  // pokemon.growl = pokemonMethods.growl; // *
  return pokemon;
}

var squirtleMethods = Object.create(pokemonMethods);
squirtleMethods.fight = function() {
  console.log( this.name + ' used Water Gun!');
}

var makeSquirtle = function(level) {
  var squirtle = Object.create(squirtleMethods);
  squirtle.name = 'squirtle';
  squirtle.level = level;
  squirtle.type = 'water'
  return squirtle;
}

var zeni = makeSquirtle(15);

zeni.growl(); // logs 'squirtle! squirtle!'
zeni.fight(); // logs 'squirtle used Water Gun!'
```

Note that `growl` is not a method available on `squirtleMethods`, but since `squirtleMethods` delegates to `pokemonMethods`, it is still available on `zeni`.

<h2 id="pseudoclassical">Pseudoclassical</h2>

The pseudoclassical pattern is the patternt that most resembles the classic class instantiation pattern of other langauges such as Java and C++. Here the function `Pokemon` is capitalized by convention to indicate that it is a **constructor** (the capitalization does nothing special besides indicate to readers that this is a constructor). Constructor functions need to be instantiated by using the `new` keyword.

The `new` keyword has the effect of letting `this` within the constructor function inherit from the prototype. The prototype in this case is `Pokemon.prototype` instead of a `pokemonMethods`. `Pokemon.prototype` is nothing more than an an object defined as a property of the constructor function (recall that functions are objects themselves, and can have arbitrary properties defined on them). For convenience, the `prototype` property is defined for you by JavaScript and is automatically the prototype of all instances created by the constructor.

```javascript
var Pokemon = function(name, level, type) {
  this.name = name;
  this.level = level;
  this.type = type;
}

Pokemon.prototype.growl = function() {
  console.log( this.name + '! ' + this.name + '!');
}

Pokemon.prototype.fight = function() {
  console.log( this.name + ' used tackle!');
}
```

The pseudoclassical pattern really shines when it comes to inheritance. Here is an example of subclassing the `Pokemon` class to a `Squirtle` class.

```javascript
var Squirtle = function(level) {
  Pokemon.call(this, 'squirtle', level, 'water'); // *
}

Squirtle.prototype = Object.create(Pokemon.prototype); // *
Squirtle.prototype.constructor = Squirtle; // *

Squirtle.prototype.fight = function() {
  console.log( this.name + ' used Water Gun!');
}

var zeni = new Squirtle(15);

zeni.growl(); // logs 'squirtle! squirtle!'
zeni.fight(); // logs 'squirtle used Water Gun!'
```

There are a couple of important things to note about subclassing. The first line within the `Squirtle` constructor merely invokes the `Pokemon` constructor and passes along the appropriate arguments. The `this` keyword, which references the instance of Squirtle being created, will have properties `name`, `level`, and `type` assigned via `Pokemon`.

More important are the lines that set up the proper prototype chain. Without these two lines, `Squirtle` would merely be a regular constructor function, where instances of Squirtle would delegate failed property lookups to `Squirtle.prototype`. As mentioned before, `Object.create` returns an object that inherits from the prototype passed in as the argument. Using the code below, `Squirtle.prototype` now inherits from `Pokemon.prototype`. The `growl` method from `Pokemon.prototype` can now be called from `zeni`:

```javascript
Squirtle.prototype = Object.create(Pokemon.prototype);
```

Something I have not mentioned earlier is that each `prototype` property of a function has another property `constructor` that points back at the function itself. That is:

```javascript
Pokemon.prototype.constructor === Pokemon; // true
```

So what happens when we set `Squirtle.prototype` to `Object.create(Pokemon.prototype)`? `Squirtle.prototype` actually does not have its own `constructor` property (`Object.create` does not conveniently set this up for you). As a result, it delegates failed lookups to `Pokemon.prototype`, which **does** have a `constructor` property...pointing to `Pokemon`. This line is required to set things back to normal.

```javascript
Squirtle.prototype.constructor = Squirtle;

// and now this makes sense
zeni.constructor === Squirtle; // true
```

The pseudoclassical instantiation pattern is usually the most popular. However, it requires a thorough understanding of the pseudoclassical instantiation pattern. Furthermore, before ES5 introduced `Object.create` there were several other conventions used to substitute this LOC, such as setting it equal to a new instance of Pokemon.

```javascript
Squirtle.prototype = Object.create(Pokemon.prototype); // correct
Squirtle.prototype = new Pokemon; // used to be common practice, not correct
```

Failed method lookups will fall through to the instance of Pokemon, and then to the prototype of Pokemon, as desired. However, this created problems if the `Pokemon` constructor required a lot of initialization parameters and could not accept undefined ones.

Know that `Object.create` is the only correct pattern to use, but there might still be established code bases out there with the old pattern. In fact, I often see blog posts and websites reference the old pattern as the proper way to set up inheritance in JS.

<h2 id="summary">Summary</h2>

Even though the pseudoclassical pattern is the most powerful and is often optimized by many runtime engines, it may sometimes be benefiicial to go with a functional pattern for simplicity and transparency. It all depends on the intended usage. In my personal opinion, the **functional** and **pseudoclassical** patterns provide the best value. Here's a summary:

- **Functional**
  + Most transparent and easy to understand
- Functional shared
  + Offers minor space cost advantage over functional
  + Adds a little complexity over functional
- Prototypal
  + Not much easier to comprehend compared to the pseudoclassical pattern
  + Does not offer any obvious advantages over pseudoclassical pattern
- **Pseudoclassical**
  + Often the most optimized pattern
  + More complex, but is a very common pattern