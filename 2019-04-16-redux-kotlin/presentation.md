# Redux (Kotlin)

https://redux.js.org/introduction/three-principles

## The Store

State is stored in a single store, making debugging and inspecting the application’s current state easier.

Actions describe changes to the store’s state – these are just a passive description of how we want to change the store’s state. As they’re a passive description structure with no logic, this should make them straightforward to serialise and store (e.g. to a local JSON file or to the cloud), and this enables features like hot reloading, and recording & replay (mentioned later).

(See https://github.com/reduxjs/redux/issues/992#issuecomment-153142966 for more)


## Reducers

Reducers are pure functions that take the previous state and an action, and return the next state. Being a pure function means they return the same output when the same input is given, and have no side-effects (like accessing global variables, or making an asynchronous network call to the cloud). This makes state changes more predictable.

Given the previous state and an action, they emit the next state. Neither actions nor the store can update themselves. Reducers are just functions, so common functionality like, say, pagination can be extracted out, and functions can be composed.

## Data Flow

I can’t take credit for the original version of the chart (source: https://hackernoon.com/lessons-learned-implementing-redux-on-android-cba1bed40c41), but I think it shows the unidirectional data flow quite well.

It also shows how accessing and modifying data requires two different paths – data is accessed by observing the store, while data is indirectly modified by dispatching actions. Querying and commanding data are segregated. This might could remind you of the CQRS or Command Query Responsibility Segregation pattern.

## Implementation

There are a few existing libraries which try to port a Redux-like implementation to Kotlin:
* [ReKotlin](https://github.com/GeoThings/ReKotlin) — a port of ReSwift to Kotlin.
* [Redux Kotlin](https://github.com/pardom-zz/redux-kotlin) — another popular library, with several surrounding libraries by the same author to plug into [RxJava](https://github.com/ReactiveX/RxJava) and provide logging functionality.

If you dig inside the GitHub repositories for both of these, you might find that there’s less code to them than you might expect. Both are about 5 files of something like 50 lines each. You could roughly implement Redux yourself based on the original structure too.

## Building Our Own Implementation

Thinking about one way we might code some of these parts in Kotlin…

The Store should expose the current state as a read-only property, allow actions to be dispatched, and lastly it should allow components to subscribe to changes in state.
```kotlin
interface Store<State> {
    val state: State
    fun dispatch(action: Action)
    fun observe(): Observable<State>
}
```

Internally, it might use an Rx relay, subject, or another custom implementation.

Our state should be kept simple and immutable. In Kotlin, we have some great constructs to help with this — data classes and sealed classes. Data classes allow us to return a new copy of the state with certain properties modified very easily.

```kotlin
data class AppState(
    val session: AuthState = AuthState.unauthorised,
    val feed = FeedState(),
    val likedPosts = LikedPostsState()
)

AppState.copy(session = AuthState.authorised(token = newUserToken)
```

Data classes are also perfect for our actions, which should be extremely simple descriptions of changes. We could switch based on their class, with a `when` statement or assign type constants to them.


```kotlin
data class CreatePostAction(val text: String)
data class DeletePostAction(val id: String)
data class LikePostAction(val id: String)
```

We could specify our reducer as an `interface`, but a more idiomatic Kotlin-y way of defining a reducer could also be with a `typealias`. It takes in the previous state, and an action, and returns a new state.

```kotlin
typealias Reducer<State> = (state: State, action: Action) -> State
```

The last piece of the puzzle, linking to the UI, is left open to the implementer. Redux doesn’t define any specification for updating the UI. The original Redux library was created for React, where the UI component tree can automatically update from state changes, based on the props or data passed into a component. In ports of Redux to iOS and Android, most Redux-like library ports leave this to the implementer to decide upon – through data binding, Rx, or manual bindings.

## Action/State Playback

Actions are passive descriptions of changes made to an initial state. This means if they were stored in an array, they could accumulate a history of how an app was used in a single user’s session.

In theory it’d then be possible to record those actions to something like a JSON file, then later load them up and replay the changes from an initial state.

This also enables a form of hot-reloading for state – if the actions can be serialised and recorded to a file, it should be possible to make changes and reload the app’s state from this on-the-fly.

You could then also be able to step-through each action taken, for example in a user’s bug report, and see how the app is changing.