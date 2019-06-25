---
layout: post
title: Passing Functions as React Props
date: 2017-07-10 20:47 -0700
---

> Originally posted on Code Mentor: https://www.codementor.io/ryan286/passing-functions-as-react-props-9fimj8ikv

I recently had a discussion with a coworker about the naming of a function property that will be passed to a React component. It got me thinking more thoroughly about the role of functions as properties. I find that they usually fall into one of two camps. The function either:

* Notifies the container of user or component action, or
* Provides some capability that the component needs.

<!--more-->

## Event Callbacks

Let's call the first an "Event Callback." This lines up well with the native event hooks provided by React. You, the component author, can provide a function (`this.handleClick`) via the prop (`onClick`) of your `<button>` component like so:

```js
class MyComponent extends React.Component {
    handleClick = (ev) => {
        if (ev.keyCode === 13) {
            console.log('Enter!');
        }
    }
    
    render () {
        return (
            <button onClick={this.handleClick}>
            	Click it real good
            </button>
        );
    }
}
```

If `<button>` was a "regular" React component, you might expect it to have a `propType` defined that will declare that `onClick` expect a function.

```js
onClick: PropTypes.func
```

What this means, intuitively, is that `<button>` accepts a function in the `onClick` prop, which will be called when the button is clicked. It doesn't, however, imply what it "means" to click the button. Does it submit a form? select an option? play a [ninja cat video](https://twitter.com/sarahjorden_/status/877005793492692992)? That responsibility lies within the **container** and is the defining characteristic of an Event Callback:

> With an Event Callback, a component is responsible for notifying the container that a user or component action has occurred, but not what the result of that action should be.

## Dependency Injection

Sometimes, a component either requires a feature be implemented by its container or allows for its container to override a feature's default behavior. 

An example of this comes from the popular [`react-router`](https://www.npmjs.com/package/react-router) package. 

[`NavLink#isActive`](https://reacttraining.com/react-router/web/api/NavLink/isActive-func) accepts a function as a prop, which is used by the component to override the default behavior of marking a link active if it matches the current URL's pathname.

Here's an example lifted directly from the package docs:x`

```js
// only consider an event active if its event id is an odd number
const oddEvent = (match, location) => {
  if (!match) {
    return false
  }
  const eventID = parseInt(match.params.eventID)
  return !isNaN(eventID) && eventID % 2 === 1
}

<NavLink
  to="/events/123"
  isActive={oddEvent}
>Event 123</NavLink>
```

In this example, the container doesn't need to be notified of anything; instead, it is injecting a new, custom behavior.

> With Dependency Injection, the component is responsible for describing the contract for the feature (via the function's interface). The container will provide the feature.

## Implementation Differences

A consistent API makes for a predictable API, which improves the overall developer experience. Below are a few implementation best practices for each type of function that leads to consistent components and happy consumers!

Event Callbacks should be:
* **Optional:** There are exceptions, but normally should be at the discretion of the consumer to handle an event
* **Named consistently:** Prefix all event callbacks with _`on`_, e.g. `onSelectThing`
* **Single argument:** Pass an object argument with relevant data for the event, e.g. `onSelectThing({index: 1})`
* **Unidirectional:** Only pass data via the callback and not rely on a return value from the callback

Dependency Injection functions should be:
* **Unidirectional:** Only pass behavior into a component and not expect feedback from the function (or its parameters!)
* **Named appropriately:** often beginning with a verb, e.g. `isActive` or `generateId`

## Wrap Up

Hopefully this post helps you make consistent decisions when accepting functions as a prop to your React components. Have you encountered other use cases for functions as props? Do you disagree with the best practices above or have other recommendations? Please comment below :)