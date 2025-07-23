# Types of FE Webapp State

Devon Coleman | Published 7/11/25

There are two distinct classes of state typically encountered when buildng a sufficiently-complex FE webapp.

## Synced state

Synced state (as the name suggests) is state that needs to be synchronized from/to some other source. This is also often referred to as "server state", but I think that fails to encapsulate flows like syncing values into IndexedDB which can be managed using the same patterns.

Local state is used by the app to track and handle user interactions — "Is this dropdown open?"/"What is the content of this text field?"-type state.
