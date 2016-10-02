---
layout: post
title: Ember Addons
---

## Already included in Ember

### [JQuery](https://jquery.com/)

jQuery is so popular, we don't need talk more about it.

### [Backburner](https://github.com/ebryn/backburner.js/)

A rewrite of the Ember.js [run loop](https://guides.emberjs.com/v1.10.0/understanding-ember/run-loop/) as a generic microlibrary.

How to the backburner.js work on Backbone and Ember [READ MORE](http://talks.erikbryn.com/backburner.js-and-the-ember-run-loop/).

Ember uses the run loop: The run loop will run from `sync` queue has higher priority than `render` or `destroy` queue:

1. The `sync` queue contains binding synchronization jobs
2. The `actions` queue is the general work queue and will typically contain scheduled tasks e.g. promises
3. The `routerTransitions` queue contains transition jobs in the router
4. The `render` queue contains jobs meant for rendering, these will typically update the DOM
5. The `afterRender` contains jobs meant to be run after all previously scheduled render tasks are complete. This is often good for 3rd-party DOM manipulation libraries, that should only be run after an entire tree of DOM has been updated
6. The `destroy` queue contains jobs to finish the teardown of objects other jobs have scheduled to destroy

Interactive visualization of the run loop:

[https://machty.s3.amazonaws.com/ember-run-loop-visual/index.html](https://machty.s3.amazonaws.com/ember-run-loop-visual/index.html)

Example:

```javascript
Ember.run.scheduleOnce('afterRender', this, () => {
  $('.data-source-list').find('.data-source-drag-drop').draggable({
    revert: 'invalid',
    containment: '#ember_app',
    cursor: 'move',
    helper: function () {
      return $('<div class="data-source-drag-helper">').appendTo('body').get(0);
    }
  });
});
```

### [RSVP](https://github.com/tildeio/rsvp.js/)

A lightweight library that provides tools for organizing asynchronous code. We talk more about later.

```javascript
reply.save().then(() => {
  this.decrementProperty('savingComment');
}).catch(() => {
  reply.deleteRecord();
}).finally(() => {
  this.scrollToEndCommentList();
});

```

## Other add-ons

### [ember-ajax](https://github.com/ember-cli/ember-ajax)

Service for making AJAX requests in Ember 1.13+ applications.

* Customizable service
* Ceturns RSVP promises
* Improved error handling
* Ability to specify request headers
* Upgrade path from ic-ajax

### [ember-infinity](https://github.com/hhff/ember-infinity)

Simple, flexible infinite scrolling for Ember CLI Apps

### [ember-route-action-helper](https://github.com/DockYard/ember-route-action-helper)

Bubble closure actions in routes

{% highlight html %}
{% raw %}
{{foo-bar clicked=(route-action "updateFoo" "Hello" "world")}}
{% endraw %}
{% endhighlight %}

```javascript
// application/route.js
import Ember from 'ember';

const { Route, set } = Ember;

export default Route.extend({
  actions: {
    updateFoo(...args) {
      // handle action
      return 42;
    }
  }
});
```

### [ember-wormhole](https://github.com/yapplabs/ember-wormhole)

This is my favorite ember addon :D. This addon provides a component that allows for rendering a block to a DOM element somewhere else on the page.

Example, given the following DOM:

```html
<body class="ember-application">
  <!-- Destination must be in the same element as your ember app -->
  <!-- otherwise events/bindings will not work -->
  <div id="destination">
  </div>
  <div class="ember-view">
    <!-- rest of your Ember app's DOM... -->
  </div>
</body>
```

{% highlight html %}
{% raw %}
{{#ember-wormhole to="destination"}}
  Hello world!
{{/ember-wormhole}}
{% endraw %}
{% endhighlight %}

Then "Hello world!" would be rendered inside the `destination` div.
