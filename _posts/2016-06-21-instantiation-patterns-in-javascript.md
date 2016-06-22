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
Functional instantiation is the easiest to understand form of object instantiation in Javascript. First thing we'll do is create our class' function signature and an object to hold its properties.

{% highlight javascript %}
var Player = function(level, inventory) {
  var playerInstance = {};
}
{% endhighlight %}

Our next step will be to give our player the properties we decided on earlier. In the functional pattern, we'll place these directly on our playerInstance object.

{% highlight javascript %}
var Player = function(level, inventory) {
  var playerInstance = {
    health: 100,
    level: level,
    inventory: inventory
  };
}
{% endhighlight %}

Finally, to add the attack method we can simply add it to our playerInstance object and return the instance.

{% highlight javascript %}
var Player = function(level, inventory) {
  var playerInstance = {
    health: 100,
    level: level,
    inventory: inventory
  };

  playerInstance.attack = function(enemy) {
    if (enemy.level < player.level) {
      enemy.health -= 10;
    }
  }

  return playerInstance
}
{% endhighlight %}

To create an instance of this new functional player class, simply call the function and assign it to a variable.

{% highlight javascript %}
var myPlayer = Player();
myplayer.health // returns 100
{% endhighlight %}