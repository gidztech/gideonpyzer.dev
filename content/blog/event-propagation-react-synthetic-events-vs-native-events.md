+++
title = "Event Propagation: React Synthetic Events vs Native Events"
description = "Recently, I came across an interesting problem involving Event Propagation in React. I want to share my discovery around React's synthetic event system, in the hope that you won't spend several hours, like I did, questioning your existing understanding of JavaScript events."
tags = [
    "javascript",
    "react",
    "synthetic",
    "events",
    "dom",
    "propagation",
    "delegation"
]
date = "2018-12-29"
highlight = "true"
+++

# 📄 Context
Recently, I came across an interesting problem involving [Event Propagation](https://www.sitepoint.com/event-bubbling-javascript/) in React. I want to share my discovery around React's synthetic event system, in the hope that you won't spend several hours, like I did, questioning your existing understanding of JavaScript events.

## Events
In order to explain the issue I had, let's briefly re-cap how events work in JavaScript. There are three phrases in Event Propagation: capture, target and bubble (in that order).

### ⛓ Capture Phase
If you specify an event listener with the [`useCapture`](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener#Parameters) option, this tells the engine to invoke the that listener first, before the target's listener. If there are multiple capture listeners for the same event, they will fire in the order they are registered.

### Ｘ Target Phase
After the capture listeners, if any, the listeners attached to the target element are invoked.

### 🛁 Bubble Phase
The final phase is where all the target parent's event listeners are invoked, all the way up to the `window` object. You can stop the event from propagating to the next parent by calling `e.stopPropagation()`, where `e` is the event.

### 👉 Event Delegation
We can avoid creating listeners for specific nodes by attaching a single event listener to a parent. We can then look at the target element (`e.target`) to determine if and what action to take by leveraging bubbling. This is called Event Delegation.

# Modal Overlay Problem
## Modal Component
We have a modal overlay component that shows up when the app's search bar is engaged. It simply displays some content over the top of the current page. 

- Clicking outside of the content area should close the modal
- Pressing the escape key should also close the modal

In order to do this, we're going to add two event listeners, a `mousedown` and a `keydown`. Since we are going to be detecting if the user is clicking outside the component, we need to use Event Delegation by attaching the listener to the document when the component mounts. 

```javascript
componentDidMount() {
    document.addEventListener('mousedown', this.handleOutsideClick);
}
```

**Remember to remove it when the component unmounts otherwise unicorns will die 🦄 🎻**

```javascript
componentWillUnmount() {
    document.removeEventListener('mousedown', this.handleOutsideClick);
}
```

In our handler, we need to check whether the element we clicked on is inside the modal container. We do this by using a [`ref`](https://reactjs.org/docs/refs-and-the-dom.html) to get hold of the container node.

```javascript
handleOutsideClick(e) {
    if !(this.modalRef.current.contains(e.target)) {
        this.closeModal();
    }
}
```

We can do the same thing for the `keydown` event, but detect the escape key instead. 

## FilterDropdown Component

Inside the modal, we need to add some content. In our case, there will be some search results and some filter options. The filters are dropdowns, and to be keyboard accessible, we should have an event listener that will close the filter when the user presses the escape key.

```javascript
handleKeyDown(e) {
    if (e.keyCode === keys.escape) {
        this.closeDropdown();
    }
}
```

```html
<div className="filter-dropdown" onKeyDown={this.handleKeyDown}>
    <ul>
    ...
    </ul>
</div>
```

If I was to open the filter dropdown and press the escape key, I might expect the dropdown to close. However, what actually happens is the whole modal closes instead. 

Remembering how events work, we'll realise that the target's event listener will fire first, but the event will bubble up the parent. The `document` also has a `keydown` event listener for the modal, so that will fire too, closing the whole modal. This is not what we want. 

Let's make a small change:

```javascript
handleKeyDown(e) {
    if (e.keyCode === keys.escape) {
        this.closeDropdown();
        e.stopPropagation();
    }
}
```

This should stop the event from bubbling to the parent, thereby preventing the modal from closing. However, when we try this, it still closes the modal. What is going wrong?

# 🔎 Investigation
## 🧪 Experiment
I put some console logs in the event handlers to see what was going on. Pressing escape on the filter had the following result:

```
FilterDropdown (React): escape triggered with stopPropagation
ModalOverlay (Native): escape triggered
```

**Native event handlers still propagate if you use `e.stopPropagation()` in React events.** So let's try converting the React event to a native one on the element.

```javascript
componentDidMount() {
    this.element.current.addEventListener('keydown', this.handleKeyDown);
}
```

**As always, think of the unicorns 🦄**

```javascript
componentWillUnmount() {
    this.element.current.removeEventListener('keydown', this.handleKeyDown);
}
```

Trying this again, the dropdown closes and the modal remains open.

```
FilterDropdown (Native): escape triggered with stopPropagation
```

## 👨‍🏫 Explanation
It turns out, React uses Event Delegation behind the scenes, and uses its own event flow to determine which handlers to invoke. 


> Event delegation: React doesn't actually attach event handlers to the nodes themselves. When React starts up, it starts listening for all events at the top level using a single event listener. When a component is mounted or unmounted, the event handlers are simply added or removed from an internal mapping. When an event occurs, React knows how to dispatch it using this mapping. When there are no event handlers left in the mapping, React's event handlers are simple no-ops.

**Source:** [https://github.com/facebook/react/issues/7094](https://github.com/facebook/react/issues/7094)

Since React uses a synthetic event system, the native event will go through the normal capture, target and bubbling phases, and then React's event flow will follow, provided that the native event doesn't stop propagation, as in my case.


## 🎉 Solution
In the mostly rare cases where you find yourself mixing React events and native events, don't! If you need Event Delegation, use only native event listeners for that particular event type. In all other cases, you should use React events, as they are come with performance benefits.

## More
There is some discussion about changing the way React's events work, with suggestions such as React Root Listeners and Element Listeners. You can read the thread [DOM Event Mount Target Considerations](https://github.com/facebook/react/issues/13713). This is all part of the strategy for [React Fire](https://github.com/facebook/react/issues/13525).