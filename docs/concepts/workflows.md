# Workflows

Ractive offers a lot of configuration options but doesn't impose any specific workflow when building apps. This allows it to be very flexible and perform  any workflow you can think of.

## Two-way binding

In this workflow, data changes are carried out using two-way binding. Any changes to either the model or UI update the other and across mappings. Any UI elements bound to the updated data re-render.

```js
const InputComponent = Ractive.extend({
  data: {
    message: ''
  },
  template: `
    <input type="text" value="{{ message }}">
  `
})

const DisplayComponent = Ractive.extend({
  data: {
    message: '',
  },
  template: `
    <div>{{ message }}</div>
  `
})

const App = Ractive.extend({
  components: {
    InputComponent,
    DisplayComponent
  },
  data: {
    msg: 'Hello, World!'
  },
  template: `
    <DisplayComponent message="{{ msg }}" />
    <InputComponent message="{{ msg }}" />
  `
})

App({ target: 'body' })
```

## Data in, event out

In this workflow, data is fed into the component and extracted via events. Two-way binding is turned off to achieve one-way data flow. For a component to emit an event, [`ractive.fire()`](../api/instance-methods.md#ractivefire) is used.


```js
const InputComponent = Ractive.extend({
  twoway: false,
  data: {
    message: ''
  },
  template: `
    <input type="text" value="{{ message }}" on-keyup="emitinput">
  `,
  on: {
    emitinput(context){
      this.fire('input', context.event.target.value)
    }
  }
})

const DisplayComponent = Ractive.extend({
  twoway: false,
  data: {
    message: '',
  },
  template: `
    <div>{{ message }}</div>
  `
})

const App = Ractive.extend({
  twoway: false,
  components: {
    InputComponent,
    DisplayComponent
  },
  data: {
    msg: 'Hello, World!'
  },
  template: `
    <DisplayComponent message="{{ msg }}" />
    <InputComponent message="{{ msg }}" on-input="updatemessage" />
  `,
  on: {
    updatemessage(context, value){
      this.set('msg', value)
    }
  }
})

App({ target: 'body' })
```

## Flux

In this workflow, the Flux pattern is used. Two-way binding is turned off to achieve one-way data flow. Changes on the UI are broadcasted via dispatched actions. State changes on the store are subscribed to and the UI is updated accordingly.

```js
const InputComponent = Ractive.extend({
  twoway: false,
  data: {
    message: ''
  },
  template: `
    <input type="text" value="{{ message }}" on-keyup="handleinput">
  `,
  on: {
    handleinput(context){
      dispatch({
        action: 'INPUT_CHANGED',
        value: context.event.target.value
      })
    }
  }
})

const DisplayComponent = Ractive.extend({
  twoway: false,
  data: {
    message: '',
  },
  template: `
    <div>{{ message }}</div>
  `
})

const App = Ractive.extend({
  twoway: false,
  components: {
    InputComponent,
    DisplayComponent
  },
  data: {
    msg: 'Hello, World!'
  },
  template: `
    <DisplayComponent message="{{ msg }}" />
    <InputComponent message="{{ msg }}" />
  `,
  oninit(){
    store.subscribe(() => {
      const state = store.getState()
      this.set('msg', state.msg)
    })
  }
})

App({ target: 'body' })
```

## Fat models

TODO
