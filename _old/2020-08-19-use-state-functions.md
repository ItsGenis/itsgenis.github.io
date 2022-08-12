---
layout: post
title: TIL Store functions in React's useState()
slug: store-functions-react-usestate
tags: React TIL
excerpt: Storing functions in React's useState hook requires shenanigans
image: "/images/posts/react.png"
---

_I'm starting a new category of low effort posts where I just store small doses of knowledge._

If you want to store a function into a React's `useState()` you should store an anonymous function that returns the function you'd like to store, like so:

```typescript
const [func, setFunc] = useState();

const someFunctionToStore = () => console.log("hey");

setFunc(() => someFunctionToStore);
```

Otherwise the stored function will be called in one of the React hooks cycles, and we don't want that.
