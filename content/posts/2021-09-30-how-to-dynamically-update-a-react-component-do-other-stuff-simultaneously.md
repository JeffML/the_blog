---
template: post
title: How to Dynamically Update a React Component Do Other Stuff Simultaneously
slug: posts/dynReact
draft: true
date: 2021-09-30T21:11:06.416Z
description: Updating a React component is usually straightforward via
  useState() mechanisms. In a computationally intensive environment, though, it
  isn't so easy. I'll walk through some simple examples to illustrate.
category: Technical
tags:
  - react
  - state
  - worker
---
I ran into an issue where I had to run a continuous computation that return immediate results I wanted to display in a React component. No matter what I tried, though, the component would only update at the *end* of the computation, and only then would the intermediate results show up--all at once.

At first I thought I was confused about React state management, but after running into several dead-ends, it occured to me that the intensive process going on in the background was not allowing the updates to occur. Fortunately I can illustrate the issue (and one solution) with a few very simple examples.

How it's suppose to work

Typically one updates a component via the useState() mechanism built into React. Let's start with a simple application that displays a list:



```
function App() {
  const [items, setItems] = useState(['x', 'y']);

  useEffect(() => {
    itemsGenerator(setItems);
  }, [])


  return (
    <div className="App">
      <header > HOWDY
        <MyList {...{ items }} />
      </header>
    </div>
  );
}
```

I initialize a state called `items` with an array; the component MyList (not shown) displays what is in `items`. This is basic React stuff.

The useEffect hook starts off my item "generator" (not a generator function) when the component is loaded. 

```
const itemsGenerator = (setItems) => {
  arr.forEach((item) => {
    setTimeout(() => setItems((prevState) => prevState.concat(item)), 1000 * delayCt++)
  })
}
```

The generator uses setTimeout() to update the items state at increasing intervals of time. When I run this, I see that MyList adds new items one at a time at one second intervals. Perfect.

How to break it

As I said, with a computationally intensive operation running, this isn't so simple. It's possible to create a generator that ties up JavaScript's single event loop.