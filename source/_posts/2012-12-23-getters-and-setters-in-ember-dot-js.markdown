---
layout: post
title: "Getters and Setters in Ember.js"
author: Mike Nicholaides
date: 2012-12-23 20:21
comments: true
categories:
- properties
---

Computed properties are the cornerstone of data binding in Ember.js. If you've written any code for Ember you've probably written your fair share of them. They tend to look something like this:

``` javascript
var User = Em.Object.extend({
  fullName: (function() {
    return this.get('firstName') + ' ' + this.get('lastName');
  }).property('firstName', 'lastName')
});
```

The above property is a "getter". That is, it is limited to retrieving its value. If you try to set the `fullName` property it will be ignored. Observe:

``` javascript
var user = User.create({
  firstName: 'John',
  lastName: 'Stamos'
});

user.get('fullName'); // => "John Stamos"
user.set('fullName', 'Danny Tanner');
user.get('fullName'); // => "John Stamos"
```

This is where "setters" come in. Setters let us set a computed property and provide logic to handle, manipulate, or deal with the input.

Let's make a getter+setter (AKA "accessor") for our `fullName` property. This property handles both getting and setting.

``` javascript
var User = Em.Object.extend({
  fullName: function (key, fullNameString) {
    if (arguments.length === 1) { // get
      return this.get('firstName') + ' ' + this.get('lastName');

    } else { //set
      var nameParts = fullNameString.split(/\s+/);
      this.set('firstName', nameParts[0]);
      this.set('lastName',  nameParts[1]);

      return fullNameString;
    }
  }.property('firstName', 'lastName')
});

```

Let's look this piece by piece.

## Get or set?

The first part to notice is this:

``` javascript
fullName: function (key, fullNameString) {
  if (arguments.length === 1) { // get
    // ...
  } else {                      // set
    // ...
```

**Getting**: When we call `user.get('fullName')`, the property is called with `key` as `'fullName'` and no `fullNameString` argument.

**Setting**: When we call `user.set('fullName', 'John Stamos')`, the property is called with `key` as `'fullName'` and `fullNameString` as `'John Stamos'`.

In other words, when `arguments.length === 1`, it should act as a getter, and when `arguments.length === 2`, it should act as a setter.

## The Getter

When this function is being called as a getter, it behaves just like normal computed properties. This line from the getter section is the same as in my first example of a normal computed property:

``` javascript
this.get('firstName') + ' ' + this.get('lastName')
```

Whatever we return here is what we get when we call `user.get('fullName')`.

Also, just like normal computed properties, you'll notice that we need to keep this line, too:

``` javascript
}.property('firstName', 'lastName')
```

## The Setter

Here are the relevant sections of the setter portion of our property:

``` javascript
fullName: function (key, fullNameString) {
  if (arguments.length === 1) { // get
    // ...
  } else { // set
    var nameParts = fullNameString.split(/\s+/);
    this.set('firstName', nameParts[0]);
    this.set('lastName',  nameParts[1]);

    return fullNameString;
  // ...
```

First, as you might expect, we split a given full name into first and last name and then set the `firstName` and `lastName` properties:

``` javascript
var nameParts = fullNameString.split(/\s+/);
this.set('firstName', nameParts[0]);
this.set('lastName',  nameParts[1]);
```

Finally, we return the value that should be cached as the `fullName` property's value:

``` javascript
return fullNameString;
```

This is the most surprising part so I'll repeat it: **whatever is returned from the setter is cached as the property's value**. One must be careful to return the right value.

And, that's all there is to know about getters and setters in Ember.js. Or, at least on a surface level.

## ur doin it wrong

The above structure for a getter+setter property is standard and is the what's prescribed by the Ember.js documentation. However, it has a fatal flaw. The following example will demonstrate.

What if the `firstName` property is also a getter+setter? Let's say, as a contrived example, that because of some new regulatory legislation we are not legally allowed to collect first names, only first initials. We might make our `firstName` property look like this:

``` javascript
firstName: function(key, firstNameStr) {
  if (arguments.length === 1) { // get
    return this.get('_firstInitial') || '';
  } else { // set
    var firstInitial = firstNameStr[0];
    this.set('_firstInitial', firstInitial);
    return firstInitial;
  }
}.property('_firstInitial')
```

This property works like this:

``` javascript
var user = User.create();
user.set('firstName', 'John');
user.get('firstName'); // => 'J'
```

Let's look at how the `fullName` property behaves when this change is introduced.

This use case works fine:

``` javascript
var user = User.create();
user.setProperties({
  firstName: 'John'
  lastName:  'Stamos'
});

user.get('fullName'); // => 'J Stamos'
```

This one... not so much:

``` javascript
var user = User.create();
user.set('fullName', 'John Stamos');

user.get('firstName'); // => 'J'
user.get('lastName');  // => 'Stamos'

user.get('fullName');  // => 'John Stamos'
```

Uh oh. Did you catch it? The `fullName` property should be returning `'J Stamos'`, not `'John Stamos'`.  The problems is that the `fullName` property assumed how the `firstName` property would handle its input.

Instead, I advocate that each property mind its own business. `fullName` should allow `firstName` to handle its own input (and caching).

## A better way

Here's a version of the `fullName` property that keeps to itself and thus works correctly:

``` javascript
fullName: function(key, fullNameString) {
  if (arguments.length > 1) { // set
    var nameParts = fullNameString.split(/\s+/);
    this.set('firstName', nameParts[0]);
    this.set('lastName',  nameParts[1]);
  }

  // get and set
  return this.get('firstName') + ' ' + this.get('lastName');
}.property('firstName', 'lastName')
```

When the property is called as a getter (i.e. with one argument), it works the same way it always had.

When the property is called setter, it sets the properties it needs to but doesn't just return `fullNameString` like the previous version did. Instead, it allows the getter logic to run, which honors `firstName`'s and `lastName`'s decisions about how they should handle being set.

Not only is this version more correct, it is shorter and more DRY.

## Conclusion

Getters and setters are an important part of data binding in Ember.js. Unfortunately,  they can be confusing if you haven't experimented a lot and read the source, which you shouldn't have to do.

Ember.js is a young technology and presents a paradigm that's new for most of us.  If you have other insights, techniques, best practices, or critiques, I'd love to hear them.
