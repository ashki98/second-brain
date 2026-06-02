# React/ Redux/ Redux-Saga

Here's a comprehensive summary of what we've discussed:

1.**React, Redux, and Redux-Saga Relationship**

```jsx
// Layer Structure
React (UI Layer)
  ↓
Redux (State Management Layer)
  ↓
Redux-Saga (Side Effects/Async Operations Layer)
  ↓
API/External Services
```

1. **Data Flow in Redux with Saga**

```jsx
// Complete Flow
UI Action
  → dispatch (Redux function, accessed via useDispatch hook)
    → reducer AND saga middleware (parallel)
      → saga makes API call
        → saga puts new action
          → reducer updates state
            → UI updates (via useSelector)

```

1. **Key Functions & Their Origins**

```jsx
// Memory Only (Lost on refresh/close)
React State:  Component-level state using useState
Redux Store:  Global application state

// Browser Storage (Persistent)
localStorage:    Permanent until cleared
sessionStorage:  Lasts until tab closes
cookies:        Can set expiration
```

1. **State Management & Storage**

```jsx
// Memory Only (Lost on refresh/close)
React State:  Component-level state using useState
Redux Store:  Global application state

// Browser Storage (Persistent)
localStorage:    Permanent until cleared
sessionStorage:  Lasts until tab closes
cookies:        Can set expiration

```

1. **Common Patterns**

**Component with Redux:**

```jsx
// Watcher Saga
function* watchFetchUser() {
  yield takeEvery('FETCH_USER', fetchUserSaga);
}

// Worker Saga
function* fetchUserSaga(action) {
  try {
    const user = yield call(api.fetchUser, action.payload);
    yield put({ type: 'SET_USER', payload: user });
  } catch (error) {
    yield put({ type: 'SET_ERROR', payload: error });
  }
}
```

**Reducer Pattern:**

```jsx
// Handle both initial action and API response
const reducer = (state, action) => {
  switch(action.type) {
    case 'FETCH_USER':
      return { ...state, loading: true };
    
    case 'SET_USER':
      return { 
        ...state, 
        loading: false,
        user: action.payload 
      };
  }
};
```

**Saga Pattern:**

```jsx
// Watcher Saga
function* watchFetchUser() {
  yield takeEvery('FETCH_USER', fetchUserSaga);
}

// Worker Saga
function* fetchUserSaga(action) {
  try {
    const user = yield call(api.fetchUser, action.payload);
    yield put({ type: 'SET_USER', payload: user });
  } catch (error) {
    yield put({ type: 'SET_ERROR', payload: error });
  }
}
```

1. **Key Concepts**

**Callbacks:**

```jsx
// Function passed as argument, executed later
function doSomething(callback) {
  // do work
  callback();
}

doSomething(() => console.log('Done!'));
```
