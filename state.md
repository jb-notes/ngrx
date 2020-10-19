- [State Overview](#state-overview)
  - [When thinking about state](#when-thinking-about-state)
  - [Types of State](#types-of-state)
  - [What belongs in state?](#what-belongs-in-state)
          - [References](#references)
  
# State Overview
![image](https://user-images.githubusercontent.com/21466657/96356963-085a5a80-10bb-11eb-9e0f-db453354f0ea.png)

A typical web application has the following six types of state:
  - Server state -> stored on the server and is provided via, for example, a REST endpoint
  - Persistent state -> a subset of the server state stored on the client, in memory
  - The URL and router state
  - Client state -> is not stored on the server | i.e filters used to create a list of items displayed to the user
  - Transient client state
  - Local UI state -> individual components can have local state governing small aspects of their behavior

## When thinking about state
It’s a good practice to reflect both the persistent and client state in the URL.
When identifying the type of state, ask yourself the following questions: Can it be shared? What is its lifetime.

## Types of State

A typical web application has the following six types of state:

    Server state
    Persistent state
    The URL and router state
    Client state
    Transient client state
    Local UI state

## What belongs in state?
The NgRx core team has come up with a principle called SHARI, that can be used as a rule of thumb on data that needs to be added to the store.

- Shared: State that is shared between many components and services
- Hydrated: State that needs to be persisted and hydrated across page reloads
- Available: State that needs to be available when re-entering routes
- Retrieved: State that needs to be retrieved with a side effect, e.g. an HTTP request
- Impacted: State that is impacted by other components

###### References
[https://blog.nrwl.io/managing-state-in-angular-applications-22b75ef5625f](https://blog.nrwl.io/managing-state-in-angular-applications-22b75ef5625f)