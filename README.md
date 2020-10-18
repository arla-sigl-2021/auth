# Auth workshop

This workshop targets students of EPITA SIGL 2021.

You will integrate an SAAS IDP called [Auth0](https://auth0.com/) to Arlaide

## Step 1: Setup your account on Auth0

Go to https://auth0.com/auth/login and create an account with your github account.

Select the EU region to avoid to cross the world for a request and use a personnal account type.

Once your account created, you will have access to auth0 home page
![home Page](img/home-page.png)

To use Auth0 service in yours app, you need credentials. To get them, go to the Applications section in Auth0 dashboard.

A default Application is already available but we will create a new one called Arla:

- Click on Create Application button
- Name it `Arla` and select `Single Page Web Application`
- Click on Create

The dashboard propose you to see a Quick Start with the different technology. You can check it if you want. Some examples are available in Quick Start.

This TP is inspired from React Quick Start.

Now, we have credentials available in the settings tab of our application. For the first part, we only need of Domain and clientId.

Before starting to code, we need to configure the Callback URLs, Logout URLs, Allowed Web Origins.

You can put these 2 URLs (with comma sperator):

- http://localhost:8080
- https://groupe<Number>.arla-sigl.fr

The first will be use for local developpement and the second for our final app on yours server.

## Step 2: Integrate Auht0 login to your frontend

### Install the Auth0 React SDK

Auth0 provide an easy to use sdk for react.

First we need to install auth0 react sdk:

```
npm i -save @auth0/auth0-react
```

You can now import the sdk and start to configure the `Auth0Provider` component.

Under the hood, the Auth0 React SDK uses React Context to manage the authentication state of your users. One way to integrate Auth0 with your React app is to wrap your root component with an Auth0Provider that you can import from the SDK.

In your `app.tsx`, add:

- `import { Auth0Provider } from "@auth0/auth0-react";` with yours import
- `Auth0Provider` configuration

```typescript
...
import React from "react";
import ReactDOM from "react-dom";
import { Auth0Provider } from "@auth0/auth0-react";
...

ReactDOM.render(
    <Auth0Provider
    domain="YOUR_DOMAIN"
    clientId="YOUR_CLIENT_ID"
    redirectUri={window.location.origin}
    cacheLocation="localstorage"
    >
        <Authenticated>
            <TemplateMachineProvider>
            ...
            </TemplateMachineProvider>
        </Authenticated>
</Auth0Provider>
```

The Auth0Provider component takes the following props:

- `YOUR_DOMAIN` and `YOUR_CLIENT_ID`: The values of these properties correspond to the "Domain" and "Client ID" values present under the "Settings" of the single-page application that you registered with Auth0. (Be careful if you have a public repo to not pushed yours credentials)
- `redirectUri`: The URL to where you'd like to redirect your users after they authenticate with Auth0.

`Auth0Provider` stores the authentication state of your users and the state of the SDK — whether Auth0 is ready to use or not. It also exposes helper methods to log in and log out your users, which you can access using the useAuth0() hook.

### Add Login to Your Application

The Authenticated is here to automaticly redirecxt the user to Auth0 when it's not logged.
To do that:

- Create a file `Authenticated.tsx` in `components`
- Put the code below inside

```typescript
import React from "react";
import { useAuth0 } from "@auth0/auth0-react";

export const Authenticated: React.FC = ({ children }) => {
  const { loginWithRedirect, user, isLoading } = useAuth0();

  React.useEffect(() => {
    const redirect = async () => {
      if (!user && !isLoading) {
        await loginWithRedirect();
      }
    };
    redirect();
  }, [isLoading]);
  return isLoading ? <span>Loading ...</span> : <>{children}</>;
};
```

We used a React useEffect to automaticly redirect the user.

### Add Logout to your Application

`useAuth0` hook is providing a `logout` method enabling the user to kill his active auth session.

Create a new `src/components/Logout.tsx` component with:

```tsx
import { useAuth0 } from "@auth0/auth0-react";
import { Button } from "@material-ui/core";
import React from "react";

export const Logout: React.FC = () => {
  const { logout } = useAuth0();
  return (
    <Button
      variant="outlined"
      color="primary"
      onClick={() => {
        logout({ returnTo: document.location.origin });
      }}
    >
      Logout
    </Button>
  );
};
```

And add the following changes to your main `TemplateLayout.tsx` component:

```tsx
import React from "react";
import { makeStyles, createStyles, Theme } from "@material-ui/core/styles";
import Grid from "@material-ui/core/Grid";
import { Typography } from "@material-ui/core";
import { TemplateMachineContext } from "../state/provider";
import { Logout } from "./Logout";
import { useAuth0 } from "@auth0/auth0-react";

// ...

export const TemplateLayout: React.FC = ({ children }) => {
  // ...
  const { user } = useAuth0();

  return (
    <Grid
      className={classes.root}
      container
      direction="row"
      justify="center"
      alignItems="center"
      spacing={3}
    >
      <Grid item xs={12}>
        <Typography variant="h1" align="center" gutterBottom>
          Frontend Workshop
        </Typography>
        <Typography variant="subtitle1" align="center">
          {user?.email || "EPITA SIGL 2021"}
        </Typography>
        <Typography variant="subtitle2" align="center" color="primary">
          <Logout />
        </Typography>
      </Grid>
      <Grid item xs={12}>
        {children}
      </Grid>
    </Grid>
  );
};
```

Important lines are:

```tsx
    const { user } = useAuth0();
    //...
   <Typography variant="subtitle1" align="center">
          {user?.email || "EPITA SIGL 2021"}
    </Typography>
    <Typography variant="subtitle2" align="center" color="primary">
        <Logout />
    </Typography>
    //...
```

This will display the user email you've logged in with and display a `Logout` button that should redirect you to the main page.

## Step 3: Secure your API

In the previous workshop, you've created a web API exposing help requests.

Now it is time to secure it.

It means a user needs to get authenticated to call your web API.

### Create your API in Auth0

Create a new API in Auth0:

- go to your Dashboard > API > Create new API
- Set the following fields (adapt for your group name):
  - Name: arla-group-XX-api
  - Identifier: https://api.groupeXX.arla-sigl.fr
  - Leave default signing algorithm: RS256

From your `api/` code, adapt with the code Auth0 provides in its QuickStart > NodeJS section:

- install dependencies

```sh
# from api/
nvm use
npm i --save express-jwt jwks-rsa
npm i --save-dev @types/express-jwt @types/jwks-rsa
```

```ts
// src/server.ts
//...
import jwt from "express-jwt";
import jwks from "jwks-rsa";
//...

const jwtCheck: express.RequestHandler = jwt({
  secret: jwks.expressJwtSecret({
    cache: true,
    rateLimit: true,
    jwksRequestsPerMinute: 5,
    jwksUri: // CHECK QUICKSTART FOR VALUES
  }),
  audience: /* CHECK QUICKSTART FOR VALUES */,
  issuer: /* CHECK QUICKSTART FOR VALUES */,
  algorithms: ["RS256"],
});

// ...

app.get(
  "/v1/help-request",
  jwtCheck,
  (request: express.Request, response: express.Response): void => {
    // ...
  }
);
```

That's it! You've secured your API. `jwtCheck` is a middleware that will check if the `Bearer` token is valid.

You can try out to curl your API again, you should get an `Unauthorized` message:

```sh
curl http://localhost:3000/v1/help-request?page=1&limit=10
# You should get an HTML containing:
# UnauthorizedError: No authorization token was found
```

And if you wanna verify if giving a fake token would work, you can try (and make sure it fails!):

```sh
curl -H "Authorization: Bearer AFakeT0kenThatMeansNothingButIAmStillTrying" http://localhost:3000/v1/help-request?page=1&limite=10
# You should get an HTML containing:
# UnauthorizedError: jwt malformed
```

The only way to get a correct token is if you are authenticated thru the Login page of frontend.

### Adapt your frontend component in the frontend

Set the `audience` property of your `<Auth0Provider />` to your `API Identifier` from your newly created API in Auth0:

```tsx
// inside src/app.tsx
// ...
<Auth0Provider
    // ...
    audience={/* Your API Identifier */}
    cacheLocation="localstorage"
  >
```

This will make sure the token you get after authentication has rights to access your API.

## Step 4: Call your Secured API

Sofar, you've been using `curl` to consume your web API.

In this section, you will consume your web API from your frontend code.

What you need to do:

- create your api call using [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)
- call your API from a component

### Create your api call

Because you will use it at several places, you will create a seperate function to call any API route:

```ts
// From a new file inside frontend
// e.g. ./src/utils/api.ts
const API_ENDPOINT = process.env.API_ENDPOINT || "http://localhost:3000";

export const callApi = (token?: string) => async (route: string) => {
  if (token) {
    const response = await fetch(`${API_ENDPOINT}${route}`, {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    });
    return await response.json();
  } else throw new Error("no auth token");
};
```

You are wrapping the `fetch` usage to make sure you're not forgetting the `Bearer` token.
This is a curry function that you can call like

```ts
callApi("<access_token>")("/v1/help-requests?page=1&limit=10");
```

You may have noticed `async` and `await` keywords. It's syntaxic sugar to replace many chain callbacks using the classic promise's `then` and `catch` keyword.

Here is the version, totally identical, using `Promises`:

```ts
// ...
const API_ENDPOINT = process.env.API_ENDPOINT || "http://localhost:3000";

export const callApi = (token?: string) => (route: string): Promise<any> => {
  if (token) {
    fetch(`${API_ENDPOINT}${route}`, {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    }).then((response) => response.json());
  } else throw new Error("no auth token");
};
```

### Call your API from a component

To try out your call, let's create a new `Button` which will consume the web API when the user clicks on it.

Let's create this button on the `TemplateView2` component, instead of the dummy `+1` count button:

```tsx
// from frontend
// e.g. src/components/TemplateView2.tsx
export const TemplateView2: React.FC = () => {
  const { getAccessTokenSilently } = useAuth0();

  const [helpRequests, setHelpRequests] = React.useState<any[]>([]);
  const [error, setError] = React.useState<any>();

  const getHelpRequests = async () => {
    const token = await getAccessTokenSilently();
    try {
      const data = await callApi(token)("/v1/help-request?page=1&limit=10");
      setHelpRequests(data);
    } catch (error) {
      // something went wrong!
      setError(error);
    }
  };

  return (
    <Grid container spacing={5}>
      <Grid item xs={12}>
        <Button
          variant="contained"
          color="primary"
          onClick={() => {
            getHelpRequests();
          }}
        >
          Load Requests
        </Button>
      </Grid>
      <Grid item xs={12}>
        {error ? (
          <ClosableError onClickClose={() => setError(undefined)}>
            Something went wrong on our side...
          </ClosableError>
        ) : (
          <HelpRequestTable helpRequests={helpRequests} />
        )}
      </Grid>
    </Grid>
  );
};
```

Where `HelpRequestTable` component is:

```tsx
const HelpRequestTable: React.FC<HelpRequestTableProps> = ({
  helpRequests,
}) => {
  return (
    <TableContainer component={Paper}>
      <Table>
        <TableHead>
          <TableRow>
            <TableCell>Help request ID</TableCell>
            <TableCell align="right">Location</TableCell>
            <TableCell align="right">Description</TableCell>
          </TableRow>
        </TableHead>
        <TableBody>
          {helpRequests.map((hr, index) => (
            <TableRow key={index}>
              <TableCell component="th" scope="row">
                {hr.id}
              </TableCell>
              <TableCell align="right">{hr.location}</TableCell>
              <TableCell align="right">{hr.description}</TableCell>
            </TableRow>
          ))}
        </TableBody>
      </Table>
    </TableContainer>
  );
};
```

And `ClosableError` component is:

```tsx
const ClosableError: React.FC<ClosableErrorProps> = ({
  onClickClose,
  children,
}) => (
  <Alert
    severity="error"
    action={
      <IconButton
        aria-label="close"
        color="inherit"
        size="small"
        onClick={() => onClickClose()}
      >
        <CloseIcon fontSize="inherit" />
      </IconButton>
    }
  >
    {children}
  </Alert>
);
```

Once setted, navigate to view 2 and click on the `Load requests` button:
![load-help-requests](img/load-requests.gif)

You can inspect your page and check `Network` tab. You should see your API call (as a XHR request) when clicking on the button. Inside the header of the request, you should see the access token in the request's header `Authorization: Bearer <YOUR ACCESS TOKEN>`:
![inspect-bearer](img/bearer-in-request.gif)

> Note: by copying this Access token from the http call, you can try out again with curl, and see that it works:
> `curl -H "Authorization: Bearer <YOUR ACCESS TOKEN>" http://localhost:3000/v1/help-request\?page\=1\&limit\=10 `

Now, you may have notice that the your new `TemplateView2` component is also handling errors.

Let's `throw` an error inside the `try` clause of `getHelpRequests` function, to check how the error handling behaves:

```tsx
// from frontend
// e.g. src/components/TemplateView2.tsx
export const TemplateView2: React.FC = () => {
  // ...
  const getHelpRequests = async () => {
    const token = await getAccessTokenSilently();
    try {
      throw new Error("BOOM!");
      //...
    } catch (error) {
      // something went wrong!
      setError(error);
    }
  };

  return; // ...
};
```

The `throw new Error("BOOM!");` should now change the behaviour when user clicks on `Load Requests` button:
![error-handling](img/error-handling.gif)

## Step 5: Adapt your build for production

You may have noticed this code snippet in the Step 4:

```ts
process.env.API_ENDPOINT || "http://locahost:3000";
```

You have different addresses for your API, depending on which environment your on:

- your machine (a.k.a local host): http://localhost:3000
- production host (ScaleWay machine): https://api.groupeXX.arla-sigl.fr

How to adapt your build to have the correct endpoint depending on which environment you're on?

You need to use environment variable.

to have the correct `API_ENDPOINT`, adapt your webpack config, by adding those changes:

```js
// inside frontend
// ./webpack.config.js
const path = require("path");
const webpack = require("webpack");
// ...

module.exports = (env) => ({
  //...,
  plugins: [
    //...
    new webpack.DefinePlugin({
      "process.env.API_ENDPOINT": JSON.stringify(
        env?.API_ENDPOINT || "http://localhost:3000"
      ),
    }),
  ],
});
```

You are using a webpack plugin to inject an environment variable **when webpack will build your project**.

Because your frontend code is **static** and cannot change once compile.

Just adapt your `build` script in your `package.json` to set the correct variable:

```json
// inside frontend/
// ./package.json
{
  "name": "arlaide-frontend",
  // ...
  "scripts": {
    "build": "webpack --mode production --env.API_ENDPOINT=https://api.groupeXX.arla-sigl.fr",
    "start": "webpack-dev-server --open"
  }
  //...
}
```

Then, when you will use:

- `npm start`: `API_ENDPOINT` will not be define and be default to `http://localhost:3000`
- `npm run build`: `API_ENDPOINT` will be set to `https://api.groupeXX.arla-sigl.fr`

You can try to push and see if you can reach your api once deploy.

## Step 6: Use permissions to have 2 different profiles of users

The aim is to use the `Auth0` permissions to restrict access to some service to some users.

This kind of segregation of users is very common in software. Think about any product following the Freemium business model, where you have free users and premium users. Premium users having access to more functionnalities and content that free users.

Here is what we want to achieve:

- Login with one premium user, and have access to see other user's profile
- Login with one non-premium user, and instead of having access to other user's profile, she/he sees an offer to become a premium member.

### Setup permission in Auth0

First step is to define `permissions` in your Auth0's dashboard.
From the API settings you've created:

- navigate to the `permission` tab, and create a new `profiles:full-access` permission, and as description `Allow user to see other user profiles`.
- navigate to the `settings` tab, and enable RBAC and Access settings as follow:

![rbac-settings](img/rbac-api-settings.png)

### Create a new service to return user permissions

In order to display content based on user permissions in the frontend, we need to know what permissions a user has.

To do so, you will create a new service in your web API `/v1/permissions` that returns the user permissions:

```ts
// from your api/
// ./src/server.ts

app.get(
  "/v1/permissions",
  jwtCheck,
  (request: express.Request, response: express.Response) => {
    try {
      const permissions: string[] = (request as any)?.user?.permissions || [];
      response.send(permissions);
    } catch (error) {
      console.log(error);
      response.statusCode = 500;
      response.send({ error });
    }
  }
);
```

This service will return the user permissions. In fact, everything is done by the `jwtCheck` middleware. It decodes the JWT and set the `permission` field in the `request` parameter.
Our service just have to return those permissions, or an error if they can't be found (could be cause by a misconfiguration of your API in Auth0)

### Keep user permissions in your frontend context

Now that you have created the web API, let's consume it from the frontend.

We want the user permissions inside the machine context, sothat they can be use by every components without having to pass it by `props` everytime.

This will give you flexibity to renders some content, depending on some permissions the user has or hasn't.

To do so, you need:

1. a new `permissions` value in your machine context
1. a new `Event` triggered when you will have your permissions from the web api
1. a new `Action` that will assign permissions in your machine context.
1. A new component to call the permisson API you've created and send permissons to the machine context.

1. Add `permissons` to your machine context:

```ts
// from frontend
// ./src/state/machine.tsx
//...
export type TemplateContext = {
  //...
  permissions?: string[];
};
//...
```

2. Create a new `Event` called `SetUserPermissionsEvent`

```ts
export enum TemplateEvents {
  toView1 = "TO_VIEW_1",
  //...
  setUserPermissions = "SET_USER_PERMISSIONS",
}

//...

export type SetUserPermissions = {
  type: TemplateEvents.setUserPermissions;
  permissions: string[];
};

//...

export const createTemplateStateMachine = () => {
  return Machine<TemplateContext>(
    {
      id: "template",
      //...
      on: {
        [TemplateEvents.toView1]: TemplateStates.view1,
        //...
        [TemplateEvents.setUserPermissions]: {
          actions: [TemplateActions.updatePermissions],
        },
      },
    },
    { actions: allActions }
  );
```

3. Create a new `action` called `updateUserPermissions` with its assign function:

```ts
//...
export enum TemplateActions {
  //...
  updatePermissions = "updatePermissions",
}

//...

const assignPermissions = assign<TemplateContext, SetUserPermissions>({
  permissions: (_, event) => event.permissions,
});

// Set all your actions there, and they will be added to your machine
export const allActions: any = {
  //...
  [TemplateActions.updatePermissions]: assignPermissions,
};
```

4. Let's modify the `Authentication.tsx` file of your frontend a new component that will save the user permissions:

```tsx
// inside frontend
// ./src/components/Authenticated.tsx
const WithPermissions: React.FC = ({ children }) => {
  const { machine, send } = React.useContext(TemplateMachineContext);
  const { permissions } = machine.context;
  const { getAccessTokenSilently } = useAuth0();

  React.useEffect(() => {
    const getClaims = async () => {
      try {
        const authToken = await getAccessTokenSilently();
        const userPermissions = await callApi(authToken)(`/v1/permissions`);
        send({
          type: TemplateEvents.setUserPermissions,
          permissions: userPermissions,
        });
      } catch (error) {
        console.log("Error getting user permissions: ", error);
      }
    };
    getClaims();
  }, []);

  return permissions === undefined ? <Loading /> : <>{children}</>;
};

export const Authenticated: React.FC = ({ children }) => {
  //...

  return isLoading ? (
    <Loading />
  ) : (
    <WithPermissions>{children}</WithPermissions>
  );
};
```

### Create a first component for premium users only

Now, let's write one component that will display `You're premium!` only if your user has `profile:full-access` permission, or will display `GET PREMIUM ACCESS TODAY` button otherwise.

First, let's create a third view.

1. Adapt your machine:

```tsx
// from frontend
// ./src/state/machine.tsx
//...
export enum TemplateStates {
  view1 = "view1",
  view2 = "view2",
  view3 = "view3",
  //...
}

export enum TemplateEvents {
  toView1 = "TO_VIEW_1",
  toView2 = "TO_VIEW_2",
  toView3 = "TO_VIEW_3",
  //...
}

export const createTemplateStateMachine = () => {
  return Machine<TemplateContext>(
    {
      id: "template",
      initial: TemplateStates.view1,
      //...
      states: {
        [TemplateStates.view1]: {},
        [TemplateStates.view2]: {},
        [TemplateStates.view3]: {},
        //...
      },
      on: {
        [TemplateEvents.toView1]: TemplateStates.view1,
        [TemplateEvents.toView2]: TemplateStates.view2,
        [TemplateEvents.toView3]: TemplateStates.view3,
        //...
      },
    },
    { actions: allActions }
  );
};
```

2. Update your `TemplateNavigation` component, by adding the view3 and adapt the selection logic:

```tsx
//...

export const TemplateNavigation = () => {
  //...

  let tabSelected = 0;
  if (machine.matches(TemplateStates.view2)) tabSelected = 1;
  else if (machine.matches(TemplateStates.view3)) tabSelected = 2;

  return (
    <Grid container spacing={5}>
      <Grid item xs={12}>
        <Tabs
          centered
          value={tabSelected}
          onChange={(_, idx) => {
            let nextView = TemplateEvents.toView1;
            if (idx === 1) nextView = TemplateEvents.toView2;
            else if (idx === 2) nextView = TemplateEvents.toView3;
            send(nextView);
          }}
          indicatorColor="primary"
          textColor="primary"
          aria-label="disabled tabs example"
        >
          <Tab label="View 1" />
          <Tab label="View 2" />
          <Tab label="View 3" />
        </Tabs>
      </Grid>
    </Grid>
  );
};
```

3. Create a new `TemplateView3` component with the conditionnal rendering on permissions:

```tsx
import Button from "@material-ui/core/Button";
import React from "react";
import { TemplateMachineContext } from "../state/provider";

const GoPremium: React.FC = () => {
  return (
    <Button variant="outlined" color="primary" onClick={() => {}}>
      Get premium access today!
    </Button>
  );
};

export const TemplateView3: React.FC = () => {
  const { machine } = React.useContext(TemplateMachineContext);
  const { permissions } = machine.context;
  const canReadProfile = permissions?.includes("profiles:full-access");

  return canReadProfile ? <span>You're premium!</span> : <GoPremium />;
};
```

You can see that TemplateView3 components reads user permissions from the state machine context, without having to pass it to the component's properties. Thanks to those two lines

```ts
const { machine } = React.useContext(TemplateMachineContext);
const { permissions } = machine.context;
```

4. Add the `TemplateView3` to the `app.tsx`, by replacing the `TemplateContent` component with:

```tsx
const TemplateContent = () => {
  const { machine } = React.useContext(TemplateMachineContext);
  let Content = <Typography>Nothing to display</Typography>;
  if (machine.matches(TemplateStates.view1))
    Content = <TemplateView1 project="Arlaide" />;
  else if (machine.matches(TemplateStates.view2)) Content = <TemplateView2 />;
  else if (machine.matches(TemplateStates.view3)) Content = <TemplateView3 />;

  return Content;
};
```

You should be all set!

Let's try out your changes.

From the Auth0 dashboard > Users & Role > User, create 2 users using the `+ CREATE USER` button:

- name one premium user (e.g. premium@test.com )
  - assign this user `profile:full-access` permissions from the created API in previous step.
- name one non-premium user (e.g. not-premium@test.com)
  - don't assign this user any permissions

Now, let's login with the premium user to your app, you should see in the `view 3` the `You're premium!` message.

Logout, and login with the non-premium user. You should see in the `view 3` the button to become a premium member.

Congrats!

### Create a new service to get other user's profile, restricted to premium users

Now, let's create a new service in your web API that is only available for premium users.

From your API code, you will need to:

- create a new express middleware that checks for permissions
- create a new service using this middleware

For this workshop, you will create a new service that returns other user's profile.

From your api, let's create a new express middleware function that checks user permissions:

```ts
/**
 * Checks if user has all permissions in parameter. If not all permissions are matched,
 * user is unauthorized.s
 * @param permissions permissions users needs to have to be authorized
 */
const permissionContains = (permissions: string[]) => (
  request: express.Request,
  response: express.Response,
  next: express.NextFunction
) => {
  const userPermissions: string[] = (request as any)?.user?.permissions;
  const allIncluded = permissions.reduce(
    (includeAll, p) => includeAll && userPermissions.includes(p),
    true
  );

  if (allIncluded) {
    next();
  } else {
    response.statusCode = 403;
    response.send({ error: "not enough permissions" });
  }
};
```

The `permissionContains` function you've just created is an express middleware.

You can use it like `jwtCheck`, by calling `permissionContains(['your:permission'])`.

Now let's create a new user profile service using the middleware you've just created.

This service will return a page of other user's profiles, still using our `FakeDB` singleton.

First, create your fake data:

```ts
// inside api/
// ./src/fake-data/user-profiles.ts
export type UserProfile = {
  id: string;
  username: string;
  job: string;
};

export const allUserProfiles: UserProfile[] = [
  {
    id: "U-1",
    username: "florent.fauchille",
    job: "developer",
  },
  {
    id: "U-2",
    username: "lucas.boisserie",
    job: "devops",
  },
];
```

Create the method in `FakeDB` to get a page of user profile, by replacing the whole content by:

```ts
import { allHelpRequests, HelpRequest } from "./fake-data/help-requests";
import { allUserProfiles, UserProfile } from "./fake-data/user-profiles";

/**
 * Fake database, since we don't want to set any real database yet.
 * This will be subject of next workshop
 */
export namespace FakeDB {
  // Private base help requests. This is restore
  // on each start of the API.
  var helpRequests: HelpRequest[] = allHelpRequests;
  var userProfiles: UserProfile[] = allUserProfiles;

  const getPage = <T>(arr: T[]) => (page: number, limit: number) => {
    const start = (page - 1) * limit;
    const end = start + limit;

    return arr.slice(start, end);
  };

  // takes a slice of the help requests, based on page and limit offsets
  export const getHelpRequest = (page: number, limit: number) => {
    return getPage<HelpRequest>(helpRequests)(page, limit);
  };

  export const getUserProfiles = (page: number, limit: number) => {
    return getPage<UserProfile>(userProfiles)(page, limit);
  };
}
```

> Note: we did a small refactoring to void having 2 times same way of slicing an array of data from page and limit parameters

Now, let's create the user profile service, from your `server.ts`:

```ts
// from api/
// ./src/server.ts

//...

const isPremium: express.RequestHandler = permissionContains([
  profilesFullAccess,
]);

app.get(
  "/v1/user-profile",
  jwtCheck,
  isPremium,
  (request: express.Request, response: express.Response): void => {
    try {
      const { page, limit } = extractPageOptions(request);
      const userProfiles: UserProfile[] = FakeDB.getUserProfiles(page, limit);
      response.send(userProfiles);
    } catch (e) {
      response.statusCode = 500;
      response.send({ error: e.message });
    }
  }
);
// ...
```

Now you're all set! You've just created a new service, restricted to premium users.

### Integrate other user's profile service to the frontend

Now, let's integrate this service to the frontend.

What you will do:

1. call the service to get other user profiles, when a user is premium
1. create a small component to display user profiles

Let's modify the `TemplateView3` component, the whole file should look like:

```ts
import { useAuth0 } from "@auth0/auth0-react";
import { Grid } from "@material-ui/core";
import Button from "@material-ui/core/Button";
import Card from "@material-ui/core/Card";
import CardActionArea from "@material-ui/core/CardActionArea";
import CardActions from "@material-ui/core/CardActions";
import CardContent from "@material-ui/core/CardContent";
import CardMedia from "@material-ui/core/CardMedia";
import { makeStyles } from "@material-ui/core/styles";
import Typography from "@material-ui/core/Typography";
import React from "react";
import { TemplateMachineContext } from "../state/provider";
import { callApi } from "../utils/api";

const useStyles = makeStyles({
  root: {
    maxWidth: 345,
  },
  media: {
    height: 140,
  },
});

const GoPremium: React.FC = () => {
  return (
    <Button variant="outlined" color="primary" onClick={() => {}}>
      Get premium access today!
    </Button>
  );
};

type ProfileCardProps = {
  username: string;
  job: string;
};

const ProfileCard: React.FC<ProfileCardProps> = ({ username, job }) => {
  const classes = useStyles();

  return (
    <Card className={classes.root}>
      <CardActionArea>
        <CardMedia
          className={classes.media}
          image="https://upload.wikimedia.org/wikipedia/commons/e/e4/Elliot_Grieveson.png"
        />
        <CardContent>
          <Typography gutterBottom variant="h5" component="h2">
            {username}
          </Typography>
          <Typography variant="body2" color="textSecondary" component="p">
            {job}
          </Typography>
        </CardContent>
      </CardActionArea>
      <CardActions>
        <Button size="small" color="primary">
          Contact
        </Button>
        <Button size="small" color="secondary">
          See activities
        </Button>
      </CardActions>
    </Card>
  );
};

const UserProfiles: React.FC = () => {
  const [userProfiles, setUserProfiles] = React.useState<any[]>([]);
  const { getAccessTokenSilently } = useAuth0();

  const getUserProfiles = async () => {
    const authToken = await getAccessTokenSilently();
    const data = await callApi(authToken)("/v1/user-profile?page=1&limit=10");
    setUserProfiles(data);
  };

  return (
    <Grid container spacing={6}>
      <Grid item xs={12}>
        <Button
          variant="contained"
          color="primary"
          onClick={() => getUserProfiles()}
        >
          Load user profiles
        </Button>
      </Grid>
      {(userProfiles as any[]).map((userProfile, index) => (
        <Grid key={index} item xs={4}>
          <ProfileCard username={userProfile.username} job={userProfile.job} />
        </Grid>
      ))}
    </Grid>
  );
};

export const TemplateView3: React.FC = () => {
  const { machine } = React.useContext(TemplateMachineContext);
  const { permissions } = machine.context;
  const canReadProfile = permissions?.includes("profiles:full-access");

  return canReadProfile ? <UserProfiles /> : <GoPremium />;
};
```

Now try out yourself with your premium user, and you should see the other user profiles. Congratulation!

![premium-other-user-profiles](img/premium-other-user-profile.gif)


