# redux for java/android (name tbd)
Redux ported to java/android

I've seen a few of these floating around, but this one has some specific benefits over other implementations.
* Any object can be used as an action or state.
* Built-in functions to help compose reducers
* Middlware that's actually implemented like you'd expect.
* A port of the thunk middleware.
* A fully-fleshed android sample.

## Usage

Create a store.
```java
ListenerStore<Action, State> store = new ListenerStore<>(initialState, reducer, middleware...);
```

Get the current state.
```java
State state = store.state();
```

Listen to state changes.
```java
store.addListener(new Listener<State>() {
  @Override
  public void onNewState(State state) {
    ...
  }
});
```

Or with rxjava.
```java
ObservableStore<Action, State> store = ...;
store.observable().subscribe(state -> { ... });
```

Dispatch actions.
```java
store.dispatch(new MyAction());
```

## Android

I suggest you use the `StateLoader` as it will tie your state updates with the activity/fragment lifecycle
```java
public class MyActivity extends AppCompatActivity implements LoaderManager.LoaderCallbacks<State> {
  ListenerStore<Action, State> store = ...;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    getSupportLoaderManager().initLoader(0, null, this);
  }
  
  @Override
  public Loader<State> onCreateLoader(int id, Bundle args) {
    return new StateLoader<>(this, store);
  }
  
  @Override
  public void onLoadFinished(Loader<State> loader, State data) {
    ...;
  }

  @Override
  public void onLoaderReset(Loader<TodoList> loader) {
  }
}
```

## Composing Reducders

It's common you'd want to switch on actions values or class type. `Reducers.matcValue()` and `Reducers.matchClass()` makes this easy.
```java
Reducer<String, State> reducer = Reducers.matchValue()
  .when("action1", new Action1Reducer())
  .when("action2", new Action2Reducer());

Reducer<Object, State> reducer = Reducers.matchClass()
  .when(Action1.class, new Action1Reducer())
  .when(Action2.class, new Action2Reducer());
```

There is also `Reducers.match()` which takes a predicate for more complicated matching setups.
```java
Reducer<Object, State> reducer = Reducers.match()
  .when(Predicates.is("action1"), new Action1Reducer())
  .when(Predicates.instanceOf(Action2.class), new Action2Reducer());
```

You can also run a sequence of reducers with `Reducers.all(reducer1, reducer2, ...)` or run reducers until one changes the state with `Reducers.first(reducer1, reducer2, ...)`.

## Thunk Middleware

```java
ListenerStore<Action, State> store = new ListenerStore<>(initialState, reducer, new ThunkMiddleware<>());

store.dispatch(new Thunk<Action, State>() {
  @Override
  public void run(Store<Action, State> store) {
    store.dispatch(new StartLoading());
    someAsyncCall(new Runnable() {
      @Override
      public void run() {
        store.dispatch(new StopLoading());
      }
    }
  }
});
```