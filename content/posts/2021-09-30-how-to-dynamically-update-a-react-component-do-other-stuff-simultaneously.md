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

```
const itemsGenerator2 = (setItems) => {
  var index = 0;
  while (index < arr.length) {
    const thenTime = Date.now()
    while (Date.now() - thenTime < 1000);  // tight wait loop
    const item = arr[index++]

    setItems((prevState) => prevState.concat(item))
  }
}
```

This functions the same as the previous generator, updating the state every second. You'll notice that it doesn't used setTimeout() to do this, however: it just runs a tight while loop, checking Data.now() continuously until a second passess for each element in the array. 

This won't work for updating MyList in real time. Instead of each item being added to the list one second apart, all items appear on the list after three seconds.

How to fix it

Because the single thread processing the event loop is all tied up checking the time, there's nothing left over for processing UI events. The solution is to create another thread where all the intense stuff goes on.  Web workers are a simple way to start up threads, so I'll use those.

```
const itemsGenerator3 = (setItems) => {
  let worker = new Worker('itemsGenerator3.js')
  worker.postMessage('Go!')
  worker.onmessage = e => {
    if (/Kill me.*/.test(e.data)) {
      worker.terminate();
    } else {
      setItems((prevState) => prevState.concat(e.data))
    }
  }
}
```

This new generator creates a worker thread, tells it to "Go!", and awaits responses.  Unless the worker asks to be terminated, each response (an item is assume) is tacked onto the items state. The code for the worker thread (also named itemsGenerator3 \[sorry]) resides in a public folder on the website (not with the source folder).

```
const arr = ["nnn", "ooo", "ppp", "qqq"];

onmessage = e => {
  if (e.data !== 'Go!')
    postMessage("no go")
  else {
    var index = 0;
    while (index < arr.length) {
      const thenTime = Date.now()
      while (Date.now() - thenTime < 1000);  // tight wait loop
      const item = arr[index++]

      postMessage(item)
    }
    postMessage("Kill me! Kill me now!")
  }
}
```

The worker awaits for the "Go!" message, and then takes off. It uses the same tight while loop as the previous generator, but does not update the items state; instead it posts a message back to the itemsGenerator3() caller. When the array is exhausted, it notifies the caller so that the worker can be terminated.

And now MyList is updated one item every second. 

What happens if we use a *real* generator function

Generator functions have a yield command that might lead you to believe that it yields the event processing thread but no--it yields a value and that is all. In fact, I see very strange goings-on if I try to use a generator function.

```

function* itemTrueGenerator() {
  var index = 0;
  while (index < arr.length) {
    const thenTime = Date.now()
    while (Date.now() - thenTime < 1000);  // tight wait loop
    const item = arr[index++]
    yield item
  }
}

const itemsGenerator4 = (setItems) => {
  const it = itemTrueGenerator()
  let item = it.next()

  while (!item.done) {
    const itemValue = item.value
    console.log({ itemValue })
    setItems(prevState => {
      console.log({ prevState, itemValue })
      prevState.concat(itemValue)
    }
    )
    console.log('next')
    item = it.next()
  }
}
```

The itemsGenerator4() function calls a true generator function, which yields values every second using that awful tight while loop. Alas, this just really messes things up. Note the console.log() statements in itemsGenerator4(). Witness:

```
App.js:56 {itemValue: 'nnn'}
App.js:56 {prevState: Array(2), itemValue: 'nnn'}
App.js:60 next
App.js:54 {itemValue: 'ooo'}
App.js:60 next
App.js:54 {itemValue: 'ppp'}
App.js:60 next
App.js:54 {itemValue: 'qqq'}
App.js:60 next
react-dom.development.js:3942 [Violation] 'message' handler took 4003ms
App.js:56 {prevState: undefined, itemValue: 'ooo'}
App.js:56 {prevState: undefined, itemValue: 'ooo'}
```

The generator is working just fine, but the setItems() call...well, not so good. I'd like to explain what is going on here, but it is beyond my mortal comprehension. I'll post a follow-up if I come across a good explaination.