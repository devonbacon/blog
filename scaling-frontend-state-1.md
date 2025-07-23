# Scaling Web App Frontend State: Part 1 — Concepts

Devon Coleman | **Published** 7/11/25

State in frontend web apps (I'll generally shorten this to FE) ranges in complexity from dead simple to wonderfully byzantine, varying as a product of app complexity, team size, and tech debt.

Most folks work at the simpler end of the spectrum. This isn't an indictment, but an observation — it's a reality borne from the field's relative youth and ease of entry. Browsers are pretty new tech in the grand scheme of things, and the standards are still catching up to the grand promise of a unified app platform.

Since it's where the vast majority of user share is, this is also where most public development effort and literature goes. This leaves a dearth of content on the truly complex side — when the papercuts in existing tech become showstoppers, generally in the enterprise space.

My goal with this series of posts is to bridge the gap — to provide some visibility into the types of problems faced when scaling existing technologies up, in the hopes of sparking discussion or debate for those who resonate strongly with these issues.

I expect many to dismiss much of the content here as overengineering, or overly dogmatic — that's fine! And true, in a lot of cases. If the problems described don't resonate with you, that probably means the solutions are unnecessary for your problem space. YAGNI — code is a solution that's meant to evolve as the problem does, so feel free to ignore part or all of this until and unless you actually need it.

This first post is mostly a braindump of concepts and terms that I'm likely to use in the followups (as of writing this, I haven't written any yet). Hopefully it'll serve as a decent primer.

## What do I mean by scale?

"Scale" in a backend context (e.g a restful Java service) generally means in terms of concurrent usage. A service that handles 10 requests per second (RPS) is scaled much lower than a service that handles 1000 RPS.

But this kind of scale doesn't really matter in FE dev. In FE, we're writing code that runs on users' machines — concurrent RPS is more of a concern for serving the code to the user, and generally handled by a CDN/edge provider.

So scaling the FE instead refers to complexity.

## Complexity

Complexity can be divided into two categories:

### Essential Complexity
This is the complexity inherent in the problem domain. If you're fetching data from a remote system, managing that requests' lifecycle is essential complexity.

### Accidental Complexity
You can call this tech debt. You can call it over-abstraction. Call it whatever — it's junk that gets in the way of actually solving the problem, or (frequently) makes solving the problem harder. I find FE extremely susceptible to accidental complexity, often due to applying patterns to problems that don't need them.

Some level of accidental complexity is expected. No abstraction is perfect, and code is an abstraction we can't escape.

Generally, the posts in this series should focus on identifying both the accidental and essential complexity inherent in a problem and my thoughts on minimizing the former in favor of the latter.

## Sources of Accidental Complexity

You can slice and dice this any which way you like — the discussion is obviously more complex than I can write here in a few paragraphs. In a complex FE webapp, there are generally a few key sources of accidental complexity:

### Conway's Law, or: Cross-team Collaboration is Hard, Actually

Conway's law is "you ship your org chart". Nowhere is this more true than in a complex app that contains work from multiple teams.

Each team generally has their problem domain — the thing they exist to solve. They have their own BE services, their own coding style, and their own roadmap. There's no requirement that they line up with yours — they might, if you get lucky, but usually won't.

My day job enforces certain standards — e.g BE uses java, FE uses React/TS — but leaves other things (like state systems) totally open to implementers. This can (and has) resulted in situations like libraries using one state system and apps that consume them using another.

It's a difficult balancing act — how do you mandate one approach without stifling innovation from trying other approaches? The reality seems to be that you can't — you just have to pick an approach, but try and stay agile to adapt to new requirements.

### FE is New

React was released in 2013 (incidentally, same year I graduated high school), so the component model of FE dev has really only been around for a decade. That might seem like a long time, but it really isn't — definitely not long enough for the profession to settle on a set of best practices and common wisdom the way we have in systems like databases, or real-world engineering like bridges. We're still figuring things out — there's a lot of map yet to be explored.

This means there are often many layers and abstractions that exist purely because of inertia. They were used when the code was first written, and are still in use because layers have grown up around them over time and they're now very difficult to change. Anyone who's worked with legacy FE systems knows exactly what I'm talking about.

Beyond the component model, FE tech churn is real. Every other day some new pattern pops up that promises to solve everything. Sometimes they do, but more often they don't. Unfortunately, the only true way to tell the difference (that I've seen) is time.

This is a topic we'll explore more later — the "intuitiveness" of an abstraction is also something I've seen that helps hint at a solution that might endure. Generally, the closer a solution maps to the "canonical" mental model of the problem, the more likely it is to stick around (see: React).

### JS is Loose

Compared to "stricter" languages like Java, Javascript is "loose": there's a million and one ways to accomplish a task, so you can be sure that it *will* be done in a million ways (that one remainder is horrible so nobody does it).

Since everything more or less works, you end up with this wild proliferation of patterns depending on the personal tastes of the devs at that moment in time.

Since there isn't a real definition, there's also no forcing function to get to "idiomatic" or "correct" JS, meaning a lot of FE dev is "I got it working, I don't know or care how, let's move on".

### Features, features features

Code needs shaping. If left to evolve on its own, it'll grow into something mostly functional but also weird and imperfect.

Not sure where I first heard it (I didn't come up with it), but I like to call this concept "gardening" — you're letting it grow to serve a purpose, but also guiding it.

When building a new feature, you can either understand existing patterns and attempt to fit it in, or (more likely) just do what has worked for you before — guess which one's faster?

Ultimately, the team or dev that produces business value is going to win, so this one isn't a knock on anyone — just a recognition of the reality that "pace over perfection" also means leaving a bunch of junk on the floor.

## So what now?

I don't have solutions to these problems — they're eternal, and anyone who permanently solves them can probably spin up a very expensive consulting firm.

What I *do* have are thoughts on the pieces of FE dev that are often impacted by these problems, and ideas on how to fix those pieces. I've seen async state managed hundreds of times, each in a slightly different way. I've seen both 3000-line god components that do very little and 200-line libraries that underpin the infrastructure of the entire product.

Next up, I'll build on this foundation by discussing the different types of FE app state and why they shouldn't mix.

[Back to home](./index.md) | [Part 2](./scaling-frontend-state-2.md)