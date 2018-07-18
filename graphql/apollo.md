# Apollo + GraphQL

```bash
// the client
npm install apollo-client

// the storage mechanism
npm install apollo-cache-inmemory

// network layer
npm install apollo-link-http

// react integration
npm install react-apollo

// turn strings into graphql queries
npm install graphql-tag

// core graphql
npm install graphql
```

## Packages

### `apollo-client`

```js
import { ApolloClient } from 'apollo-client';
import { HttpLink } from 'apollo-link-http';
import { InMemoryCache } from 'apollo-cache-inmemory';

const client = new ApolloClient({
  // network layer (required)
  link: new HttpLink({
    uri: 'https://example.com/graphql'
  }),
  // cache mechanism (required)
  cache: new InMemoryCache(),
  // true when rendering on server
  ssrMode: false,
  // TBD
  ssrForceFetchDelay: 1000,
  // paired with Apollo Client DevTools
  connectToDevTools: true,
  // when false, will send query even if an identical one is already in flight
  queryDeduplication: true,
  // default option overrides for watchQuery, query, and mutate
  defaultOptions: {...}
})
```

#### Network Layer

```js
// returns a Promise
client.query({
  query: gql`{ test }`
})
```

### GraphQL

#### gql

```js
import gql from 'graphql-tag';
```

`gql` is a [tagged template literal](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals#Tagged_templates) used to define GraphQL queries. These will be parsed into a GraphQL AST. `gql` templates can be inserted into other `gql` templates.


### `react-apollo`

#### <ApolloProvider>

This component is required to make nested components Apollo-aware.

```jsx
import { ApolloProvider } from 'react-apollo';

const client = new ApolloClient({...});

ReactDOM.render((
  <ApolloProvider client={client}>
    <App />
  </ApolloProvider>
), holder);
```

#### <Query>

A component that uses a render-invoked function to render the data for a GraphQL query. `children()`, the render-invoked function, receives an object with `data`, `loading`, and `error` properties to determine what to render.

```jsx
import { Query } from "react-apollo";

<Query query={SOME_QUERY}>
  {({ loading, data, error }) => (
    loading
      ? <div>loading...</div>
      : <div>{data.someQuery.title}</div>
  )}
</Query>
```

#### <Mutation>

A component that uses a render-invoked function to call and track the same of a GraphQL mutation. `children()`, the render-invoked function, receives the mutation function and an object with `data`, `loading`, and `error` properties.

```jsx
import { Mutation } from "react-apollo";

<Mutation mutation={SOME_MUTATION}>
  {(mutate, { data, error, loading }) => (
    <div>
      <button
        onClick={e => {
          mutate();
        }}
      >
        Submit
      </button>
    </div>
  )}
</Mutation>
```

In order to call the mutation outside of an inline function, the arguments from the `<Mutation>`'s render-invoked `children()` function can be passed to another component.

```jsx
class Mutant extends React.Component {
  submit = () => {
    this.props.submit({ variables: {...} });
  }

  render() {
    if (this.props.result.loading) {
      return <div>loading...</div>;
    }
    // ...
  }
}

export default () => (
  <Mutation mutation={SOME_MUTATION}>
    {(submit, result) => (
      <Mutatnt submit={submit} result={result} />
    )}
  </Mutation>
)
```

#### graphql()

A high-order component factory; this function takes a GraphQL query and returns a function to create a higher-order component (HOC). That function takes a React component that you want to inject Apollo/GraphQL props into.

**Note:** This was a precursor to the `<Query>` and `<Mutation>` components, which are generally preferable to using the `graphql()` HOC.

```jsx
import { graphql } from "react-apollo";

const ApolloAware = graphql(gql`query { test }`)(MyComponent);
```

Injected behavior depends on query type (query, mutation, or subscription).

##### config

An optional second argument to `graphql()`.

```js
graphql(gql`...`, {
  // define component behavior with graphql data
  // behavior varies based on query type
  options: {},
  // this can also be a function that returns variables. The
  // function will receive the component's props
  options: props => ({ variables: { thing: props.thing } })
  // map function to create props object that will
  // be injected into wrapped component
  props: ({ data }) => {
    return {
      thing: data.thing
    }
  },
  // when true, skip over any GraphQL code
  // this can also be a function that returns a boolean
  skip: false,
  // change name of injected prop from data/mutate to ___
  // useful when composing multiple queries
  name: 'Bill',
  // if you need to access the wrapped component through refs
  withRef: true,
  // configure wrapper component's displayName
  alias: 'myApolloWrapper'
});
```

##### querying

When you `query`, a `data` prop is injected into the component. Alongside the queried values there are other properties on the `data` object. These include:

1. `loading` is a boolean indicating if the query is still loading
2. `error` indicates if there was an error with GraphQL

[`graphql` query options](https://www.apollographql.com/docs/react/basics/queries.html#graphql-query-options)

[The `data` prop](https://www.apollographql.com/docs/react/basics/queries.html#graphql-query-data)

##### mutating

When you `mutate`, a `mutate` prop is injected into the component. This function can then be called by the component.

```jsx
class Thing extends React.Component {
  clickHandler = e => {
    this.props.mutate({
      variables: { one: 2 }
    })
  }
}

const ApolloThing = graphql(gql`mutate updateThing(...){ ... }`)(Thing);
```

The `props` option can be used to pre-format the mutation function so it can just be called in the component.

```js
const ApolloThing = graphql(gql`mutate updateThing(...){ ... }`, {
  props: ({ mutate }) => ({
    click: arg => mutate({ variables: { one: arg }})
  })
})(Thing);
// In the component: this.props.click(1);
```

[`graphql` mutate options](https://www.apollographql.com/docs/react/basics/mutations.html#graphql-mutation-options)

Multiple mutations can be chained, but a cleaner solution would be to use `compose()`. When doing this, each mutation should be renamed using `config.name`.

An `optimisticResponse` can be passed to the `mutate` call to specify a response, which is useful for pre-rendering before a mutation's response is received.

```js
const ApolloThing = graphql(gql`mutate updateThing(...){ ... }`, {
  props: ({ mutate }) => ({
    click: arg => mutate({
      variables: { one: arg },
      optimisticResponse: {
        __typename: 'Mutation',
        updateThing: {
          __typename: 'Thing',
          one: arg
        } 
      }
    })
  })
})(Thing);
```

The `update` property of `mutate` is used to update the `cache`.

**Note:** If you try to update a query that has not been run yet, you will get an error. This can be mitigated by checking that the root query (cache.data.data.ROOT_QUERY) and the query you want to run (cache.data.data.ROOT_QUERY.<query_name>) exist. Alternatively, `refetchQueries` can be used to refetch the entire query.

```js
const ApolloThing = graphql(gql`mutate updateThing(...){ ... }`, {
  props: ({ mutate }) => ({
    click: arg => mutate({
      variables: { one: arg },
      update: (cache, { data: { updateThing } }) => {
        const data = proxy.readQuery({ query: QueryThing });
        data.thing = updateThing;
        cache.writeQuery({ query: QueryThing, data });
      }
    })
  })
})(Thing);
```

#### withApollo

```js
import { withApollo } from 'react-apollo';

const Wrapped = withApollo(MyComponent);
```

Inject your Apollo client into a component as `client`.

#### compose

Compose multiple HOCs together.

```js
import { compose } from 'react-apollo';
```

### `apollo-link-http`

This is used to setup the networking layer.

```js
import { HttpLink } from 'apollo-http-link';
const link = new HttpLink({ url: 'https://example.com/graphql' });
```

Middleware can be used to interact with requests. This can be useful for adding headers. These are created using a function exported by the `apollo-link` packages. This package also exports a `from` function which can be used to group these together.

```js
import { ApolloLink, from } from 'apollo-link';

const auth = new ApolloLink((operation, forward) => {
  operation.setContext(
    ({ headers = {} }) => {
      headers: {
        ...headers,
        other: 'thing'
      } 
    }
  );

  return forward(operation);
})

const client = new ApolloClient({
  link: from([ auth, link ]),
});
```

Afterware can be used to interact with responses. A big use case for this is detecting when a user is no longer authenticated.

```js
import { onError } from 'apollo-link-error';

const logout = onError(
  ({ networkError }) => {
    if (networkError.statusCode === 401 /* unauthorized */) {
      // call the app's logout function
    }
  }
);
```

### `apollo-cache-inmemory`

Apollo stores queried data in a cache. The cache provided by `apollo-cache-inmemory` is the default cache.

```js
import { InMemoryCache } from 'apollo-cache-inmemory';

const cache = new InMemoryCache({
  // add __typename to document
  addTypename: true,
  // normalize ids for data with unique identifier
  dataIdFromObject: function() {...},
  // resolve data from other parts of cache ???
  cacheResolvers: {}
});
```

```js
// readQuery is used to read data from the cache given a GraphQL query
// this only works on root types
cache.readQuery({
  query: SomeQuery,
  variables: { ... }
});

// readFragment can read data from anywhere in the cache (not just root)
cache.readFragment({
  id: ..., // the id of the data you want to query
  fragment: SomeFragment
});

// writeQuery/writFragment write data to the cache (but not the server).
client.writeFragment({
  id: ...,
  fragment: SomeFragment,
  data: { ... }
});
```

A cache can be hydrated using its `restore` method. This is useful for apps with server-side rendering.

```js
const cache = new InMemoryCache().restore(window.__APOLLO_STATE__);
```

`apollo-cache-persist` can be used to re-use a cache. For example, this can be used in React Native to store state when the user closes the app and restore the state the next time they launch it.
