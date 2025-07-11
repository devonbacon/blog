# `useEffect` considered harmful?

Devon Coleman | **Published** 7/10/25 | **Updated** 7/11/25

I hear this a lot at my day job. In a codebase as large and stratified as the one I work in, badly-written effects have caused no end of bugs.

From infinite rerender loops to subtle synchronization bugs, it really does sometimes feel like a bunch of footguns in a trenchcoat.

However, I would argue that `useEffect` is just a convenient scapegoat for the real issue:

FE developers don't think about object references.

## References

Primitives in JS are stored/passed/compared "by value". Any string `"test"` is equal to any other string `"test"`, because the contents of the strings (their actual values) is tested.

```ts
"test" === "test" // true
true === true // true
123 === 123 // true
```

In constrast, objects are stored/passed/compared "by reference". Assigning an object to a variable gives you a reference to that specific object, and that reference is what is passed around and compared.

Critically, the *contents* of the object are not checked, only its reference.

```ts
{} === {} // false
["1"] === ["1"] // false

const obj = {};
const other = obj;

other === obj // true
```

[Here is a more in-depth writeup.](https://javascript.info/object-copy)

## Why do references matter?

React uses a simple `Object.is()` check (similar to `===`, but some different edge case behavior) for most user-facing change detection.

This is how the state setter returned from `useState` knows when you're setting a new value.

This is how a component wrapped with `React.memo()` knows whether its props have changed.

And this is how `useMemo`/`useCallback`/`useEffect` determine whether entries in their dependency arrays have changed (and so, whether they should rerun).

As we've established, objects are compared by reference. So if you want React to notice changes to an object, you need to generate a new reference.

If we flip this around, we uncover the problem: React interprets a new object reference as "this data has changed", *regardless of whether it actually did*.

This is why docs, literature, and the React community put such an emphasis on immutable update patterns.

## Changes, changes everywhere, but not a bit was flipped

On the whole, folks either don't know, don't care, or simply don't understand that keeping object references stable until change occurs (what I refer to as "referential integrity") is critical to performance when scaling a React app beyond a simple SPA.

This means tons of unnecessary rerenders and effects, burning CPU for what are generally very CPU-limited apps. This causes noticeable performance issues, not to mention the potential for behavioral bugs from effects that aren't otherwise gated (the type of bug that inspired the title of this post).

To be clear, I have a lot of empathy for these engineers. This is a difficult topic to wrap your head around; it didn't fully click for me until I read [Dan Abramov's "A Complete Guide to useEffect"](https://overreacted.io/a-complete-guide-to-useeffect/).

I also believe that my personal exposure to other languages (particularly Rust, which forced me to think about lifetimes) made this type of thinking more natural for me than many, especially those who are self-taught or have only worked in FE for their entire careers.

But the fact remains that I've seen many, many critical performance and behavioral bugs introduced because folks (even otherwise excellent engineers) don't pay attention to their references.

## Why do we do this?

I think this happens because:

#### It's hard to do right (and easy to do wrong)

I aspire to the design principle "make the easy way, the only way"; design the interface so that users literally can't screw it up (or it's very clear when they're going somewhere unsafe).

To be clear, I don't know how to fix this, but I do think some of the blame rests on the relevant React apis. The design does make sense, but only once you've got the right mental model in place.

The hook apis feel like something intended for lower-level usage (and maybe that was the intent originally) but that isn't how they're being used in practice.

#### JS devs are (generally) trained to think that objects are cheap

Idiomatic JS often has us create and throw away objects with abandon. This doesn't translate well to a world where creating a new object can have effects far removed from the site of a change.

#### It doesn't matter until you get really big

React is generally fast enough. I believe these issues only show up when reaching a certain scale of company and app; more developers x more code = less coordination.

So, there isn't a lot of discussion out there about these types of issues, since most React devs probably won't ever face them.

I've seen similar patterns with other open-source/widely-used technologies such as MSW. They work very well at smaller scales where most users sit, but run into some pretty damaging papercuts/sharp edges as companies and apps scale up.

# So what can we do about it?

### **First**, get comfortable thinking about the lifetime of your data (both the container and the contents).

Consider where an object will go once you create it, and what the effects of it changing will be.

Is it being passed as a prop? Is the component it's being passed to cheap to render? Generally, the leafier you get, the less expensive a rerender is: passing an inline callback to a simple text input is much less destructive than passing an inline object to the `style` of your page container.

Over time, you'll develop a sense of how a piece of data behaves and when you need to be very careful vs when it's okay to let it slide.

### **Second**, when accessing data you don't control, take note of how well its reference is managed.

Do you need to memoize it based on other values in your component? Can you extract a subset of the object that *is* referentially stable?

For example, I once found that while the return value from a library's hook was an inline object (so, not referentially stable), the individual functions within the return value were memoized and could be passed as props directly.

### **Third**, design your apis with referential integrity in mind.

The library was clearly designed for consumers to destructure the object in the caller, but wrapping the return value in a `useMemo` would have been a cheap way to support the approach we were using as well.

### **Fourth**, (probably) use [react-compiler](https://react.dev/learn/react-compiler) once it's stable.

I haven't used it so I can't comment on how much it helps, but it appears to be aimed at solving this problem and should provide some level of referential integrity out of the box.

[Back to home](index.md)