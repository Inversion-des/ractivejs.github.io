# Decorators

A decorator is a simple way to add behaviour to a node when it is rendered, or to augment it in some way. Decorators are a good way to integrate DOM manipulation libraries with Ractive, such as [jQuery UI](http://jqueryui.com/) or [Bootstrap](https://getbootstrap.com/).

## Writing

```js
const myDecorator = function(node[, ...args]) {
  // Setup code
  return {
    teardown: function(){
      // Cleanup code
    },
    update: function([...args]){
      // Update code
    }
  };
};
```

Decorators are simply functions that are called upon to setup the decorator once Ractive detects its use. It takes a `node` argument and returns an object with a `teardown` and `update` property.

`node` is the element to which the decorator is applied to.

`args` are optional arguments provided by the decorator directive.

`teardown` is a function that gets called when the decorator is torn down.

`update` is an optional function that gets called when the arguments update.

Any updates to the arguments will call the decorator's `teardown` and run the decorator function again, essentially setting up the decorator again. If an `update` function is provided on the return object, that will be called instead of the `teardown` and setup function.

## Registering

Like other plugins, there's 3 ways you can register decorators:

### Globally, via the `Ractive.decorators` static property.

```js
Ractive.decorators.myDecorator = myDecorator;
```

### Per component, via the component's `decorators` initialization property.

```js
const MyComponent = Ractive.extend({
  decorators: { myDecorator }
});
```

### Per instance, via the instance's `decorators` initialization property.

```js
const ractive = new Ractive({
  decorators: { myDecorator }
});
```

## Using

You can invoke one or more decorators on your elements by using a decorator directive. Arguments are optional. Argument-less decorators can simply use the directive without value. Decorators with arguments take a comma-separated set of expressions that resolve to the element's context.

```html
<!-- without arguments -->
<div as-myDecorator>...</div>

<!-- with arguments -->
<div as-myDecorator="arg1, .some.other.arg2, 10 * @index" as-somethingElseToo>...</div>
```

## Examples

The following example builds a decorator that updates the time.

```js
Ractive.decorators.timer = function(node, time) {
  node.innerHTML = 'Hello World!';

  return {
    teardown: function() {
      node.innerHTML = '';
    },
    update: function(time) {
      node.innerHTML = time;
    }
  }
};

const ractive = new Ractive({
  el: 'body',
  template: `
    <span as-timer="time"></span>
  `,
  data: {
    time: 0
  }
});

setInterval(function() {
  ractive.set('time', Date.now())
}, 1000);
```
