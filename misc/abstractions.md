# What Makes a Good Abstraction?

Devon Coleman | **Published** 7/25/25

"A bad abstraction is worse than no abstraction"

I'd say this is a pretty common take in software engineering. Working around a bad abstraction is often harder or more annoying than if the abstraction didn't exist in the first place.

I thought it might be interesting to write down what I think makes a good abstraction. I'm sure I'm just rehashing other folks' ideas and thoughts, so consider this more of a formalization of how I think rather than anything novel or particularly relevant.

A good abstraction has a few key elements:

## It Provides a Direct Mapping to Users' Mental Models

A good abstraction "feels right". This is pretty fuzzy criteria, but I would argue it all comes down to how naturally the abstraction matches developers' mental models of the problem space.

React, for example, provided a fantastic abstraction in the component model. Developers naturally like to think of code as composable, reusable chunks (functions) — it makes complete sense that we'd conceptualize UI in the same way.

Conversely, RTK's `createAsyncThunk` [always resolving](https://redux-toolkit.js.org/api/createAsyncThunk#unwrapping-result-actions) is a bad abstraction that breaks the mental model of promises. I've seen many bugs caused by people assuming the thunk will reject on failure.

And you're right: they should have tested the sad path, or not used a spy/mock, or read the docs. But that leads me to the second major property of a good abstraction:

## It Makes the "Right Way" Also the "Easy Way" (or Only Way)

The right abstraction makes it *really* hard for users to screw up.

Sometimes you're limited by the primitives of the language, but generally, the easier it is to do the right thing, the more likely users are to do the right thing.

Folks don't read docs. They don't test every edge case. They operate on a patchwork quilt of assumptions, experience, and luck.

This isn't a bad thing, really. It's just how the world works. We've all met the S-class bikeshedders in our career that slow everything down beyond all reason because they need the code to be perfect the first time around — that might be a path to perfection, but you're going to take way too long to get there. Moving fast and collecting feedback is (outside of safety-critical environments like medical or space systems) a much better path to providing value.

But if you really nail an abstraction, it can eliminate an entire class of bugs caused by this behavior.

Lately I've been gravitating to fluent builder patterns for complex configuration for exactly this reason.

### Abstracting Config

Consider a batcher that collects calls during some time window, then dispathes a single request to fulfill all calls.

One abstraction to represent that config would be a flat object:

```ts
const batcher = createBatcher({
    type: 'debounce',
    debounceMs: 250,
    identify: (reqArgs: { id: string }) => reqArgs.id,
    execute: requests => makeRequests(requests),
    extract: (id, results) => results[id],
})
```

This does work! But there are a few issues:

#### The types for this config object will be difficult

If you've ever written something like this, you've probably run into some difficulty with typescript's (TS from here on out) inference. It's definitely possible, but it's tricky and sometimes you do hit limits in the system that make certain valid states unrepresentatable.

#### Config fields aren't discoverable

Without intellisense, users won't have any ability to discover all the different config values aside from your docs. Even with intellisense, it can be hard to determine which values are valid (is `debounceMS` still used for a `type: 'window'` style?) and which work together.

#### Correct ordering isn't enforced

`identify` specifies the input args for a given call. `execute` consumes an array of call inputs, and `extract` consumes the results of `execute` plus an `id` to extract the results of that specific call.

If specified in this order, TS can infer the types for each function (after `identify` starts the chain), but there's no enforcement that this is the case.

Further, other fields (like `type`, and any `type`-specific config fields) can be interspersed or ordered any way the developer at the time felt like it, making the specific config hard to grok at a glance.

___

In a simple example like this, these aren't too big a deal, but in a complex system with many different config variants and options, I think this is the wrong abstraction.

This config is naturally ordered. You pick a time behavior, configure it, then build the rest of the behavior based on your `identify`/`execute`/`extract` functions.

But this abstraction doesn't match that natural ordering. It also makes it possible to do things wrong — users who haven't read the docs could pass `extract` before `execute`, requiring unnecessary type annotations and the potential for mismatches (or, if they're lazier, lots of `as any`).

So what's a *better* config abstraction?

### Fluent Builder

I think a better abstraction for config looks like this:

```ts
const batcher = createBatcher()
  .debounce(500)
  .identify((args: { id: string }) => args.id)
  .execute(requests => fetch(requests))
  .extract((results, id) => results[id])
  .build()
```

Each step is immediately discoverable via intellisense. JSDoc comments carry through, so users know exactly what each step in the config does.

TS is much more capable of inferring the builders' type through functions this way (and AI assistance like Claude is usually pretty good at producing these builders).

Importantly, the natural ordering is fully enforced by the type system. You **literally** can't define `extract` before `execute` — if you try, your code won't compile.

The only valid values are those that are presented to you for each step, and ultimately it's only a little more verbose than flat config objects.

I think this approach rises to the level of a good abstraction: It matches the natural mental model for this system, and it makes invalid states unrepresentatable (making the "right way" also the "only way").

Consider these principles when designing your own abstractions, but more importantly, when using those that others have designed. See where they stack up, see how they fail — and use that knowledge to make the boundaries between your own systems better.

[Back to home](../index.md)