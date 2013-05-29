---
layout: post
author: Mike Nicholaides
date: 2013-03-01 10:55
comments: true
categories:
- forms
- best practice
- validation
- usability
title: "Use Real Forms with Ember.js"
---

When creating interfaces with Ember.js, it's not necessary to use actual HTML `<form>` elements because you bind your `<input>`s' data to models rather than submitting the data to the server through the form. In fact, submitting your form to the server defeats the purpose of making single-page apps.

While you don't *need* to use `<form>` elements, I would suggest that you do, because you gain a number of tangible benefits through the default behavior that browsers implement for HTML forms.

## Press Enter to Submit

The simplest and most immediate benefit you get by using a real HTML `<form>` tag is that pressing <kbd>ENTER</kbd> submits the form if it contains a submit input (`<input type="submit">`) or a button (`<button>`).

To capture the submit event just add a `submit` method to your view. You'll probably just want view to delegate the real work to the controller.

``` javascript
App.MyForm = Em.View.extend({
  tagName: 'form',
  submit: function (event) {
    this.get('controller').save()
  }
});
```

``` html
{{#view App.MyForm}}
  {{view Em.TextField valueBinding="phoneNumber"}}
  <input type="submit">
{{/view}}
```
Here's an example of this in action. Notice that you can hit <kbd>ENTER</kbd> to submit the form.
<iframe style="width: 100%; height: 300px" src="http://jsfiddle.net/YADJb/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Using this technique, we can take advantage of browsers' default behavior around submitting forms, which is what users have come to expect.


## HTML5 Form Validation

If you're developing for newer browsers and you use the HTML `<form>` tag you can take advantage browsers' built-in form validations. These allow us to prevent a form from being submitted if the input is invalid. This includes built-in validations like "required", URL format, email format, arbitrary regexp format and error messages. If you're not familiar with this feature that's built into modern browsers, do some reading. A good place to start would be the [HTML5 Rocks tutorial on Constraint Validation][html5rocks]

Here's an example of using new HTML5 form features:

```
{{view Em.TextField required=true type="email" placeholder="Your Name"}}
```

If you want to use these special attributes, you 'll have to make sure that the `Em.TextField` view passes through the attributes you set. By default these attributes are passed through:

- `type`
- `value`
- `size`
- `pattern`

If you want to use other attributes on `Em.TextField`s, you'll need to add them to the list of bound attributes. To do so, just put this snippet in your JavaScript. Obviously, you'll want to edit the list to accommodate your needs.

``` javascript
Em.TextField.reopen({
  attributeBindings: ['required', 'placeholder']
});
```

Here's an example of this in action.

If you use an actual HTML `<form>` tag, you can take advantage of these built-in validations.

## Reset

HTML forms also have built-in "reset" functionality that you can take advantage of. This feature isn't used often but is worth mentioning.

Ember doesn't have a built-in view for this because it doesn't need one. Just put an `<input type="reset">` in your form to return its fields to their original values

Here's an example of this in action.
...

## Conclusion

There are tangible benefits to using real HTML `<form>` elements in your Ember.js apps even though they aren't strictly necessary. Not can you gain better usability, but it can also save you time and lines of code.

[html5rocks]:           http://html5rocks.com/en/tutorials/forms/constraintvalidation/
