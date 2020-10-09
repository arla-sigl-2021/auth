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

<!-- explain callback, logout and Allowed Web Origins -->
You can put these 2 URLs:
- http://localhost:3000
- http://goupe<Number>.arla-sigl.fr

The first will be use for local developpement and the second for our final app on yours server.

## Step 2: Integrate Auht0 login to your frontend

### Install the Auth0 React SDK
Auth0 provide an easy to use sdk for react.

In first, we need to add it to our package.json:
```json
// package.json
{
    ...
    "dependencies": {
        "@auth0/auth0-react": "1.1.0",
        ...
    },
}
```

You can now import the sdk and start to configure the Àuth0Provider` component.

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
    >
        <Authenticated>
            <TemplateMachineProvider>
            ...
            </TemplateMachineProvider>
        </Authenticated>
</Auth0Provider>
```

The Auth0Provider component takes the following props:
- `domain` and `clientId`: The values of these properties correspond to the "Domain" and "Client ID" values present under the "Settings" of the single-page application that you registered with Auth0.
- `redirectUri`: The URL to where you'd like to redirect your users after they authenticate with Auth0.

`Auth0Provider` stores the authentication state of your users and the state of the SDK — whether Auth0 is ready to use or not. It also exposes helper methods to log in and log out your users, which you can access using the useAuth0() hook.

The Authenticated is here to automaticly redirecxt the user to Auth0 when it's not logged.
To do that:
- Create a file `Authenticated.tsx` in `components`
- Put the code below inside

``` typescript
import React from "react";
import { useAuth0 } from "@auth0/auth0-react";

export const Authenticated: React.FC = ({ children }) => {
    const { loginWithRedirect, user, isLoading } = useAuth0();

    React.useEffect(() => {
        const redirect = async () => {
            if (!user && !isLoading) {
                await loginWithRedirect();
            }
        }
        redirect()
    }, [isLoading])
    return isLoading ? <span>Loading ...</span> : <>{children}</>;
};
```

We used a React useEffect to automaticly redirect the user.

### Add Login to Your Application


## Step 3: Secure your API
https://auth0.com/docs/quickstart/spa/react/02-calling-an-api

## Step 4: Deploy your changes


