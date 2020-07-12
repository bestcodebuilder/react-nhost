# React Nhost

Make it easy to use Nhost with React.

- `NhostAuthProvider` - AuthProvider to check logged-in state.
- `NhostApolloProvider` - ApolloProvider preconfigured with authentication for GraphQL mutations, queries and subscriptions.

If a user is logged in, the `Authorization` header will be set with your JWT token.

## Initiate

### Create React App

Add `NhostAuthProvider` and `NhostApolloProvider`.

`src/index.js`

```jsx
import React from "react";
import ReactDOM from "react-dom";
import { NhostAuthProvider, NhostApolloProvider } from "react-nhost";
import { auth } from "utils/nhost.js";
import App from "./App";

ReactDOM.render(
  <React.StrictMode>
    <NhostAuthProvider auth={auth}>
      <NhostApolloProvider
        auth={auth}
        gql_endpoint={`https://hasura-xxx.nhost.app/v1/graphql`}
      >
        <App />
      </NhostApolloProvider>
    </NhostAuthProvider>
  </React.StrictMode>,
  document.getElementById("root")
);
```

`src/utils/nhost.js`

Learn more about `auth` and `storage` in the [nhost-js-sdk](https://github.com/nhost/nhost-js-sdk) repository.

```js
import nhost from "nhost-js-sdk";

const config = {
  base_url: "https://backend-xxx-nhost.app",
};

nhost.initializeApp(config);

const auth = nhost.auth();
const storage = nhost.storage();

export { auth, storage };
```

#### Usage

**GrahQL**

```jsx
import React from "react";
import { useQuery } from "@apollo/client";
import gql from "graphql-tag";

const GET_TODOS = gql`
  query {
    todos {
      id
      created_at
      name
      completed
    }
  }
`;

export function function App() {
  const { loading, data } = useQuery(GET_TODOS);

  if (loading) {
    return <div>Loading..</div>;
  }

  return (
    <div>
      <h1>in app</h1>
      {!data ? (
        "no data"
      ) : (
        <ul>
          {data.todos.map((todo) => {
            return <li key={todo.id}>{todo.name}</li>;
          })}
        </ul>
      )}
    </div>
  );
}

```

**Auth**

````jsx
import React from "react";
import { useAuth } from "react-nhost";

export MyComponent() {
  const { signedIn } = useAuth();

  if (!signedIn) {
    return (
    <div>You are not signed in.</div>;
    );
  }

  return (
    <div>You are signed in 🎉!</div>
  );
}

---

### NextJS

_(coming soon)_

---

## Protected Route

### React Router

`src/components/privateroute.jsx`

```jsx
import React from "react";
import { useAuth } from "react-nhost";
import { Route, Redirect } from "react-router-dom";

export function PrivateRoute({ children, ...rest }) {
  const { signedIn } = useAuth();

  if (signedIn === null) {
    return <div>Loading...</div>;
  }

  return (
    <Route
      {...rest}
      render={({ location }) =>
        signedIn ? (
          children
        ) : (
          <Redirect
            to={{
              pathname: "/login",
              state: { from: location },
            }}
          />
        )
      }
    />
  );
}
````

#### Usage

```jsx
import PrivateRoute from "components/privteroute.jsx";

<Router>
  <Switch>
    /* Unprotected routes */
    <Route exact path="/register">
      <Register />
    </Route>
    <Route exact path="/login">
      <Login />
    </Route>
    /* Protected routes */
    <PrivateRoute exact path="/">
      <Dashboard />
    </Route>
    <PrivateRoute exact path="/settings">
      <Settings />
    </Route>
  </Switch>
</Router>;
```

---

### NextJS

`components/privateroute.jsx`

```jsx
import { useAuth } from "react-nhost";

export function privateRoute(Component) {
  return () => {
    const { signedIn } = useAuth();

    // wait to see if the user is logged in or not.
    if (signedIn === null) {
      return <div>Checking auth...</div>;
    }

    if (!signedIn) {
      return <div>Login form or redirect to `/login`.<div>;
    }

    return (<Component {...arguments} />);
  };
}
```

#### Usage

`pages/dashboard.jsx`

```jsx
import React from "react";
import { protectRoute } from "components/privateroute.jsx";

function Dashboard(props) {
  return <div>My dashboard</div>;
}

export default privateRoute(Dashboard);
```
