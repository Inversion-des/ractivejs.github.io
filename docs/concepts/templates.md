# Templates

Strictly speaking, Ractive templates are not HTML. They are markup representations of objects that are used to construct HTML. Simply put, templates are _HTML-like_. Ractive parses templates into ASTs which contain everything Ractive needs to know to construct an instance's DOM, data bindings, events and transitions etc.

```js
Ractive.parse('<div class="message">Hello World!</div>')

// {"v":4,"t":[{"t":7,"e":"div","m":[{"n":"class","f":"message","t":13}],"f":["Hello World!"]}]}
```

## Mustaches

[Mustache](http://mustache.github.io/) is one of the most popular HTML templating languages. It has a very lightweight, readable syntax with a [comprehensive specification](https://mustache.github.io/mustache.5.html). If you've used [Handlebars](http://handlebarsjs.com) or [Angular](http://angularjs.org), you'll also find mustaches familiar. Ractive implements a subset of the Mustache specification and adds a few extensions to the language.

### Variables

`{{ }}`, `{{& }}` or `{{{ }}}` render a referenced value. If the reference does not exist, nothing is rendered. `{{ }}` escapes the referenced value while `{{& }}` and `{{{ }}}` do not.

```js
Ractive({
  data: {
    name: "Chris",
    company: "<b>GitHub</b>"
  },
  template: `
    {{name}}      <!-- Chris -->
    {{age}}       <!--  -->
    {{company}}   <!-- &lt;b&gt;GitHub&lt;/b&gt; -->
    {{&company}}  <!-- <b>GitHub</b> -->
    {{{company}}} <!-- <b>GitHub</b> -->
  `
})
```

### Sections

Sections render a block of markup depending on the value referenced.

If the reference is an array, it renders the block of markup for each item in the array. If the array is empty, nothing is rendered. The context of the section is the current item of the array. The current iteration index is made available by adding a `:` after the array reference followed by the index reference name.

```js
Ractive({
  data: {
    people: [{name: 'Alice'},{name: 'Bob'},{name: 'Eve'}]
  },
  template: `
    {{#people}} {{name}} {{/people}}
    {{#people:i}} {{i}} {{name}} {{/people}}
  `
})
```

If the reference is an object _and the iteration key is provided_, the section iterates through the object properties. If no properties exist, nothing is rendered. The context of the section is the current property value. The current iteration key is made available by adding a `:` after the object reference followed by the key reference name.

```js
Ractive({
  data: {
    places: { loc1: 'server room', loc2: 'networking lab', loc3: 'pantry'}
  },
  template: `
    {{#places:key}}
      {{ key }} {{ this }}
    {{/places}}
  `
})
```

If the reference is some other truthy value or an object but not using the iteration key reference, it renders the block of markup using the reference as context. If the reference is falsy, nothing is rendered.

```js
Ractive({
  data: {
    isAdmin: true,
    foo: {
      bar: {
        baz: {
          qux: 'Hello, World!'
        }
      }
    }
  },
  template: `
    {{#isAdmin}} Hello Admin! {{/isAdmin}}

    {{#foo.bar.baz}} {{qux}} {{/foo.bar.baz}}
  `
})
```

### Inverted Sections

Inverted sections render a block of markup if the value is falsy or an empty iterable.

```js
Ractive({
  data: {
    real: false,
    people: []
  },
  template: `
    {{^real}} Nope, not real {{/real}}

    {{^people}} There's no people {{/people}}
  `
})
```

### Optional section closing text

Sections can be closed without having to match the opening text. If the section is opened with a reference and closing text is provided, the closing text must match the opening text. If the section is opened with an expression, the closing text is ignored.

```js
Ractive({
  data: {
    items: [1,2,3]
  },
  template: `
    {{#items}}
      {{this}}
    {{/items}}

    {{#items}}
      {{this}}
    {{/}}

    {{# a.concat(b) }}
      {{this}}
    {{/ I'm actually ignored but should be something meaningful like a.concat(b) }}
  `
})
```

### Handlebars-style control structures

Borrowed from Handlebars, Ractive supports `if`, `unless`, `each` and `with`. `else` and `elseif` are supported for `if`, `each` and `with`. These come in to action if prior `if` or `elseif` conditions fail, or if the `each`'s iterable is empty, or if the context referenced by `with` is missing.

```js
Ractive({
  data: {
    foo: false,
    bar: false,
    real: true,
    people: [{name: 'Alice'},{name: 'Bob'},{name: 'Eve'}]
  },
  template: `
    {{#if foo}}
      foo
    {{elseif bar}}
      bar
    {{else}}
      baz
    {{/if}}

    {{#unless real}}
      Fake
    {{/unless}}

    {{#each people}}
      Hi! I'm {{name}}!
    {{else}}
      There's nobody here
    {{/each}}

    {{#with people.3}}
      {{name}}
    {{/else}}
      Context missing
    {{/with}}
  `
})
```

### In-template partials

`{{#partial }}` defines a partial. Partials defined this way are scoped to the nearest enclosing element, or the containing component if defined at the top level of the template.

```js
Ractive({
  data: {
    people: [{name: 'Alice'},{name: 'Bob'},{name: 'Eve'}],
    places: [{name: 'server room'},{name: 'networking lab'},{name: 'pantry'}]
  },
  template: `
    {{#partial item}}
      <li class="item">{{this}}!</li>
    {{/partial}}

    <ul>
      {{#each people}}
        {{> item }}
      {{/each}}
    </ul>

    <ul>
      {{#each places}}
        {{> item }}
      {{/each}}
    </ul>
  `
})
```

### Static mustaches

`[[ ]]`, `[[& ]]` and `[[[ ]]]` render the referenced values only on the initial render. After the initial render, any changes to the referece will not update the UI, nor do any changes on bound UI elements cause the data to change. They are the one-time render counterparts of `{{ }}`, `{{& }}` and `{{{ }}}`, respectively.

```js
const instance = Ractive({
  data: {
    msg: 'Hello, World!',
    admin: false
  },
  template: `
    Will change when updated: {{ msg }}     <!-- changes to "Me, Hungry!" after the change -->
    Will not change when updated: [[ msg ]] <!-- remains "Hello, World!" after the change -->

    [[# if admin ]]
      Hello, admin
    [[else]]
      Hello, normal user
    [[/if]]
  `
})

instance.set({ msg: 'Me, Hungry!' })
instance.set('admin', true) // rendering remains 'Hello, normal user'
```

### Expressions

Expressions in mustaches are evaluated, and its result is used as the referenced value. Any changes to the expression's dependencies will re-evaluate the expression and update the rendered value.

```js
Ractive({
  data: {
    num1: 2,
    num2: 3,
    a: [1,2,3],
    b: [4,5,6],
    fn: () => true
  },
  template: `
    {{ num1 + num2 }}

    {{# a.concat(b) }} {{this}} {{/}}
    {{#each a.concat(b) }} {{this}} {{/each}}

    {{# fn() }} Yasss!!! {{/}}
    {{#if fn() }} Yasss!!! {{/if}}
  `
})
```

### Comments

`{{! }}` defines a template comment. Comments are ignored by the parser and never make it to the AST.

```html
<h1>Today{{! ignore me }}.</h1>
```

### Custom delimiters

`{{= =}}` defines custom delimiters. Custom delimiters should not contain whitespace or the equals sign.

```html
{{foo}}

{{=<% %>=}}
<% foo %>
```

### Escaping mustaches

`\` prepended on a mustache interprets the mustache as literal text.

```html
{{ ref }} \{{ ref }} <!-- value {{ ref }} -->
```

For multi-mustache structures, `\` must be prepended on all involved mustaches.

```html
\{{#if foo }} \{{ bar }} \{{/if}} <!-- {{#if foo }} {{ bar }} {{/if}} -->
```

To interpret a `\` as a literal slash before a mustache, simply prepend another `\`. Any further `\` prepended will be treated in this manner.

```html
\\{{ ref }}   <!-- \value -->
\\\{{ ref }}  <!-- \\value -->
\\\\{{ ref }} <!-- \\\value -->
```

## Keypaths

A keypath is a string that represents the location of a piece of data.

```js
Ractive({
  data: {
    foo: {
      bar: {
        baz: {
          qux: 'Hello, World!'
        }
      }
    }
  },
  template: `
    {{ foo.bar.baz.qux }}
  `
})
```

### Dot and bracket notations

Dot and bracket notation rules for keypaths are similar to vanilla JS. The only addition to this is that the dot notation can also be used to access arrays, by using the index directly on the segment.

```js
const instance = Ractive({
  data: {
    items: [1, 2, 3],
    foo: {
      bar: {
        baz: {
          qux: 'Hello, World!',
          'dotted.key': 'Me, Hungry!'
        }
      }
    },
    dynamicKey: 'bar'
  },
  template: `
    {{ foo['bar']['baz']['qux'] }} <!-- bracket notation object access -->
    {{ foo.bar.baz.qux }}          <!-- dot notation object access -->

    {{ items[0] }} <!-- bracket notation array access -->
    {{ items.0 }}  <!-- dot notation array access -->

    {{ foo.bar.baz['dotted.key'] }} <!-- dotted key access -->

    {{ foo[dynamicKey].baz.qux }} <!-- dynamic key access -->
  `
})
```

### Missing properties

In JavaScript, trying to access a child property of an object that does not exist would throw an error.

```js
const data = { numbers: [ 1, 2, 3 ]};

// Throws an error - cannot read property '0' of undefined.
const letter = data.letters[0];

// Code not reached when error happens.
someElement.innerHTML = letter;
```

Ractive would simply return `undefined` or render nothing.

```js
const instance = Ractive({
  data: {
    numbers: [ 1, 2, 3 ]
  },
  template: `
    {{ letters[0] }}
  `
})

ractive.get( 'letters[0]' ) // undefined
```

### Upstream and downstream keypaths

Ractive has this concept of "upstream" and "downstream" keypaths.

Upstream keypaths are ancestor keypaths. For the keypath `foo.bar.baz.qux`, it's upstream keypaths are `foo`, `foo.bar` and `foo.bar.baz`.

Downstream keypaths are descendant keypaths. For the `foo` keypath, it's downstream keypaths would be `foo.bar`, `foo.bar.baz` and `foo.bar.baz.qux`.

## References

A reference is a string that refers to a piece of data. A [keypath](#keypaths) is an example of a reference, one that points to a specific location in the data. [Special references](../api/references.md#special-references) are also a form of reference, one that points to a certain value.

### Reference resolution

Ractive follows the following resolution algorithm to find the value of a reference:

1. If the reference a [special reference](../api/special-references.md), resolve with that keypath.
2. If the reference is [explicit](../api/keypath-prefixes.md) or matches a path in the current context exactly, resolve with that keypath.
3. Grab the current virtual node from the template hierarchy.
4. If the reference matches an [alias](#aliasing), section indexes, or keys, resolve with that keypath.
5. If the reference matches any [mappings](../extend/components.md#binding), resolve with that keypath.
6. If the reference matches a path on the context, resolve with that keypath.
7. Remove the innermost context from the stack. Repeat steps 3-7.
8. If the reference is a valid keypath by itself, resolve with that keypath.
9. If the reference is still unresolved, add it to the 'pending resolution' pile. Each time potentially matching keypaths are updated, resolution will be attempted for the unresolved reference.

### Context stack

Steps 6 and 7 of the [resolution algorithm](#reference-resolution) defines the ability of Ractive to "climb" contexts when a reference does not resolve in the current context. This is similar to how JavaScript climbs to the global scope to resolve a variable.

To do this, whenever Ractive encounters [section mustaches](#sections) or similar constructs, it stores the context in a *context stack*. Ractive then resolves references starting with the context on the top of the stack, and popping off contexts until the reference resolves to a keypath.

<div data-playground="N4IgFiBcoE5QdgVwDbIL4BoQBcogDwDOAxjAJYAO2ABITMQLwA6422FhkA9F4vBQGsA5gDpiAewC2XGAENi2MgDcApiwB8+LiXJV1ILITwAleYtUAKYE3jVqK5JGoByAEbiAJgE9nGG3ewVSQpkWUCnAAN-O2pgYABiREIVGDQ0aJjqfAp1AHUHCUkVald5AQxY4HhZIrSAQgzMyviiwkJZIRVCNMam6gBNcURqMFlVSr4YFVkPNOpJ6Y9qcQAzSuxxbFl0NGoNreRqVvbOwhFepsHh0MIaZHEhTqWyW3FbOJvsABkHl7Tz2x9SpcY4dLo9QGZLQ5RpxXjJVLpQERPyAjxhWROayQ+YIrEXapFJzOABSZEkvguoNO+JxmX22ycAEYAAyooHzeBTGZOADMF0wF0+PyEL2J+Q88C66J8jSRdiRaAAlABuEBoIA"></div>

```js
Ractive({
  el: 'body',
  template: `
    {{#user}}
      <p>Welcome back, {{name}}!
        {{#messages}}
          You have {{unread}} unread of {{total}} total messages.
          You last logged in on {{lastLogin}}.
        {{/messages}}
      </p>
    {{/user}}
  `,
  data: {
    user: {
      name: 'Jim',
      messages: {
        total: 10,
        unread: 3
      },
      lastLogin: 'Wednesday'
    }
  }
})

// Welcome back, Jim! You have 3 unread of 10 total messages. You last logged in on Wednesday.
```

`{{# user }}` creates a context and the context stack becomes `['user']`. To resolve `name`, the following context resolution order is followed, where `name` resolves with the `user.name` keypath:

1. `user.name` (resolved here)
2. `name`

In the same way, `{{# messages }}` also creates a context. Since the `messages` section under the `user` section, the context stack becomes `['user', 'user.messages']`. To resolve `unread` and `total`, the following resolution order is followed:

`unread`

1. `user.messages.unread` (resolved here)
2. `user.unread`
3. `unread`

`total`

1. `user.messages.total` (resolved here)
2. `user.total`
3. `total`

In the case of `lastLogin`, the `user.messages.lastLogin` keypath does not exist. What Ractive does is pop off `user.messages` from the context stack and tries to resolve `lastLogin` using `user.lastLogin`. Since `user.lastLogin` is a valid keypath, `lastLogin` resolves as `user.lastLogin`.

1. `user.messages.lastLogin`
2. `user.lastLogin` (resolved here)
3. `lastLogin`

## Aliasing

Aliasing is the ability to rename one reference with another or replace one context with another.

Any section (or `{{#with}}` section) provides its own context to the template that falls within it, and any references within the section will be resolved against the section context. Ambiguous references are resolved up the model hierarchy _and_ the context hierarchy. Given a data structure that looks like
```js
{
  foo: {
    baz: 99,
    bar: {
      baz: 42
    }
  },
  list: [
    baz: 198,
    bar: {
      baz: 84
    }
  ]
}
```
and a template
```html
{{#each list}}
  explicit 1: {{.bar.baz}}
  {{#with .bar}}
    implicit 1: {{baz}}
    {{#with ~/foo}}
      explicit 2: {{.bar.baz}}
      implicit 2: {{baz}}
    {{/with}}
  {{/with}}
{{/each}}
```
there is no way to reference `~/list.0.baz` from the second implicit site because the site has a different context (`~/foo`) and using an ambiguous reference (`baz`) results in `~/foo.baz`  being used. Aliasing offers an escape hatch for similarly complex scenarios where ambiguity can cause the wrong reference to be used or performance issues to arise, because ambiguity is expensive.

Alias block use the existing `{{#with}}` mustache, but instead of setting a context, they set names for one or more keypaths. Aliases follow the form `destination as alias`, where destination is any valid reference at that point in the template e.g. `{{#with .foo as myFoo, @key as someKey, 10 * @index + ~/offset as someCalculation, .baz.bat as lastOne}}`. Because plain reference aliases, like the `myFoo` and `lastOne` aliases in the example, refer to exactly one non-computed keypath, they can also be used for two-way binding deeper in the template. For example, `<input value="{{myFoo}}" />` as a child of the alias block would bind to `.foo` in the context where the alias block is defined.

Aliasing is also extended to `{{#each}}` blocks so that the iterated item can be named rather than just referred to as `this` or `.`. For instance, `{{#each list as item}}` would make `item` equivalent to `this` directly within the `each` block, but `item` would still refer to same value in further nested contexts. Index and key aliases can still be used with an aliased iteration e.g. `{{#each object as item: key, index}}`.

Finally, partials can also be used with alias shorthand in much the same way that they can be passed context e.g. `{{>somePartial .foo.bar as myBar, 20 * @index + baz as myComp}}`.



## Conditional attributes

You can wrap one or more attributes inside an element tag in a conditional section, and Ractive will add and remove those attributes as the conditional section is rendered and unrendered. For instance:

```html
<div {{#if highlighted}}class="highlighted"{{/if}}>Highlightable element</div>
```

Any number of attributes can be used in a section, and other [Mustache](#mustaches) constructs can be used to supply attributes.

```html
<div {{#if highlighted}}class="highlighted {{ anotherClass }}" title="I'm highlighted"{{/if}}>Highlightable element</div>
```


## Attributes

Sections may also be used within attribute values and around attribute values. Using a conditional section around an attribute or group of attributes will exclude those attributes from the DOM when the conditional is `false` and include them when it is `true`. Using a conditional section within an attribute only affects the value of the attribute, and there may be multiple sections within an attribute value.

In the following terribly contrived example, if `big` is truthy, then the button will have a class `big` in addition to the fixed class `button`. If `planetsAligned` is truthy, the button will also get an annoying `onmousemove` attribute. **Note** that ractive directives cannot currently be placed within a section, but that may change in the future.

```html
<button class="{{#big}}big {{/}}button" {{#planetsAligned}}onmousemove="alert('I am annoying...')"{{/}}>I sure hope the planets aren't aligned...</button>
```










































## Expressions

### Valid expressions

These are, of course, JavaScript expressions. Almost any valid JavaScript expression can be used, with a few exceptions:

* No assignment operators (i.e. `a = b`, `a += 1`, `a--` and so on)
* No `new`, `delete`, or `void` operators
* No function literals (i.e. anything that involves the `function` keyword)

Aside from a subset of global objects (e.g. `Math`, `Array`, `parseInt`, `encodeURIComponent` - full list below), any references must be to properties (however deeply nested) of the Ractive instance's data, rather than arbitrary variables. Reference resolution follows the [normal process]().


### Does this use `eval`?

Yes and no. You've probably read that 'eval is evil', or some other such nonsense. The truth is that while it does get abused, and can theoretically introduce security risks when user input gets involved, there are some situations where it's both necessary and sensible.

But repeatedly `eval`ing the same code is a performance disaster. Instead, we use the `Function` constructor, which is a form of `eval`, except that the code gets compiled once instead of every time it executes.


### A note about efficiency

Using the `Function` constructor instead of `eval` is just one way that Ractive optimises expressions. Consider a case like this:

```html
{{a}} + {{b}} = {{ a + b }}
{{c}} + {{d}} = {{ c+d }}
```

At *parse time*, Ractive generates an [abstract syntax tree](http://en.wikipedia.org/wiki/Abstract_syntax_tree) (AST) from these expressions, to verify that it's a valid expression and to extract any references that are used. It then 'stringifies' the AST, so that the expression can later be compiled into a function.

As anyone who has seen minified JavaScript can attest, JavaScript cares not one fig what your variables are called. It also doesn't care about whitespace. So both of the expressions can be stringified the same way:

```js
"_0+_1"
```

When we *evaluate* `{{ a + b }}` or `{{ c+d }}`, we can therefore use the same function but with different arguments. Recognising this, the function only gets compiled once, after which it is cached. (The cache is shared between all Ractive instances on the page.) Further, the result of the evaluation is itself cached (until one or more of the dependencies change), so you can repeat expressions as often as you like without creating unnecessary work.

All of this means that you could have an expression within a list section that was repeated 10,000 times, and the corresponding function would be created once *at most*, and only called when necessary.




### Supported global objects

* `Array`
* `Date`
* `JSON`
* `Math`
* `NaN`
* `RegExp`
* `decodeURI`
* `decodeURIComponent`
* `encodeURI`
* `encodeURIComponent`
* `isFinite`
* `isNaN`
* `null`
* `parseFloat`
* `parseInt`
* `undefined`

### Expression dependencies

Any functions that you want to call, outside of the available globals above, must be properties of the Ractive instance's data as well. Functions can also depend on other references and will be re-evaulated when one of their dependencies is changed.

Depedendencies are determined by capturing references in the viewmodel while the function is executing. Dependencies for functions are re-captured each time the function is executed.

```html
<p>{{ formattedName() }}</p>
```

```js
var ractive = Ractive({
  template: template,
  el: output,
  data: {
    user: { firstName: 'John', lastName: 'Public' },
    formattedName: function() {
      return this.get('user.lastName') + ', ' + this.get('user.firstName')
    }
  }
};
```

Result:
```html
<p>Public, John</p>
```

In this example, the function ```formattedName``` will depend on both ```user.firstName``` and ```user.lastName```, and updating either (or ```user```) will cause any expressions referencing ```formattedName``` to be re-evaluated as well.

```js
ractive.set('user.firstName', 'Jane')
```

Result:
```html
<p>Public, Jane</p>
```











## Optimization

### Pre-parsing

Parsing templates can be a very slow operation. As an optimization option, templates can be pre-parsed outside of runtime, speeding up app initialization. Most [loaders](../integrations/loaders.md) do pre-parsing of templates as part of their build process. A parsed template is approximately 30-40% larger than the markup version, making it a trade-off between space and processing.

### Move expressions out of the template

If you use a particular expression frequently, you can save time by adding it Ractive's default data. This way you won't have to set up the expressions on each individual `ractive` instance.

The example below adds expressions for some frequenlty used parts of [moment.js](http://momentjs.com/) to the default data:

```js
var helpers = Ractive.defaults.data;
helpers.fromNow = function(timeString){
  return moment(timeString).fromNow()
}
helpers.formatTime = function(timeString){
  return moment(timeString).format("ddd, h:mmA")
}
helpers.humanizeTime = function(timeString){
  return moment.duration(timeString).humanize()
}
```
