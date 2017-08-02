# ReduxMiddleware

Redux Flow / Thunk / Middleware

Questions: 

What is the Redux Flow?

What is Thunk? 

When should you use Thunk? What problems does it solve?

How does Redux handle async operations? What role does Thunk play?

How does dispatching multiple actions using Thunk work?

What is Saga? How do generator functions work?

Thunk vs. Saga?

What are some disadvantages to using Thunk and other middleware with Redux?	


Redux Flow

Strict unidirectional data flow
View —> Dispatch —> Actions —> Reducer —> Store —> View
1. store.dispatch(action) is called
2. store calls reducer, passing current state tree and action to it
3. reducer takes action and current state and calculates the next state
reducer is pure function and does not mutate state
only computes the next state
every reducer returns a discrete property of the state, regardless of the number of conditions inside
combineReducers() may be used to create a single state tree from a root reducer
4. store saves the complete state tree returned by the root reducer
5. view is updated to reflect changes

Thunk

Redux action creators do not support async actions like fetch
Thunk allows you to write action creators that return a function instead of just a POJO
The inner function (thunk) can receive dispatch and getState as parameters
Thunks can return multiple dispatches
Redux Thunk middleware looks like this:

function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => next => action => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument)
    }
    return next(action)
  }
}
const thunk = createThunkMiddleware()
thunk.withExtraArgument = createThunkMiddleware
export default thunk
In the above snippet, Thunk checks to see if your action creator is returning a function and, if so, it calls that function with dispatch, getState and extraArgument as its arguments
Using Thunk with async / await: http://ohyayanotherblog.ghost.io/redux-thunk-middleware-and-async-await/
Benefits:
Async operations are now possible within action creators
Frees developers up to implementing callback logic inside action creators
Action creator can call multiple dispatches
Drawbacks:
Difficult to test and debug
Must mock all functions invoked by the logic inside thunks to test
Actions can no longer be easily serialized, which complicates logging
Logic is scattered throughout components and action creators
Dan Abramov is generally against accessing state inside action creators
Increased freedom could enable arbitrary usage and negate the point of using a framework like Redux

Saga

Saga uses generator functions to handle side effects with Redux
Rather than running to completion, generator functions can be paused and resumed
Generator functions can also return (yield) multiple values
These values are normally POJOs called effects that contain instructions to be performed by the middleware
Instructions include call, take, fork, and put, which are pure functions
To perform these tasks, create a saga that will run in the background and watch for the dispatched action and execute them according to the provided instructions
The middleware handles all invocation and and execution and returns the result to the generator
Benefits:
Testing logic is straightforward
All operations inside sagas are yielded as POJOs that get executed by the middleware
Sagas listen to actions which are dispatched as regular synchronous actions
Async operations can be tested easily
Built-in instruction functions are pure and keep in 
Drawbacks:
Similar to thunks, increased freedom can be a good or bad thing depending on usage


Three Ways to Fetch With Redux

Redux Promise Middleware
action creator can return a promise inside the action
promise must be under the action’s payload key

function getUserName(userId) {
    return {
        type: "SET_USERNAME",
        payload: fetch(`/api/personalDetails/${userId}`)
                .then(response => response.json())
                .then(json =>  json.userName)
    }
}
Upon success, the middleware automatically dispatches: “SET_USERNAME_PENDING”  and “SET_USERNAME_FULFILLED”
Upon failure, the middleware automatically dispatches: “SET_USERNAME_REJECTED”
Best usage:
Simple fetch requests

2.   Redux Thunk Middleware
As mentioned above, Thunk lets action creators return functions that take dispatch as an arg

function getUserName(userId) {
    return dispatch => {
        return fetch(`/api/personalDetails/${userId}`)
        .then(response => response.json())
        .then(json => dispatch({ type: "SET_USERNAME", userName: json.userName })
    }
}
The action creator calls dispatch inside .then() to asynchronously execute the action
Action creator can call dispatch multiple times
Best usage:
Making many fetch requests in one action
Dispatching numerous actions

3.   Redux Saga Middleware
Uses generators to execute asynchronous logic
Fetch calls occur in a saga rather than an action creator

import { call, put, takeEvery } from 'redux-saga/effects'

// call getUserName when action SET_USERNAME is dispatched
function* mySaga() {
  yield takeEvery("SET_USERNAME", getUserName);
}
function* getUserName(action) {
   try {
      const user = yield call(fetch, `/api/personalDetails/${userId}`);
      yield put({type: "SET_USERNAME_SUCCEEDED", user: user});
   } catch (e) {
      yield put({type: "SET_USERNAME_FAILED", message: e.message});
   }
}
export default mySaga

The * is called a superstar and indicates a generator function
Yield is a generator keyword

Sources:

http://blog.jakoblind.no/redux-ajax-best-practices/

https://survivejs.com/blog/redux-saga-interview/

https://medium.com/@stowball/a-dummys-guide-to-redux-and-thunk-in-react-d8904a7005d3

http://ohyayanotherblog.ghost.io/redux-thunk-middleware-and-async-await/

http://blog.isquaredsoftware.com/2017/01/idiomatic-redux-thoughts-on-thunks-sagas-abstraction-and-reusability/

https://shift.infinite.red/using-redux-saga-to-simplify-your-growing-react-native-codebase-2b8036f650de

http://jamesknelson.com/can-i-dispatch-multiple-actions-from-redux-action-creators/
