---
{"dg-publish":true,"dg-permalink":"why-and-how-migrate-from-firebase-to-serverless-stack","permalink":"/why-and-how-migrate-from-firebase-to-serverless-stack/"}
---




Search

![Why and How Migrate From Firebase to Serverless Stack?](https://images.unsplash.com/photo-1633356122102-3fe601e05bd2?ixlib=rb-1.2.1&q=85&fm=jpg&crop=entropy&cs=srgb)

# Why and How Migrate From Firebase to Serverless Stack?

serverless

sst

cdk

aws

Mar 19, 2022

_This article is the third of a series around SST - Serverless Stack. I will try to let you discover some amazing aspects of this particular solution in the serverless world. You can find the first article_ _[here (introduction)](https://sidoine.org/sst-the-most-underrated-serverless-framework-you-need-to-discover)_ _and the second one_ _[here (some constructs presentation)](https://sidoine.org/sst-the-most-underrated-serverless-framework-you-need-to-discover-part-2)__._

[Firebase](https://firebase.google.com/) is a fantastic tool. It allows you to build mobile or web applications without having to manage a backend by yourself. But somehow, this comes with some drawbacks. In this article I will explain you why you may want to switch, and a practical guide to switch.

In [a concrete example](https://nelligan.sidoine.org/) I will migrate a [React](https://reactjs.org/) application that is relying on both Firebase and a [Serverless Framework](https://www.serverless.com/) backend to a single stack (with [Serverless Stack](https://serverless-stack.com/))

### 

[](https://sidoine.org/why-and-how-migrate-from-firebase-to-serverless-stack#6c87bd3209584776b8a6ed73cfa516e5 "Short Presentation of Each Solutions")Short Presentation of Each Solutions

-   **[Firebase](https://firebase.google.com/products-build)** is a product backed by Google. It allow you to create mobile and web applications based on a set of Firebase components. It contains an **authentication** layer, a **database** (FireStore), a **storage** component to save files, and a **hosting** solution to ship your application. It’s also possible to rely on **Cloud Function** to run code in **backend functions**.

-   **[Serverless Framework](https://www.serverless.com/)** is a solution to host your backend components in a dedicated cloud provider without having to manage servers. For exemple on AWS it will allow your to manage Lambda functions easily.

-   [**Serverless Stack**](https://serverless-stack.com/) is a new solution that can do what Serverless Framework offer. But it offer also to handle the hosting of your web application, and provide a better developer experience in my opinion. I have already written a couple of article on the subject: [here for an introduction](https://sidoine.org/sst-the-most-underrated-serverless-framework-you-need-to-discover) and [here for some constructs presentation](https://sidoine.org/sst-the-most-underrated-serverless-framework-you-need-to-discover-part-2).

-   **[React](https://reactjs.org/)** is a Javascript library to build user interface 😇

### 

[](https://sidoine.org/why-and-how-migrate-from-firebase-to-serverless-stack#d533b10caa8848cbad4eb9c9b96eb3fc "Why You May Want to Migrate?")Why You May Want to Migrate?

I was running my system to manage [Montreal library cards](https://sidoine.org/the-web-is-an-api-scrap-it) since a few year based on **Firebase**. Because I was using the [free version of Firebase](https://firebase.google.com/pricing), I wasn’t able to use **Cloud Functions**. But to query Montreal library system, it was needed to run some functions somewhere. Back in the days, I have selected **Serverless Framework** to operate this backend API on my own AWS account. But it was not ideal, because I was dealing with too much stacks. Focusing on Firebase, here is a list of items that can limit you:

-   **Firebase is offering a limited set of functionalities**: the integrated solutions is providing a really nice set of features for common web application (authentication, storage, database...). But it’s not easily extensible. When you use directly AWS, you can use any service provided by the cloud provider. Think about _Machine Learning_ service, _Queue_ systems, _Container_ workload...

-   **Pricing model is not cheap**: when you leave the [no-cost plan](https://firebase.google.com/pricing) (Spark), Firebase can be quite expensive, depending on your usage. For reference this classic article [30k bill on Firebase](https://www.dottedsquirrel.com/30k-firebase/) is a good reference! The _backend-as-a-service_ model can lead to such issues if not well optimized. AWS is not cheap either, but you will pay only what you are using and you have more options to build your product (does the frontend is running query on the database directly or via a backend API?)

-   **Developer experience can be limited**: local development is a must for serverless application : it reduces the feedback time it take you to test each feature. Firebase offer you a [local emulator suite](https://firebase.google.com/docs/emulator-suite) to provide you a local environment. It will allow you to test quickly the cloud function built, without waiting them to be shipped. But it’s only an emulation, not real cloud function running on your cloud provider. On the opposite, Serverless Stack is providing you a [live lambda development](https://docs.serverless-stack.com/live-lambda-development) environment that is relying on AWS services, not emulation.

### 

[](https://sidoine.org/why-and-how-migrate-from-firebase-to-serverless-stack#218c83b9852e4acdb6e0373a52303680 "Running the Migration in 6 Steps!")Running the Migration in 6 Steps!

#### 

[](https://sidoine.org/why-and-how-migrate-from-firebase-to-serverless-stack#0e413e04d6c04e54adf81e0c38141d89 "Step 1: Init your Serverless Stack application")Step 1: Init your Serverless Stack application

Following the [quick-start](https://docs.serverless-stack.com/#quick-start):

```bash
# Create a new SST app
npx create-serverless-stack@latest my-sst-app
cd my-sst-app
```

Take some time to explore the organisation of the folder. `stacks/` contains your infrastructure setup, `src/` will contains your Lambda function code.

#### 

[](https://sidoine.org/why-and-how-migrate-from-firebase-to-serverless-stack#e4749ab6a43a41ccab5440814973a5c4 "Step 2: Migrate from Serverless Framework to the new application")Step 2: Migrate from Serverless Framework to the new application

In my specific case, I was migrating functions from Serverless Framework. The guys from SST have a decent documentation for this classic case: [****Migrating From Serverless Framework****](https://docs.serverless-stack.com/migrating/serverless-framework)****.****

Basically I have reused directly the javascript files from the old project, and place them in the `src/` folder of the new project. Then inside `stacks/MyStack.ts`, I have [created my API routes](https://github.com/julbrs/nelligan-plus/blob/7fcff53b8be57a2505ccbbe1556576c46c02df98/stacks/MyStack.ts):

```typescript
// Create a HTTP API
const api = new sst.Api(this, "Api", {
  defaultAuthorizationType: sst.ApiAuthorizationType.AWS_IAM,
  cors: true,
  routes: {
    "GET /cards": "src/cards.list",
    "POST /cards": "src/cards.add",
    "DELETE /cards/{id}": "src/cards.remove",
    "GET /cards/{id}/books": "src/books.list",
		...
  },
});
```

The `defaultAuthorizationType` allow me to secure the API with an IAM authentication (see next step!).

#### 

[](https://sidoine.org/why-and-how-migrate-from-firebase-to-serverless-stack#4aef4289463f48c4b3d8ef7d23d3258a "Step 3: Replace the Firebase Authentication")Step 3: Replace the Firebase Authentication

Firebase is handy because it comes with an authentication layer built-in. Inside SST the best option is to use the `Auth` [construct](https://docs.serverless-stack.com/constructs/Auth), that is relying behind the scene on AWS Cognito.

In `stacks/MyStack.ts`, I am adding:

```typescript
// Create auth
const auth = new Auth(this, "Auth", {
  cognito: {
    userPoolClient: {
      supportedIdentityProviders: [UserPoolClientIdentityProvider.GOOGLE],
      oAuth: {
        callbackUrls: [
          scope.stage === "prod"
            ? `https://${prodDomainName}`
            : "http://localhost:3000",
        ],
        logoutUrls: [
          scope.stage === "prod"
            ? `https://${prodDomainName}`
            : "http://localhost:3000",
        ],
      },
    },
  },
});

if (
  !auth.cognitoUserPool ||
  !auth.cognitoUserPoolClient ||
  !process.env.GOOGLE_AUTH_CLIENT_ID ||
  !process.env.GOOGLE_AUTH_CLIENT_SECRET
) {
  throw new Error(
    "Please set GOOGLE_AUTH_CLIENT_ID and GOOGLE_AUTH_CLIENT_SECRET"
  );
}

const provider = new UserPoolIdentityProviderGoogle(this, "Google", {
  clientId: process.env.GOOGLE_AUTH_CLIENT_ID,
  clientSecret: process.env.GOOGLE_AUTH_CLIENT_SECRET,
  userPool: auth.cognitoUserPool,
  scopes: ["profile", "email", "openid"],
  attributeMapping: {
    email: ProviderAttribute.GOOGLE_EMAIL,
    givenName: ProviderAttribute.GOOGLE_GIVEN_NAME,
    familyName: ProviderAttribute.GOOGLE_FAMILY_NAME,
    phoneNumber: ProviderAttribute.GOOGLE_PHONE_NUMBERS,
  },
});

// make sure to create provider before client (https://github.com/aws/aws-cdk/issues/15692#issuecomment-884495678)
auth.cognitoUserPoolClient.node.addDependency(provider);

const domain = auth.cognitoUserPool.addDomain("AuthDomain", {
  cognitoDomain: {
    domainPrefix: `${scope.stage}-nelligan-plus`,
  },
});

// Allow authenticated users invoke API
auth.attachPermissionsForAuthUsers([api]);
```

This will allow me the use Google as my principal authentification system (inside **Cognito User Pool**). There is an alternate way to use Cognito Identity Pool with a simpler declaration:

```typescript
new Auth(this, "Auth", {
  google: {
    clientId:
      "xxx.apps.googleusercontent.com",
  },
});
```

But it’s harder to manage in the React app so I prefer my initial version 😇.

#### 

[](https://sidoine.org/why-and-how-migrate-from-firebase-to-serverless-stack#b74de36e26b2417db11f0a20f9efc3b4 "Step 4: Replace the Firestore Database")Step 4: Replace the Firestore Database

The Firebase project rely on Firestore to store some data related to each user. On the new stack you must build a new system to store data. The equivalent structure in AWS world is a **DynamoDB** table, with a cost per usage. It fits well serverless deployments. There is useful `Table` [construct](https://docs.serverless-stack.com/constructs/Table) available in SST:

```typescript
// Table to store cards
  const table = new Table(this, "Cards", {
    fields: {
      cardId: TableFieldType.STRING,
      cardUser: TableFieldType.STRING,
      cardCode: TableFieldType.STRING,
      cardPin: TableFieldType.STRING,
    },
    primaryIndex: { partitionKey: "cardId" },
  });
```

#### 

[](https://sidoine.org/why-and-how-migrate-from-firebase-to-serverless-stack#a47a6ed771df4315a139ae62d3b814ff "Step 5: Replace the Firebase Hosting")Step 5: Replace the Firebase Hosting

Here there is multiple approach possible. I am suggesting the most integrated solution for an SST stack:

-   use the new [ReactStaticSite construct](https://docs.serverless-stack.com/constructs/ReactStaticSite)

-   take advantage of the [static-site-env](https://docs.serverless-stack.com/packages/static-site-env) to handle automatically the environment variables

First add in `MyStack.ts`:

```typescript
// Create frontend app
const reactApp = new ReactStaticSite(this, "ReactSite", {
  path: "react-app",
  buildCommand: "yarn && yarn build",
  environment: {
    REACT_APP_REGION: this.region,
    REACT_APP_API_URL: api.url,

    REACT_APP_GA_TRACKING_ID: "UA-151729273-1",
    REACT_APP_USER_POOL_ID: auth.cognitoUserPool.userPoolId,
    REACT_APP_USER_POOL_CLIENT_ID:
      auth.cognitoUserPoolClient.userPoolClientId,
    REACT_APP_IDENTITY_POOL_ID: auth.cognitoIdentityPoolId,
    REACT_APP_USER_UI_DOMAIN: domain.domainName,
    REACT_APP_DOMAIN:
      scope.stage === "prod"
        ? `https://${prodDomainName}`
        : "http://localhost:3000",
  },
  customDomain:
    scope.stage === "prod"
      ? {
          domainName: prodDomainName,
          hostedZone: "sidoine.org",
        }
      : undefined,
});
```

The `environment` props allow to pass environment variables to the React stack. The `path` is the relative path that contains your React app.

#### 

[](https://sidoine.org/why-and-how-migrate-from-firebase-to-serverless-stack#7958856643ac4425b6ecf41efbdaf1db "Step 6: Adapt your React Application")Step 6: Adapt your React Application

So following step 5, in the `react-app/` folder I move my existing React application and start changing it to support my new stack content. Here is a general guidance follow:

-   Remove any occurence of `firebase` library

-   Add `aws-amplify` instead (it’s a simple wrapper for using AWS ressources like auth, api, etc...)

-   Add `@serverless-stack/static-site-env` to manage environment variable from SST

-   Configure `aws-amplify` (see example [here](https://github.com/julbrs/nelligan-plus/blob/7fcff53b8be57a2505ccbbe1556576c46c02df98/react-app/src/amplify.js), based on environment variables)

-   Replace `firebase` calls by `aws-amplify` calls (that’s probably the most long task!)

For reference here is two examples of `aws-amplify` usage:

-   The `SignIn` [component](https://github.com/julbrs/nelligan-plus/blob/7fcff53b8be57a2505ccbbe1556576c46c02df98/react-app/src/components/SignIn.js) to sign in the application (rely on `CognitoHostedUIIdentityProvider`)

-   The `Card` [component](https://github.com/julbrs/nelligan-plus/blob/7fcff53b8be57a2505ccbbe1556576c46c02df98/react-app/src/components/Card.js) that is calling an API endpoint, using the `API` object from `aws-amplify`

#### 

[](https://sidoine.org/why-and-how-migrate-from-firebase-to-serverless-stack#8a6dc3f9d28e477c8b3d9d69c73cc3d6 "Link to the Project Before and After the Migration")Link to the Project Before and After the Migration

For reference, you can dig into the project before and after the migration:

Before the migration:

[

GitHub - julbrs/nelligan-plus at sls_firebase

An application to manage multiple accounts of book library for the Montreal Nelligan system. This is a frontend application using ReactJS. It consumes: a dedicated backend to get info regarding books. (see backend section) Firebase for the auth system and card storing Runs the app in the development mode.

![GitHub - julbrs/nelligan-plus at sls_firebase](https://github.com/favicon.ico)

https://github.com/julbrs/nelligan-plus/tree/sls_firebase

![GitHub - julbrs/nelligan-plus at sls_firebase](https://opengraph.githubassets.com/b779b542c4d07b6d6532023c1bb50f9fad2cd0a12835dfa6118ce59a175a9eda/julbrs/nelligan-plus)

](https://github.com/julbrs/nelligan-plus/tree/sls_firebase)

After the migration:

[

GitHub - julbrs/nelligan-plus at 7fcff53b8be57a2505ccbbe1556576c46c02df98

An application to manage multiple accounts of book library for the Montreal Nelligan system. This is a SST application (backend functions) and a ReactJS app for the frontend. SST app: React: cd react-app yarn yarn start Runs the app in the development mode. Open http://localhost:3000 to view it in the browser.

![GitHub - julbrs/nelligan-plus at 7fcff53b8be57a2505ccbbe1556576c46c02df98](https://github.com/favicon.ico)

https://github.com/julbrs/nelligan-plus/tree/7fcff53b8be57a2505ccbbe1556576c46c02df98

![GitHub - julbrs/nelligan-plus at 7fcff53b8be57a2505ccbbe1556576c46c02df98](https://opengraph.githubassets.com/b779b542c4d07b6d6532023c1bb50f9fad2cd0a12835dfa6118ce59a175a9eda/julbrs/nelligan-plus)

](https://github.com/julbrs/nelligan-plus/tree/7fcff53b8be57a2505ccbbe1556576c46c02df98)

### 

[](https://sidoine.org/why-and-how-migrate-from-firebase-to-serverless-stack#090630e7c2ce41059d986e158b6d8149 "Conclusion")Conclusion

The switch have been a game-changer for me. And it’s not because of the cost or features, but more for the **developer experience**. Before the migration, I use to first build the backend function, test it, ship it. Then use this backend function in the frontend application after shipping the backend part. Then maybe I need to go back to the backend to adapt the contract or modify the code... You get it, it was a slow back-and-forth process, not very efficient.

Today I have a single stack:

-   First I start SST via `npx sst start`

-   Then I start my React app locally (`yarn start`)

The advantages:

-   I am working on a **development environment** without link to the production system (thanks to the [stages](https://docs.serverless-stack.com/architecture#stages))

-   I can **change my backend code** directly in the IDE, and it’s available instantly! Thanks to [Live Lambda Development](https://docs.serverless-stack.com/live-lambda-development)!

-   I don’t have to manage directly the **environment variables of my frontend stack** (no more `.env` file to update!)

-   When it’s time to **ship my project**, just a single command to push both backend and frontend! `npx sst deploy --stage prod`

👉

Get a look at [Serverless Stack (SST)](https://serverless-stack.com/), or my example project ([here on Github](https://github.com/julbrs/nelligan-plus)). It’s really worth the time to understand the main concepts, and then you will be **more efficient building full stack serverless applications!** Continue the discussion on [twitter](https://twitter.com/_julbrs/status/1505278822429700096) 😎

Copyright 2022 Julien Bras

[](https://twitter.com/_julbrs "Twitter @_julbrs")[](https://github.com/julbrs "GitHub @julbrs")[](https://www.linkedin.com/in/julienbras "LinkedIn Julien Bras")

Why and How Migrate From Firebase to Serverless Stack?