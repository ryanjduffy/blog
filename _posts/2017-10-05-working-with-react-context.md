---
layout: post
title: Working with React Context
date: 2017-10-05 20:33 -0700
---

> Originally published on Code Mentor: https://www.codementor.io/ryan286/working-with-react-context-9vgn0gdof

One of the most important concepts of React is unidirectional data flow. Data enters the system at one point and flows downstream with each component filtering and augmenting that data for its children. This is an incredibly simple but also very powerful paradigm that enables you to build complex systems with simple data flows.

<!--more-->

When trying to build generalized components that can be composed in interesting, and often unforeseen ways, I learned that passing data via props wasn’t necessarily the best solution. This was most apparent when designing “container” components that needed to react (pun intended) to changes downstream.

## Example Time

Let’s take a js-based Scroller that needs to adjust it’s scrollbar size to reflect the size of the content. (Sure, you could use native scrollers with scroll-behavior: smooth, passive event listeners, and IntersectionObserver but we’ll assume our target platform doesn’t support those or we have some other zany requirements). It might start out at first render with content twice as long so that the scrollbar is half the height.

Let’s introduce an accordion component within our scroller. How do we handle components which change in size? The typical approach might be to add an event handler to be called when the accordion opens or closes and then tell the scroller to update.

```
class ScrollerSample extends React.Component {
  handleOpenClose = () => {
  }
  render () {
    return (
      <Scroller>
        <Accordion onOpen={this.handleOpenClose} onClose={this.handleOpenClose}>
          {someChildren}
        </Accordion>
      </Scroller>
    );
  }
}
```

But how do we notify the scroller of the change in size? You could make the size of the contents a prop on the scroller and recalculate it but that misplaces the responsibility and adds a lot of boilerplate. Imaging a scroller with multiple components that resize; you’d have to wire up multiple event handlers to adapt the resize notifications and remeasure the contents just to have a functioning scroller!

Another option is an imperative API for the scroller with which you could call an update() method.

```
class ScrollerSample extends React.Component {
  handleOpenClose = () => {
    // scroller, measure thyself!
    this.scroller.update();
  }
 
  setScroller = (scroller) => {
    this.scroller = scroller;
  }
  render () {
    return (
      <Scroller ref={this.setScroller}>
        <Accordion onOpen={this.handleOpenClose} onClose={this.handleOpenClose}>
          {someChildren}
        </Accordion>
      </Scroller>
    );
  }
}
```

That’s not too bad for a single integration but can start to get out of control the more variable-sized components you have. But what if Accordion resides inside another custom component, MyMenu? You’d have to add props to that component just to pass the callbacks through to Accordion. Also, if Scroller and Accordion are from the same component library, you’d expect them to “just work” out of the box, right?

## Enter React Context

Wouldn’t it be great if React had a way to pass arbitrary data or references down the component hierarchy to provide almost seamless integration between components?

React’s Context is that magical tool (that you probably shouldn’t use)!.

But maybe you have a good use case. I’d argue that our example here is such a use case but I could be convinced otherwise. Let’s assume so and explore how Context can help solve the problem.

### How it works

Beyond props (data sent into a component from its container) and state (data maintained by a component), lies Context. Data passed in Context travels down the component tree but is only available when a component specifically requests it. Think of Context as sprinkles — any child component can have them but only if they ask nicely.

> Think of Context as sprinkles — any child component can have them but only if they ask nicely.

### Context vs Props

Context parameters are a lot like props in that they:

* Can be any type of data including functions, 
* Are defined using prop-types, 
* Are registered using a static member, contextTypes, on the component, and
* Flow down the component hierarchy.

You define the props a component sends or receives using the same means you might use for props: an object with keys representing the props and values for their types. Once defined, attach the object to the static member, contextTypes on your Component and React will pass the data from the last ancestor that provided it.
Context is mutable. Any component in the tree can provide a value to its children for any key it chooses — even if that key was used by an ancestor of it!

Here’s a simple example of a component that can receive a value from context. Since context data can’t be required like a prop can be for design-time validation, it’s important that consumers of context be resilient both to empty context and unexpected values.

```
import PropTypes from 'prop-types';
import React from 'react';
const MyComponent = (props, context /* Look, a new arg! */) => (
    <div {...props}>
        {/* Pluck value from context */}
        {context.value || 'Context was empty :('}
    </div>
);
// Tell React which context types MyComponent should receive
MyComponent.contextTypes = {
    value: PropTypes.string
};
```

While the receiving end of Context data is very similar, the sender side is a bit different. To pass data via context, the sender must:

* define the same data types as the receiver on a static member —  childContextTypes, 
* implement a special lifecycle method, getChildContext(), which will be called by React to populate context for all descendants.

The implication of the last point is that context can only be provided by “full” components (those that inherit from React.Component or are created by createClass) and not by Stateless Functional Components (SFCs).

Here’s a basic example of a component that sends a Context value down its component tree. In this example, any component created within MyProvider that also defines the same context types will receive 'My Provider' in the value key of Context.

```
import PropTypes from 'prop-types';
import React from 'react';
const MyProvider = class extends React.Component {
    getChildContext () {
        return {
            value: 'From MyProvider'
        };
    }
    render () {
        return (
            <div>
                <h1>All my children know my name</h1>
                {children}
            </div>
         );
     }
};
MyProvider.childContextTypes = {
    value: PropTypes.string
};
```

## Understanding the Data Flow

The key difference with props, though, is how context flows down the component hierarchy. Context doesn’t require you explicitly pass data through each layer of the tree; it flows downstream from the provider until its overwritten by a descendant or it reaches the last descendant.

This capability allows for some useful magic. For example, I can safely wrap MyComponent with another custom component and it will still work with MyProvider! Below, MyComponent will still display the value passed by MyProvider even though there’s another component in the middle (that isn’t forwarding any props).

```
import MyComponent from './MyComponent';
import MyProvider from './MyProvider';
const InTheMiddle = (props) => (
    <div>
      This component decorates MyComponent without impeding the data
      passed via Context.
      <MyComponent />
    </div>
);
const App = (props) => (
    <MyProvider>
        <InTheMiddle />
    </MyProvider>
);
```

const InTheMiddle = (props) => (
    <div>
      This component decorates MyComponent without impeding the data
      passed via Context.
      <MyComponent />
    </div>
);
const App = (props) => (
    <MyProvider>
        <InTheMiddle />
    </MyProvider>
);
```

## Beware, there be dragons!

Although Context is a powerful tool, it’s one that shouldn’t be used lightly. There are a two significant “gotchas” — one architectural and one technical — that should not be overlooked.

First, Context makes your applications more difficult to grok. In the typical React app, data flows from component to component via props. You can very easily see how data is passed which is powerful both when developing the app and when supporting it over time (particularly when others are supporting it!). Context data flows outside the usual path and, as such, is easy to be missed. As you refactor an app, something might stop working and it’s not clear why because the data passed via context was lost or overwritten unexpectedly.

Second, changes in context data can be prevented by a component that returns true from shouldComponentUpdate(). This “feature” caught me up recently. If you are passing data via context that can change, if a descendant prevents an update, its descendants will be stuck with a stale context value — even if they themselves later update themselves. In other words, once any component in the tree prevents updates, the rest of its subtree will be blocked from updates to context data until every component between it and the provide allows updates.

As a result, if you choose to use context, I highly recommend only passing immutable data or functions in context. It’s better to pass a getter (e.g. getValue()) than a primitive value to ensure that all descendants can always get the most recent value.

> If you choose to use context, I highly recommend only passing immutable data or functions in context.

_Hopefully that helps you understand what Context is and how to use (or avoid!) it. Have you used Context successfully? Are there other pitfalls of which others should be aware?_