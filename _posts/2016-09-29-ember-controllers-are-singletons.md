---
layout: post
title: Ember - Controllers are singletons
---

## Controllers are singletons

Controllers in Ember are singletons. Controllers in Ember are singletons. Controllers in Ember are singletons.

When the user leaves a page and goes to another one, the controller is not torn down. It lives on, keeping its properties.

This makes total sense for a framework that aims to be a tool for creating long-lived, rich client side applications but is something to watch out for when you develop Ember applications.

If you have a long background in back-end development, like yours truly, it is especially easy to fall prey to this, as you could see.

## Understanding the problem

For example, we have a controller `ChartController` which have `changedCalculation` is mark for any changes in the calculation of this chart. If `changedCalculation` is true, we will have an `Apply` button bellow input of this property.

{% highlight javascript %}
{% raw %}
changedCalculation: function () {
  this.set('controller.changedCalculation', true);
}
{% endraw %}
{% endhighlight %}

{% highlight html %}
{% raw %}
{{#if changedCalculation}}
  <button class="btn btn-primary size-mini btn-ctrl pull-right" {{action 'appyCalculation'}}>
    {{i18n js.charts.daten_tab.insert}}
  </button>
{{/if}}
{% endraw %}
{% endhighlight %}

When the transition is entered between the two charts, the chart object is changed and consequently any data bound to the chart (and the `changedCalculation` property of the `ChartController` controller) is going to be rerendered, but since `changedCalculation` is not changed, unrelated data will stay unchanged on screen with `Apply` button still in there.

## How to fix

Here are some solutions and it depends on the specific case:

1. **setupController**: A hook you can use to setup the controller for the current route. This method is called with the controller for the current route and the model supplied by the model hook.
2. **resetController**: A hook you can use to reset controller values either when the model changes or the route is exiting. (It has existed from Ember 1.7).
3. **deactivate**: This hook is executed when the router completely exits this route. It is not executed when the model for the route changes.

Example:

```javascript
setupController: function(controller, model) {
  controller.set('changedCalculation', false);
  this._super.apply(this, arguments);
}
```

## What I can do with it

The `render` helper renders a combination of a controller and template, and optionally allows you to provide a specific model to be set to the content property of the controller. If you do not provide a model, the singleton instance of the controller will be used.

Example:

{% highlight html %}
{% raw %}
{{render 'follow-authors'}}
{% endraw %}
{% endhighlight %}

```javascript
loadData: Ember.on('init', function() {
  this.get('store').query('author', {
    type: 'sugget_authors'
  }).then((authors) => {
    this.set('authors', authors);
  });
})
```
For this example, data of `authors` property only load 1 time with anytime next rendering.

There are confused between `render` and `component` but we talk about it next time.

**Source:**

1. [Ember Gotcha: Controllers Are Singletons](http://balinterdi.com/2014/06/26/ember-gotcha-controllers-are-singletons.html)
