---
layout: post
title: Create a New React.js App
date: 2017-01-06 00:00:00
lang: en
ref: create-a-new-react-js-app
description: In this chapter we'll use Create React App to create a new React.js app. We'll be deploying our React app to AWS using the SST StaticSite construct. It'll also load the environment variables from our serverless app.
comments_id: create-a-new-react-js-app/68
---

We are now ready to work on our frontend. So far we've built and deployed our backend API and infrastructure. We are now going to build a web app that connects to our backend.

We are going to create a single page app using [React.js](https://facebook.github.io/react/). We'll use the [Create React App](https://github.com/facebookincubator/create-react-app) project to set everything up.

### Create a New React App

{%change%} Run the following command in your project root.

```bash
$ npx create-react-app frontend --use-npm
$ cd frontend
```

This should take a second to run, and it will create your new project in the `frontend/` directory.

### Loading SST Environment Variables

We also want to load the environment variables from our backend. To do this, we’ll be using the [@serverless-stack/static-site-env package](https://www.npmjs.com/package/@serverless-stack/static-site-env). It'll find the environment variables from our SST app and load it while starting the React development environment.

{%change%} Run the following **in the `frontend/` directory**.

```bash
$ npm install sst --save-dev
```

Now to use this package, we'll add it to our `package.json` scripts.

{%change%} Replace the `start` script in your `frontend/package.json`.

```js
"start": "react-scripts start",
```

{%change%} With.

```js
"start": "sst bind react-scripts start",
```

### Add the React App to SST

We are going to be deploying our React app to AWS. To do that we'll be using the SST [`StaticSite`]({{ site.docs_url }}/constructs/StaticSite) construct.

{%change%} Create a new file in `stacks/FrontendStack.js` and add the following.

```js
import { StaticSite, use } from "sst/constructs";
import { ApiStack } from "./ApiStack";
import { AuthStack } from "./AuthStack";
import { StorageStack } from "./StorageStack";

export function FrontendStack({ stack, app }) {
  const { api } = use(ApiStack);
  const { auth } = use(AuthStack);
  const { bucket } = use(StorageStack);

  // Define our React app
  const site = new StaticSite(stack, "ReactSite", {
    path: "frontend",
    buildOutput: "build",
    buildCommand: "npm run build",
    // Pass in our environment variables
    environment: {
      REACT_APP_API_URL: api.customDomainUrl || api.url,
      REACT_APP_REGION: app.region,
      REACT_APP_BUCKET: bucket.bucketName,
      REACT_APP_USER_POOL_ID: auth.userPoolId,
      REACT_APP_IDENTITY_POOL_ID: auth.cognitoIdentityPoolId,
      REACT_APP_USER_POOL_CLIENT_ID: auth.userPoolClientId,
    },
  });

  // Show the url in the output
  stack.addOutputs({
    SiteUrl: site.url || "http://localhost:3000",
  });
}
```

We are creating a new stack in SST. We could've used one of the existing stacks but this allows us to show how to connect stacks together.

We are doing a couple of things of note here:

1. We are pointing our `StaticSite` construct to the `frontend/` directory where our React app is.
2. We are passing in the outputs from our other stacks as [environment variables in React](https://create-react-app.dev/docs/adding-custom-environment-variables/). This means that we won't have to hard code them in our React app. You can read more about this over in our chapter on, [Setting serverless environments variables in a React app]({% link _chapters/setting-serverless-environments-variables-in-a-react-app.md %}).
3. And finally, we are outputting out the URL of our React app.

### Adding to the app

Let's add this new stack to the rest of our app.

{%change%} Replace the `stacks` function in `sst.config.ts` with.

```js
stacks(app) {
  app
    .stack(StorageStack)
    .stack(ApiStack)
    .stack(AuthStack)
    .stack(FrontendStack);
},
```

{%change%} Also, import the new stack at the top.

```js
import { FrontendStack } from "./stacks/FrontendStack";
```

### Deploy the Changes

If you switch over to your terminal, you'll notice that you are being prompted to redeploy your changes. Go ahead and hit _ENTER_.

Note that, you'll need to have `sst start` running for this to happen. If you had previously stopped it, then running `npx sst start` will deploy your changes again.

You should see that the new frontend stack has been deployed.

```bash
✓  Deployed:
   ...
   FrontendStack
   SiteUrl: http://localhost:3000
```

### Start the React App

Let’s start our React development environment.

{%change%} In the `frontend/` directory run.

```bash
$ npm start
```

This should fire up the newly created app in your browser.

![New Create React App screenshot](/assets/new-create-react-app.png)

### Change the Title

{%change%} Let's quickly change the title of our note taking app. Open up `public/index.html` and edit the `title` tag to the following:

```html
<title>Scratch - A simple note taking app</title>
```

Create React App comes pre-loaded with a pretty convenient yet minimal development environment. It includes live reloading, a testing framework, ES6 support, and much more.

Now we are ready to build our frontend! We are going to start by creating our app icon and updating the favicons.
