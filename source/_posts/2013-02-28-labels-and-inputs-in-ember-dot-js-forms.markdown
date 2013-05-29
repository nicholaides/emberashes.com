---
layout: post
title: "Labels and Inputs in Ember.js Forms"
date: 2013-02-28 18:41
comments: true
categories:
- forms
- html
---

When creating HTML forms, you usually want `<label>` tags to focus their associated `<input>` element when clicked. This is especially important for inputs that are hard to click like checkboxes and radio buttons.

In plain old HTML, you have two options to make this happen. The first is to wrap the `<input>` in the `<label>` tag, like so:

``` html
<label>
  <input name="name">
  Name
</label>
```

This works for some situations, but it can be difficult to style this to make it look the way you want.

A better, more flexible technique, is to make the `<label>`'s `for="..."` attribute reference the `<input>` by its `id`, like so:

``` html
<input id="name-input">
<label for="name-input">Name</label>
```

I prefer this method primarily because it provides flexibility with styling and arranging DOM elements.

However, Ember.js presents us with a difficulty: It's common when using Ember.js to want the same form template used on the page in multiple places. If the `id` of the `<input>` is hard coded, the two (or more) instances of the `<input>` will have the same `id`, and all of the `<label>`s that reference that `id`, when clicked, will focus the first `<input>` on the page has that `id`.

How do we avoid hardcoding the `id` in the template? [I found the solution to this problem on StackOverflow][stackoverflow] and I'll summarize it here.

If you use inspect one of your `<input>`s with [Firebug][firebug] or [Web Inspector][web-inspector] you'll find it looking something like this:

``` html
<input id="ember123456">
```

Ember.js gives `id`s to all of its views' elements by default, including `Ember.TextField`, `Ember.Checkbox`, etc.  We can bind our `<label>`'s `for` attribute to the `<input>`'s auto-generated `id`. This is how we do that:

``` html
{{view Em.TextField viewName="nameInput"}}
<label {{bindAttr for="view.nameInput.elementId"}}>Name</label>
```

The magic lies in `Ember.View`'s `viewName` property. When our `Em.TextField` is given a `viewName` property, it registers itself with its parent view as a property whose name is defined by `viewName`. In this instance, the template can reference that `Em.TextField` as `view.nameInput`. The `elementId` is the `Em.TextField`'s auto-generated `id`. In this way, the `<label>`'s `for` attribute is bound to the `<input>`'s `id`.

With this solution we get the following benefits:
- we avoid `id` collisions, so we can have multiple instances of the same template on the page at the same time.
- we avoid having to manually specify an `id`
- we are able to have the `<input>` outside of the `<label>`, which makes it easier to style



[stackoverflow] http://google.com
[firebug]       http://google.com
[web-Inspector] http://google.com
