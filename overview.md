- [NGRX Overview](#ngrx-overview)
  - [What is NGRX?](#what-is-ngrx)
  - [What primitives makes up NGRX?](#what-primitives-makes-up-ngrx)
  - [Whats the events pipeline of NGRX?](#whats-the-events-pipeline-of-ngrx)
  - [Actions](#actions)
    - [What makes up an action?](#what-makes-up-an-action)
    - [Three categories of actions](#three-categories-of-actions)
      - [Command actions](#command-actions)
      - [Document actions](#document-actions)
      - [Event actions](#event-actions)
    - [Action Sample file](#action-sample-file)
    - [Using Several Actions to Implement a Single Interaction](#using-several-actions-to-implement-a-single-interaction)
    - [Keep actions in sync with ids](#keep-actions-in-sync-with-ids)
    - [Processing actions](#processing-actions)
  - [Effects Classes](#effects-classes)
    - [Action Deciders](#action-deciders)
      - [Overview](#overview)
      - [Filtering Decider](#filtering-decider)
      - [Content-Based Decider](#content-based-decider)
      - [Context-Based Decider](#context-based-decider)
      - [Splitter](#splitter)
      - [Aggregator](#aggregator)
    - [Action Transformers](#action-transformers)
      - [Content Enricher](#content-enricher)
      - [Normalizer & Canonical Actions](#normalizer--canonical-actions)
  - [Reducer Example](#reducer-example)
  - [Services](#services)
    - [What is a facade?](#what-is-a-facade)
      - [Facade example](#facade-example)
  - [Selectors](#selectors)
          - [References](#references)
          - [Extra](#extra)
# NGRX Overview
![Hero](https://user-images.githubusercontent.com/21466657/96356134-09d25580-10b0-11eb-9b9e-df70d38ceef1.png)

## What is NGRX?
Programming with NgRx is programming with messages.
Messages = NGRX Actions

## What primitives makes up NGRX?
- Actions -> the message
- Effects -> which as the name suggests are side effects that happen when an action is dispatched
- Reducers -> state updater, always a newly built object, and we never mutate the state.
- Store -> which a central repository from which we can select the state from every component and service within the Angular DI

## Whats the events pipeline of NGRX?
Component -> Action -> Effects -> Action(s) -> Reducer -> Store -> Component

1. Components create and dispatch actions.
2. Action can be processed by effect classes to execute side effects
3. Action can be processed by reducers to update state, by apply action to current state
4. Component react to state changes via a subscription to the store

![image](https://user-images.githubusercontent.com/21466657/96356232-3044c080-10b1-11eb-9a48-3c7bad3ef56a.png)

## Actions
> Navigation: [What is NGRX?](#what-is-ngrx) | [NGRX Primitives](#what-primitives-makes-up-ngrx) | [Pipeline](#whats-the-events-pipeline-of-ngrx)

![image](https://user-images.githubusercontent.com/21466657/96356327-57e85880-10b2-11eb-803b-d666fbd253ae.png)

### What makes up an action?
Two properties:
- type *The type property is a string indicating what kind of action it is.*
- payload *The payload property is the extra information needed to process the action.*

```javascript
@Component({
  selector: 'todos',
  templateUrl: './todos.component.html'
})
class TodosComponent {
  constructor(private store: Store<any>) {}

  addTodo(data: {text: string}) {
    this.store.dispatch({
      type: 'ADD_TODO',
      payload: data
    });
  }
}
```

### Three categories of actions
Actions can be divided into the following three categories: commands, documents, events.
> Interestingly, the same categorization works well for most messaging systems.

#### Command actions
- Naming convention: 
  - VERB_NOUN
  - I.E. "ADD_TODO"
- Dispatching a command is akin to invoking a method: we want to do something specific.
- This means we can only have one place, one handler, for every command.
- This also means that we may expect a reply: either a value or an error.

```javascript
// Command
{
  type: 'ADD_TODO',
  payload: data
}
```

#### Document actions
- Naming convention: 
  - NOUN
  - I.E. "TODO"
- Dispatching a document is notifying the application that some entity has been updated.
- Can have multiple handlers
- Shouldn't expect a reply

```javascript
// Document
{
  type: 'TODO',
  payload: data
}
```

#### Event actions
- Naming convention: 
  - NOUN
  - I.E. "TODO"
- Dispatching a event we notify the application about an occurred change.
- Can have multiple handlers
- Shouldn't expect a reply

```javascript
// Document
{
  type: 'TODO',
  payload: data
}
```

### Action Sample file
```javascript
export enum DashboardActionTypes {
    AddTile = '[Dashboard] ADD_TILE',
    RemoveTile = '[Dashboard] REMOVE_TILE',
    UpdateTile = '[Dashboard] UPDATE_TILE'
}

export const addTile = createAction(
    DashboardActionTypes.AddTile, // action name
    props<{ payload: Tile }>() // action payload type
);

export const removeTile = createAction(
    DashboardActionTypes.RemoveTile,
    props<{ payload: string }>()
);

export const updateTile = createAction(
    DashboardActionTypes.UpdateTile,
    props<{ payload: Tile }>()
);
```

### Using Several Actions to Implement a Single Interaction
We often use several actions to implement an interaction. Here, for instance, we use a command and an event to implement the todo addition. We handle the ADD_TODO command in an effects class, and then the TODO_ADDED event in the reducer.

```javascript
class TodosEffects {
  constructor(private actions: Actions, private http: Http) {}

  @Effect() addTodo = this.actions.ofType('ADD_TODO').
    concatMap(todo => this.http.post(…).
    map(() => ({type: 'TODO_ADDED', payload: todo})));
 }

function todosReducer(todos: Todo[] = [], action: any): Todo[] {
  if (action.type === 'TODO_ADDED') {
    return [...todos, action.payload];
  } else {
    return todos;
  }
}
```



### Keep actions in sync with ids
Using the id of document at a *correlation id*,
if no document id available
we can always create a synthetic id

### Processing actions
![image](https://user-images.githubusercontent.com/21466657/96356528-67b56c00-10b5-11eb-9e45-218297ff1635.png)
- A dispatched action can be processed by effects classes and reducers.
- Reducers are simple: they are synchronous pure functions creating new application states.
  - Reducers don’t deal with the hard parts: 
    -  asynchrony 
    -  process coordination
    -  talking to the server

Effects classes deal with all of these. 
And that’s where, it turns out, many patterns used for building message-based systems on the backend work really well.

## Effects Classes
![image](https://user-images.githubusercontent.com/21466657/96356554-bebb4100-10b5-11eb-936b-eff3cd20a0c2.png)

Effects classes have three roles:

  - Action deciders -> They decide on how to process actions. 
  - Action Transformers -> They transform actions into other actions.
  - TThey perform side effects.

### Action Deciders
- An action decider determines if an effects class should process a particular action.
- A decider can also map an action to a different action.

#### Overview
![image](https://user-images.githubusercontent.com/21466657/96356789-0c857880-10b9-11eb-9c7a-4003c4ffdddc.png)

The most common deciders:
  - A filtering decider uses the action type to filter actions.
  - A content-based decider uses the action payload to map an action to a different action.
  - A context-based decider uses some injected object to map an action to another one.
  - A splitter maps an action to an array of actions.
  - A aggregator maps an array of actions to a single action.

#### Filtering Decider
![image](https://user-images.githubusercontent.com/21466657/96356598-59b41b00-10b6-11eb-9e99-6b6c6cb7bd48.png)

- Filter dispatched actions for a specific action type

```javascript
class TodosEffects {
  constructor(private actions: Actions, private http: Http) {}

  @Effect() addTodo = this.actions.ofType('ADD_TODO').
    concatMap(todo => this.http.post(…).
    map(() => ({type: 'TODO_ADDED', payload: todo})));
}
```

#### Content-Based Decider
![image](https://user-images.githubusercontent.com/21466657/96356614-a26bd400-10b6-11eb-9ef4-02874d019d96.png)

- A content-based decider uses the payload of an action to map it to a different action.

In the following example, we are mapping ADD_TODO to ether APPEND_TODO or INSERT_TODO, depending on the content of the payload.

```javascript
class TodosEffects {
  constructor(private actions: Actions) {}

  @Effect() addTodo = this.actions.typeOf('ADD_TODO').map(add => {
    if (add.payload.addLast) {
      return ({type: 'APPEND_TODO', payload: add.payload});
    } else {
      return ({type: 'INSERT_TODO', payload: add.payload});
    }
  });
}
```

#### Context-Based Decider
![image](https://user-images.githubusercontent.com/21466657/96356636-13ab8700-10b7-11eb-8858-6cfd9ccfa8c3.png)

- A context-based decider uses some information from the environment to map an action to a different action.
- Using it allows us to have distinct workflows the component dispatching the action is not aware of.

In the following example, we are using ```env.confirmation``` to determine which action to dispatch.

```javascript
// 
class TodosEffects {
  constructor(private actions: Actions, private env: Env) {}

  @Effect() addTodo = this.actions.typeOf('ADD_TODO').map(addTodo => {
    if (this.env.confirmationIsOn) {
      return ({type: ‘ADD_TODO_WITH_CONFIRMATION’, payload: addTodo.payload});
    } else {
      return ({type: ‘ADD_TODO_WITHOUT_CONFIRMATION', payload: addTodo.payload});
    }
  );
}
```

#### Splitter
![image](https://user-images.githubusercontent.com/21466657/96356674-a3513580-10b7-11eb-8bdb-5bb91780978f.png)

- A splitter maps one action to an array of actions, i.e., it splits an action.

In the following example, we are mapping `REQUEST_ADD_TODO` to `ADD_TODO` and `LOG_OPERATION`

```javascript
// Splitter Effect
class TodosEffects {
  constructor(private actions: Actions) {}

  @Effect() addTodo = this.actions.typeOf('REQUEST_ADD_TODO').flatMap(add => [
    {type: 'ADD_TODO', payload: add.payload},
    {type: 'LOG_OPERATION', payload: {loggedAction: 'ADD_TODO', payload: add.payload}}
  ]);
}
```

#### Aggregator
![image](https://user-images.githubusercontent.com/21466657/96356741-b9132a80-10b8-11eb-9177-5f831e6228d0.png)

- An aggregator maps an array of actions to a single action.

In the following example, we are mapping `REQUEST_ADD_TODO` to `ADD_TODO` and `LOG_OPERATION`

```javascript
// Aggregator Effect
class TodosEffects {
  constructor(private actions: Actions) {}

  @Effect() aggregator = this.actions.typeOf(‘ADD_TODO’).flatMap(a =>
    zip(
      // note how we use a correlation id to select the right action
      this.actions.filter(t => t.type == 'TODO_ADDED' && t.payload.id === a.payload.id).first(),
      this.actions.filter(t => t.type == ‘LOGGED' && t.payload.id === a.payload.id).first()
    )
  )).map(pair => ({
    type: 'ADD_TODO_COMPLETED',
    payload: {id: pair[0].payload.id, log: pair[1].payload}
  }));
}
```

### Action Transformers
Two types:
- Content Enricher -> adds some information to an action’s payload.
- Normalizer & Canonical Actions -> A normalizer maps a few similar actions to the same (canonical) action.

#### Content Enricher
![image](https://user-images.githubusercontent.com/21466657/96356846-b6650500-10b9-11eb-87e2-172a836e2255.png)

A content enricher adds some information to an action’s payload.

```javascript
// Content Enricher
class TodosEffects {
  constructor(private actions: Actions, private currentUser: User) {}

  @Effect() addTodo = this.actions.ofType('ADD_TODO').
    map(add => ({
       action: 'ADD_TODO_BY_USER',
       payload: {...add.payload, user: this.currentUser}
    }));
}
```
#### Normalizer & Canonical Actions
![image](https://user-images.githubusercontent.com/21466657/96356856-e01e2c00-10b9-11eb-8047-3664e4a39451.png)


A normalizer maps a few similar actions to the same (canonical) action.

```javascript
// Normalizer & Canonical Actions
class TodosEffects {
  constructor(private actions: Actions) {}

  @Effect() insertTodo = this.actions.ofType('INSERT_TODO').
    map(insert => ({
      action: 'ADD_TODO',
      payload: {...insert.payload, append: false}
    }));

  @Effect() appendTodo = this.actions.ofType('APPEND_TODO').
    map(insert => ({
      action: 'ADD_TODO',
      payload: {...insert.payload, append: true}
    }));
}
```

## Reducer Example

```javascript
// Reducer
function todosReducer(todos: Todo[] = [], action: any): Todo[] {
  if (action.type === 'ADD_TODO') {
    return [...todos, action.payload];
  } else {
    return todos;
  }
}
```
Another example
```javascript
// we create the state by adding an empty tile_

const emptyTile = new Tile(undefined);
const initialState = dashboardAdapter.addOne(
    emptyTile,
    dashboardAdapter.getInitialState()
);

export const dashboardReducerFn = createReducer(
    initialState,
    on(addTile, (state, { payload }) => {
        return dashboardAdapter.addOne(payload, state);
    }),
    on(removeTile, (state, { payload }) => {
        return dashboardAdapter.removeOne(payload, state);
    }),
    on(updateTile, (state, { payload }: { payload: Tile }) => {
        return dashboardAdapter.updateOne(
            { id: payload.id, changes: { assetId: payload.assetId } },
            state
        );
    })
);

export function dashboardReducer(
    state = initialState,
    action: Action
): EntityState<Tile> {
    return dashboardReducerFn(state, action);
}
```

## Services
A State Management service module only needs to export two things:
  - the module itself in order to register its providers
  - a facade service that acts as a bridge between the UI components of our feature module and the store

### What is a facade?
A Facade, in terms of software engineering, is implemented as an object that offers a unified and simpler interface behind a more complex system.

In other terms, it abstracts the complex system (NGRX) behind a single Service.

What advantages does this pattern have?
  - we abstract UI components from the State Management used
  - we simplify the interfaces using clear, small methods
  - we minimize the number of dependencies used by a component
  - we provide a central service to fetch data and dispatch commands
  - if we import the module from a lazy loaded route — this will be imported only when the route is loaded. Sometimes, you may  need multiple feature modules in a specific route — in which case you may be forced to import them from AppModule as well
  - Better separation/encapsulation from the UI. The components don’t need to know what state-management you’re using
  - We can refactor/change the state-management
  - Abstraction Let’s say we start this project using NGRX and one day we decide to switch to NGXS, Akita, or some other State Management tooling. By using facades, we never have to refactor components and services that rely on the library used.
  - Simplicity A facade will hide away the technicalities and implementation of the library we use from the consumers, which result in components being lean and simple.
  - Reusability A facade will help with reusing some of the code used to dispatch and create actions, or select fragments from the store, as you never need to write that twice.

#### Facade example
```typescript
@Injectable()
export class DashboardFacadeServiceImpl implements DashboardFacadeService {
    public tiles$: Observable<Tile[]> = this.store.select(selectAllTiles);

    constructor(private store: Store<EntityAdapter<Tile>>) {}

    addTile(payload: Tile) {
        this.store.dispatch(addTile({ payload }));
    }

    updateTileAsset(id: string, assetId: string) {
        this.store.dispatch(updateTileAsset({ payload: { id, assetId } }));
    }
}
```
In the UI
```typescript
@Component({
    selector: 'cf-dashboard',
    templateUrl: './dashboard.component.html',
    styleUrls: ['./dashboard.component.scss'],
    changeDetection: ChangeDetectionStrategy.OnPush
})
export class DashboardComponent implements OnInit {
    public tiles$ = this.dashboardFacade.tiles$;

    constructor(
        private dashboardFacade: DashboardFacadeService,
        private assetsFacade: AssetsFacadeService
    ) {}

    ngOnInit() {
        this.assetsFacade.getAssets();
    }

    addTile() {
        this.dashboardFacade.addTile(new Tile(undefined));
    }
}
```

## Selectors
Selectors are simply functions we define to select information from the store’s object.

Pro Tip: selectors are super useful, always write granular selectors and try to encapsulate logic within selectors rather than in your services or components
  
```javascript
// selectors
import { createSelector, createFeatureSelector } from '@ngrx/store';const selectDashboardState = createFeatureSelector('dashboard');

export const selectAllWidgets = createSelector(
    selectDashboardState, 
    (state: DashboardState) => state.widgets
);

// service
export class DashboardRepository {
    widgets$ = this.store.select(selectAllWidgets);constructor(private store: Store<DashboardState>) {}
}
```

###### References
[https://blog.nrwl.io/ngrx-patterns-and-techniques-f46126e2b1e5](https://blog.nrwl.io/ngrx-patterns-and-techniques-f46126e2b1e5)
[https://itnext.io/state-management-with-ngrx-introduction-1aae0803e988](https://itnext.io/state-management-with-ngrx-introduction-1aae0803e988)
[https://angularbites.com/abstracting-state-with-ngrx-facades/](https://angularbites.com/abstracting-state-with-ngrx-facades/)

###### Extra
[https://angular-checklist.io/](https://angular-checklist.io/)