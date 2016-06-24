---
layout: post
title:  Instantiation Patterns in Javascript
tags:
- Javascript
- Blog
- Post
---

If you come from a programming background other than Javascript, classes in JS can seem a little strange. This is because Javascript is not technically a class-based language in the classical sense (I won't apologize for puns), but rather a **[prototype-based language](https://en.wikipedia.org/wiki/Prototype-based_programming)**. That being said, there are a number of ways to recreate class-like behavior.
<div class="divider"></div>
In Javascript, there are four basic instantiation patterns we can follow for creating our class definitions:

1. Functional
2. Functional-shared
3. Prototypal
4. Pseudoclassical

<div class="divider"></div>
Each has its pros and cons, which we'll discuss as we go.

## The Scenario

Let's say we're making a game and we need a class to represent our hero. Let's make it this guy, **because life is too short not to make a Pokemon with Mega Man buster guns for hands.**

->![Totoman](/sustain/assets/images/totoman.png)<-

Let's say this hero should have:

* A health bar
* A current level
* An inventory
* An attack method

##Functional Instantiation
Functional instantiation is the easiest to understand form of object instantiation in Javascript.

First thing we'll do is create our class' function signature and an object to hold its properties:

{% highlight javascript %}
var Player = function(level, inventory) {
  var playerInstance = {};
}
{% endhighlight %}

Our next step will be to give our player the properties we decided on earlier. In the functional pattern, we'll place these directly on our playerInstance object:

{% highlight javascript %}
var Player = function(level, inventory) {
  var playerInstance = {
    health: 100,
    level: level,
    inventory: inventory
  };
}
{% endhighlight %}

Finally, to add the attack method we can simply add it to our playerInstance object and return the instance:

{% highlight javascript %}
var Player = function(level, inventory) {
  var playerInstance = {
    health: 100,
    level: level,
    inventory: inventory
  };

  playerInstance.attack = function(enemy) {
    if (enemy.level < this.level) {
      enemy.health -= 10;
    }
  }

  return playerInstance
}
{% endhighlight %}

To create an instance of this new functional player class, simply call the function and assign it to a variable. Because we're nice, we'll make our player level 50:

{% highlight javascript %}
var myPlayer = Player(50, ['a sandwich or something']);
myplayer.level // returns 50
{% endhighlight %}

**Pros:** Pretty simple, everything belongs to an object we've explicitly declared.

**Cons:** Every time you construct an object, new functions will be added in memory.

##Functional-shared Instantiation

The functional-shared instantiation pattern looks a lot like the functional pattern, with the biggest difference being that the methods are stored outside our constructor and extended onto the object. This means we only create our methods once, and then pass them by reference to our instance.

First, we'll create our object in the same way we did in the functional pattern:

{% highlight javascript %}
var Player = function(level, inventory) {
  var playerInstance = {
    health: 100,
    level: level,
    inventory: inventory
  };
}
{% endhighlight %}

Next, we'll define our methods outside the object:

{% highlight javascript %}
var Player = function(level, inventory) {
  var playerInstance = {
    health: 100,
    level: level,
    inventory: inventory
  };
}

// method(s) are now stored outside the function
var methods = {
  attack: function(enemy) {
    if (enemy.level < this.level) {
      enemy.health -= 10;
    }
  }
}
{% endhighlight %}

Finally, we'll extend our methods onto our object and return the instance. One thing to note is we'll extend using Underscore.js, as there is no native extend function. However, you could extend in any way you see fit:

{% highlight javascript %}
var Player = function(level, inventory) {
  var playerInstance = {
    health: 100,
    level: level,
    inventory: inventory
  };

  // extend the methods onto the object
  _.extend(playerInstance, Player.methods);

  return playerInstance;
}

var methods = {
  attack: function(enemy) {
    if (enemy.level < this.level) {
      enemy.health -= 10;
    }
  }
}
{% endhighlight %}

As with the functional pattern, to create an instance of this class simply call the function:

{% highlight javascript %}
var myPlayer = Player(50, ['a sandwich or something']);
myplayer.level // still returns 50
{% endhighlight %}

**Pros:** Better for memory usage than functional, we only have to put the methods into memory once

**Cons:** Extending the methods onto the player object is cumbersome, and introduces either a dependency or for us to write another function to handle the extension

## Prototypal Instantiation

Prototypal instantiation takes advantage of the prototype property of our class to get around the need to extend our object. We achieve this by creating manually creating a prototypal delegation on our new object.

First create our object with the delegation, as well as our properties:

{% highlight javascript %}
var Player = function(level, inventory) {
  var playerInstance = Object.create(Player.prototype);

  playerInstance.health = 100;
  playerInstance.level = level;
  playerInstance.inventory = inventory;

  return playerInstance;
}
{% endhighlight %}

Next, we'll define our methods _on the prototype_:

{% highlight javascript %}
var Player = function(level, inventory) {
  var playerInstance = Object.create(Player.prototype);

  playerInstance.health = 100;
  playerInstance.level = level;
  playerInstance.inventory = inventory;

  return playerInstance;
}

// method(s) are now stored on the prototype
Player.prototype.attack = function(enemy) {
  if (enemy.level < this.level) {
    enemy.health -= 10;
  }
}
{% endhighlight %}

Now we can instantiate the object as usual:

{% highlight javascript %}
var myPlayer = Player(50, ['a sandwich or something']);
myplayer.level // still returns 50
{% endhighlight %}

**Pros:** More efficient than the functional patterns, as now rather than setting up pointers to functions, we simply allow method lookup to delegate to the prototype

**Cons:** Unnecessarily verbose, requires that we manually create our delegation

## Pseudoclassical Instantiation

The pseudoclassical pattern is much like the prototypal pattern, but utilizes the new keyword during instantiation to remove excess code. How does it do this? The Javscript interpreter knows when it sees the keyword new that there are a couple of lines it should insert in your constructor.

Let's start with the prototypal pattern we just learned:

{% highlight javascript %}
var Player = function(level, inventory) {
  var playerInstance = Object.create(Player.prototype);

  playerInstance.health = 100;
  playerInstance.level = level;
  playerInstance.inventory = inventory;

  return playerInstance;
}

Player.prototype.attack = function(enemy) {
  if (enemy.level < this.level) {
    enemy.health -= 10;
  }
}
{% endhighlight %}

So what lines are now unnecessary? It turns out, using the pseudoclassical pattern we no longer need to create our delegation or return our new object. Javascript is smart enough to add these lines for us! It creates a new object under bound to _this_ that will be returned automatically.

{% highlight javascript %}
var Player = function(level, inventory) {
  // This line is automatically added by the interpreter
  // var this = Object.create(Player.prototype);

  this.health = 100;
  this.level = level;
  this.inventory = inventory;

  // As is this one
  // return this;
}

Player.prototype.attack = function(enemy) {
  if (enemy.level < this.level) {
    enemy.health -= 10;
  }
}
{% endhighlight %}

The only catch to using the pseudoclassical pattern is, as I previously mentioned, we must now instantiate our objects using the new keyword.

{% highlight javascript %}
var myPlayer = new Player(50, ['a sandwich or something']);
myplayer.level // still returns 50
{% endhighlight %}

**Pros:** All the benefits of the prototypal pattern with less code

**Cons:** Lack of object creation and return in our function can be confusing for anyone not used to seeing the new keyword

<div class="divider"></div>

So which pattern is best to use? It's really up to you! Each has its own benefits and drawbacks, and ultimately whichever one makes the most sense to you/your employer is your best bet.

I tend to prefer the pseudoclassical style, as that most closely mimcs classes in many other languages. Making that relationship even tighter, ES6 introduced the class keyword, bringing much more standard class constructors to Javascript. No matter which pattern you use, *make sure not to forget to add blasters for arms*. Whatever that may mean in your code.