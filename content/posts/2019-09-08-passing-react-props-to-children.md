---
template: post
title: Passing React props to children
slug: /posts/reactprops
draft: false
date: 2019-09-08T23:53:12.905Z
description: Yet another way to pass self-named props to child components
category: Technical
tags:
  - react
  - props
---
![Oliver wants more gruel](/media/illustration-depicting-oliver-twist-asking-for-more-food-by-j-mahoney-517216578-59f93a010d327a0010b2dd78.jpg)

Yes, this is a topic that's been [done](https://flaviocopes.com/react-pass-props-to-children/) [to](https://medium.com/better-programming/passing-data-to-props-children-in-react-5399baea0356) [death](https://zhenyong.github.io/react/docs/jsx-spread.html).  Lately, though, I've started using a technique that I haven't come across on the interwebs. It's not particularly clever, so I'm sure it's not novel, but it is concise. Why not write about it?

# Props the verbose way

I'll base my examples on a [React application](https://pokermap.netlify.com/) that uses [jsheatmap](https://www.npmjs.com/package/jsheatmap), a package I wrote for generating heat map data.  The presentation of the heat map is done via a `<table>`, where each cell's background color is set to an RGB value that jsheatmap generates from a [given set of input values](https://www.freecodecamp.org/news/a-heat-map-implementation-in-typescript/). 

```js
const HeatMapTable = () => {
  const [players, setPlayers] = useState(2);
  const [suited, setSuited] = useState(false)
  const [ties, setTies] = useState(false)
  const [data, setData] = useState(getNewData(players, suited, ties))
```

There is a PlayersRow component that contains controls to allow the user to set the number of players needed to determine certain poker odds. It needs not only the initial value, but a setter to set new values. These properties (props) are `players` and `setPlayers`.

One could use the time-honored technique of passing these props as attributes when including the component in its container (HeatMapTable).

```js
<Players players={players} setPlayers={setPlayers} />
```

Pretty basic stuff. 

# Props the concise way

In this case (as often happens), the variables used to hold the props often have the same names as the prop attribute names. This allows a more concise syntax to pass the props down to the child.

In our example, there are two props, but you might have many more, so one technique is to use an intermediate object to hold the props that are needed by the child component, then use the spread operator to "expand" the prop object into attribute values.

```js
const props = {players, setPlayers, anotherProp, yetAnotherProp, etc};
<Players {...props} />
```

Which is equivalent to the much more verbose:

```js
<Players players={players} setPlayers={setPlayers} anotherProp={anotherProp} yetAnotherProp={yetAnotherProp} etc={etc} />
```

# Props the conciser way

The intermediate variable is not really needed, because you can just do this:

```js
<Players {...{players, setPlayers, anotherProp, yetAnotherProp, etc}} />
```

Or even something that mixes intermediate objects with the spread operator:

```js
const props = {anotherProp, yetAnotherProp, etc};
<Players {...{players, setPlayers, ...props}} />
```

That's pretty much it:  a little trick I find myself using more often these days.
