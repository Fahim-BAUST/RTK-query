# WHAT YOU'LL LEARN

- What RTK Query is and what problems it solves
- What APIs are included in RTK Query
- Basic RTK Query usage
  RTK Query is a powerful data fetching and caching tool. It is designed to simplify common cases for loading data in a web application, eliminating the need to hand-write data fetching & caching logic yourself.

  RTK Query solves different kinds of problems than Redux:

Tracking loading state in order to show UI spinners.
Avoiding duplicate requests for the same data.
Optimistic updates to make the UI feel faster.
Managing cache lifetimes as the user interacts with the UI.

# API Slice Parameters

When we call createApi, there are two fields that are required:

- baseQuery: a function that knows how to fetch data from the server. RTK Query includes fetchBaseQuery, a small wrapper around the standard fetch() function that handles typical processing of requests and responses. When we create a fetchBaseQuery instance, we can pass in the base URL of all future requests, as well as override behavior such as modifying request headers.

- endpoints: a set of operations that we've defined for interacting with this server. Endpoints can be queries, which return data for caching, or mutations, which send an update to the server. The endpoints are defined using a callback function that accepts a builder parameter and returns an object containing endpoint definitions created with builder.query() and builder.mutation().

## About Reducer Path

createApi also accepts a reducerPath field, which defines the expected top-level state slice field for the generated reducer. For our other slices like postsSlice, there's no guarantee that it will be used to update state.posts - we could have attached the reducer anywhere in the root state, like someOtherField: postsReducer. Here, createApi expects us to tell it where the cache state will exist when we add the cache reducer to the store. If you don't provide a reducerPath option, it defaults to 'api', so all your RTKQ cache data will be stored under state.api.

If you forget to add the reducer to the store, or attach it at a different key than what is specified in reducerPath, RTKQ will log an error to let you know this needs to be fixed

# How to export endpoints Name

RTK Query's React integration will automatically generate React hooks for every endpoint we define! Those hooks encapsulate the process of triggering a request when a component mounts, and re-rendering the component as the request is processed and data is available. We can export those hooks out of this API slice file for use in our React components.
The hooks are automatically named based on a standard convention:

- use, the normal prefix for any React hook
- The name of the endpoint, capitalized
- The type of the endpoint, Query or Mutation

#### we were able to replace the multiple useSelector calls and the useEffect dispatch with a single call to useGetPostsQuery().

## Each generated query hook returns a "result" object containing several fields, including:

- data: the actual response contents from the server. This field will be undefined until the response is received.
- isLoading: a boolean indicating if this hook is currently making the first request to the server. (Note that if the parameters change to request different data, isLoading will remain false.)
- isFetching: a boolean indicating if the hook is currently making any request to the server
- isSuccess: a boolean indicating if the hook has made a successful request and has cached data available (ie, data should be defined now)
- isError: a boolean indicating if the last request had an error
- error: a serialized error object

## important

It's also important to note that the query parameter must be a single value! If you need to pass through multiple parameters, you must pass an object containing multiple fields (exactly the same as with createAsyncThunk). RTK Query will do a "shallow stable" comparison of the fields, and re-fetch the data if any of them have changed.

### Cached data for particular time

By default, unused data is removed from the cache after 60 seconds, but this can be configured in either the root API slice definition or overridden in the individual endpoint definitions using the keepUnusedDataFor flag, which specifies a cache lifetime in seconds.

### If we want to fetch the list of users outside of React, we can dispatch the getUsers.initiate() thunk in our index file:

`store.dispatch(apiSlice.endpoints.getUsers.initiate())`

# If you're not using React Hooks, you can access refetch like this:

`const { status, data, error, refetch } = dispatch( pokemonApi.endpoints.getPokemon.initiate('bulbasaur') )`

`// has the same effect as `refetch` for the associated query dispatch( api.endpoints.getPosts.initiate( { count: 5 }, { subscribe: false, forceRefetch: true } ) )`

# Removing a subscription

Removing a cache subscription is necessary for RTK Query to identify that cached data is no longer required. This allows RTK Query to clean up and remove old cache data.

The result of dispatching the initiate thunk action creator of a query endpoint is an object with an unsubscribe property. This property is a function that when called, will remove the corresponding cache subscription.

With React hooks, this behavior is instead handled within useQuery, useQuerySubscription, useLazyQuery, and useLazyQuerySubscription.

Unsubscribing from cached data
// Adding a cache subscription
const result = dispatch(api.endpoints.getPosts.initiate())

// Removing the corresponding cache subscription
result.unsubscribe()

# Accessing cached data & request status

Accessing cache data and request status information can be performed using the select function property of a query endpoint to create a selector and call that with the Redux state. This provides a snapshot of the cache data and request status information at the time it is called.

CAUTION
The endpoint.select() function creates a new selector instance - it isn't the actual selector function itself!

With React hooks, this behaviour is instead handled within useQuery, useQueryState, and useLazyQuery.

Accessing cached data & request status
const result = api.endpoints.getPosts.select()(state)
const { data, status, error } = result

# How to write a proof of concept

A proof of concept consists of the following six fundamental steps:

- Define the idea and what it is trying to achieve, including objectives, scope and necessary resources.
- Identify and organize the team involved in the decision-making and production development process as well as any stakeholders.
- Develop and measure the success criteria by creating specific use cases within the scope.
- Test the idea in an operational environment to examine its functionality, including the design, deployment plan and success criteria.
- Perform test cases with both positive and negative scenarios to examine its durability and document the results.
- Gather and evaluate test results with the team and stakeholders to compare outcomes and see if it met the defined success criteria.
