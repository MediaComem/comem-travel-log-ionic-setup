# COMEM+ Travel Log Ionic Setup

<a name="top"></a>

This repository contains instructions to build a skeleton application that can serve as a starting point to develop the Travel Log mobile application.
The completed skeleton app is available [here](https://github.com/MediaComem/comem-travel-log-ionic-starter).

This tutorial is used in the [COMEM+](http://www.heig-vd.ch/comem) [Mobile Applications course](https://github.com/MediaComem/comem-appmob) taught at [HEIG-VD](http://www.heig-vd.ch).

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Prerequisites](#prerequisites)
- [Features](#features)
- [Design the user interface](#design-the-user-interface)
- [Set up the application](#set-up-the-application)
  - [Create a blank Ionic app](#create-a-blank-ionic-app)
  - [Serve the application locally](#serve-the-application-locally)
- [Set up the navigation structure](#set-up-the-navigation-structure)
  - [Create the pages](#create-the-pages)
  - [Update the app to use the pages](#update-the-app-to-use-the-pages)
  - [Default tab](#default-tab)
- [Set up security](#set-up-security)
  - [Check the documentation of the API's authentication resource](#check-the-documentation-of-the-apis-authentication-resource)
  - [Create model classes](#create-model-classes)
  - [Create an authentication service](#create-an-authentication-service)
  - [Create the login screen](#create-the-login-screen)
  - [Use the authentication service to protect access to the layout page](#use-the-authentication-service-to-protect-access-to-the-layout-page)
  - [Storing the authentication credentials](#storing-the-authentication-credentials)
  - [Log out](#log-out)
  - [Configuring an HTTP interceptor](#configuring-an-http-interceptor)
- [Multi-environment & sensitive configuration](#multi-environment--sensitive-configuration)
  - [Environment files](#environment-files)
  - [Create the actual configuration file](#create-the-actual-configuration-file)
  - [Add the environment files to your `.gitignore` file](#add-the-environment-files-to-your-gitignore-file)
  - [When are the environment files used](#when-are-the-environment-files-used)
  - [Feed the configuration to Angular](#feed-the-configuration-to-angular)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Prerequisites

These instructions assume that you are using the Travel Log API based on one of the suggestions of the previous [Web-Oriented Architecture](https://github.com/MediaComem/comem-archioweb),
and that you are familiar with the [documentation of the reference API](https://devmobil-travel-log-api.herokuapp.com).

You will need to have [Node.js](https://nodejs.org) installed.
The latest LTS (Long Term Support) version is recommended (^16.0.0 at the time of writing).

<a href="#top">↑ Back to top</a>

## Features

This guide describes a proposed list of features and an example user interface based on those features.
**This is only a suggestion** ; you can support other features and should definitely make a different user interface (the proposal below is not necessarily the best approach)

The proposed app should allow users to do the following:

- Create new trips & places.
- See visited places on an interactive map.
- Browse the list of trips.

The following sections describe a proposed UI mockup of the app and steps to set up a skeleton implementation.

<a href="#top">↑ Back to top</a>

## Design the user interface

Before diving into the code, you should always take a moment to design the user interface of your app.
This doesn't have to be a final design, but it should at least be a sketch of what you want.
This helps you think in terms of the whole app and of the relationships between screens.

![UI Design](setup/ui-structure.png)

As you can see, we propose to use a tab view with 3 screens, and an additional 4th screen accessible from the trip list:

- The create trip tab.
- The places map tab.
- The trip list tab.
  - A trip details page.

Now that we (somewhat) know what we want, we can start setting up the app!

<a href="#top">↑ Back to top</a>

## Set up the application

### Create a blank Ionic app

Make sure you have [Ionic][ionic] installed, and that your computer is correctly configured to deploy on a mobile device :

```bash
$> ionic -v
6.18.1
```

> If you have an error when running the above command, this probably means that you need to install [Ionic][ionic]. To do so, execute:
>
> ```bash
> $> npm install -g ionic@latest
> ```

Go in the directory where you want your app source code to be located, then start generating your Ionic app with the following command:

> No need to create a dedicated directory for your app ; it will be created for you by the CLI tool

```bash
$> ionic start
```

The command will ask you if you want to use the app creation wizard. Answering "yes" will open your browser on a user-friendly configuration page. This is entirely optional and you can define this configuration using the CLI creation wizard also.

> Since there is no way of generating an app with the **Blank** starter from this wizard, its recommended to answer "No" and keep using the CLI instead.

Whatever you choose to do, be sure to:

- Select **Angular** as your app's underlying framework
- Select the **Blank** starter template

  > If you are using the browser creation wizard, select whichever template seems to fit best your app's mockups.

  > Even though it is not recommended to start your project using a template other than the **Blank** one, you are more than advised to generate test apps using those other starters to look at how they are developed and use what you need in your own app.

Then wait for the install to proceed... ⏳

Go into the app directory. The `ionic start` command should already have initialized a Git repository and made the first commit:

```bash
$> cd travel-log
$> git log
commit 2a3f83f14ae2a82d00cb2b2960dda1c1e0b0a432 (HEAD -> master)
Author: John Doe <john.doe@example.com>
Date:   Mon Nov 4 14:25:29 2019 +0100

    Initial commit
```

<a href="#top">↑ Back to top</a>

### Serve the application locally

To make sure everything was set up correctly, use the following command from the root of your project directory to serve the application locally in your browser:

```bash
$> ionic serve
```

You should see something like this in your browser (or a dark version, depending on your OS settings):

![Serving the blank app](setup/blank-app-serve.png)

<a href="#top">↑ Back to top</a>

## Set up the navigation structure

As defined in our UI design, we want the following 4 screens:

- The trip creation tab.
- The places map tab.
- The trip list tab.
  - The trip details screen.

Let's start by setting up the 3 tabs.
We will use Ionic's [Tabs][ionic-tabs] component.

> If your app **does not** use tabs, you should use whatever's appropriate instead. Refer to the Ionic documentation if need be.

<a href="#top">↑ Back to top</a>

### Create the pages

Each page will be an [Angular component][angular-component].
Ionic has a `generate` command that can automatically set up the files we need to create each page's component.

We will create a `Layout` page that will contain the layout for the tabs (that is where the tabs bar and the slot where each tab will be displayed) ; each tab page should therefor be a subpage of `Layout`.

First let's delete the `src/app/home` folder entirely. Then update the `app-routing.module.ts` file to empty the route object:

```ts
const routes: Routes = [
  // Nothing here !
];
```

Then, use the `ionic generate` command to recreate the `Layout` page.

```bash
$> ionic generate page Layout
```

Update the `app-routing.module.ts` file to add the routes leading to this new `Layout` page:

```ts
const routes: Routes = [
  {
    path: "",
    loadChildren: () =>
      import("./layout/layout.module").then((m) => m.LayoutPageModule),
  },
];
```

Now, we can easily create and plug the subpages:

> Use the up arrow key to retrieve the last executed command. This will prevent you typing the line three times

```bash
$> ionic generate page layout/CreateTrip
$> ionic generate page layout/PlacesMap
$> ionic generate page layout/TripList
```

> Notice how each page name is **preceeded by `layout/`**. That tells the CLI where to put the page's files.

This will generate the following files:

> For the CreateTrip page

```
src/app/layout/create-trip/create-trip-routing.module.ts
src/app/layout/create-trip/create-trip.module.ts
src/app/layout/create-trip/create-trip.page.html
src/app/layout/create-trip/create-trip.page.spec.ts
src/app/layout/create-trip/create-trip.page.ts
src/app/layout/create-trip/create-trip.page.scss
```

> For the PlacesMap page

```
src/app/layout/places-map/places-map-routing.module.ts
src/app/layout/places-map/places-map.module.ts
src/app/layout/places-map/places-map.page.html
src/app/layout/places-map/places-map.page.spec.ts
src/app/layout/places-map/places-map.page.ts
src/app/layout/places-map/places-map.page.scss
```

> For the TripList page

```
src/app/layout/trip-list/trip-list-routing.module.ts
src/app/layout/trip-list/trip-list.module.ts
src/app/layout/trip-list/trip-list.page.html
src/app/layout/trip-list/trip-list.page.spec.ts
src/app/layout/trip-list/trip-list.page.ts
src/app/layout/trip-list/trip-list.page.scss
```

For each page, we have:

- A module declaration file (`*.module.ts`).
- A routing module declaration file (`*-routing.module.ts`)
- An HTML template (`*.page.html`).
- A [Sass/SCSS][sass] stylesheet (`*.page.scss`).
- An Angular component (`*.page.ts`).
- A test suite with a default test (`*.page.spec.ts`).

Now update the HTML template for each page and add some content within the `<ion-content>` tag.
For example, in `src/app/layout/create-trip/create-trip.page.html`:

```html
<ion-content class="ion-padding"> Let's create a trip. </ion-content>
```

### Update the app to use the pages

Now that the pages are ready, we need to display them.

All your pages have been added to the `layout-routing.module.ts` file and have thus specific route that can display them.

The current `routes` array should be as follows (minus the comments):

```ts
const routes: Routes = [
  {
    // Default route
    path: "",
    component: LayoutPage,
  },
  {
    // Route that loads the CreateTrip module
    path: "create-trip",
    loadChildren: () =>
      import("./create-trip/create-trip.module").then(
        (m) => m.CreateTripPageModule
      ),
  },
  {
    // Route that loads the PlacesMap module
    path: "places-map",
    loadChildren: () =>
      import("./places-map/places-map.module").then(
        (m) => m.PlacesMapPageModule
      ),
  },
  {
    // Route that loads the TripList module
    path: "trip-list",
    loadChildren: () =>
      import("./trip-list/trip-list.module").then((m) => m.TripListPageModule),
  },
];
```

For the tabs to propertly works, you need to **change this structure** so that each tab page route is **a child** of the default route.

> This is because each tab page should be rendered **inside** the `LayoutPage` instead of completely replacing it.

Add a `children` property to the default route with an empty array, then move the three tab page's route in this empty array:

```ts
const routes: Routes = [
  {
    // Default route
    path: "",
    component: LayoutPage,
    children: [
      {
        // Route that loads the CreateTrip module
        path: "create-trip",
        loadChildren: () =>
          import("./create-trip/create-trip.module").then(
            (m) => m.CreateTripPageModule
          ),
      },
      {
        // Route that loads the PlacesMap module
        path: "places-map",
        loadChildren: () =>
          import("./places-map/places-map.module").then(
            (m) => m.PlacesMapPageModule
          ),
      },
      {
        // Route that loads the TripList module
        path: "trip-list",
        loadChildren: () =>
          import("./trip-list/trip-list.module").then(
            (m) => m.TripListPageModule
          ),
      },
    ],
  },
];
```

Now, **update the layout page's component** (`src/app/layout/layout.page.ts`) to include the list of tabs we want:

> You can also delete the empty `ngOnInit()` method and remove the `OnInit` interface implementation from the class definition.

```ts
import { Component } from "@angular/core";

// Custom type that represent a tab data.
declare type PageTab = {
  title: string; // The title of the tab in the tab bar
  icon: string; // The icon of the tab in the tab bar
  path: string; // The route's path of the tab to display
};

@Component({
  selector: "app-layout",
  templateUrl: "layout.page.html",
  styleUrls: ["layout.page.scss"],
})
export class LayoutPage {
  tabs: PageTab[];

  constructor() {
    this.tabs = [
      { title: "New Trip", icon: "add", path: "create-trip" },
      { title: "Places Map", icon: "map", path: "places-map" },
      { title: "Trip List", icon: "list", path: "trip-list" },
    ];
  }
}
```

> Be sure that the value of each `PageTab`'s `path` property matches the corresponding route in the `layout-routing.module.ts` file.

Third, we will **replace the ENTIRE content of the layout page's template** (`src/app/layout/layout.page.html`) with a template that uses Ionic's [Tabs component][ionic-tabs].

Angular's `ngFor` directive allows us to iterate over the `tabs` array we declared in the layout page's component,
and create an `<ion-tab-button>` element for each of them, instead of defining them by hand:

```html
<ion-tabs>
  <ion-tab-bar slot="bottom">
    <ion-tab-button [tab]="tab.path" *ngFor="let tab of tabs">
      <ion-icon [name]="tab.icon"></ion-icon>
      <ion-label>{{ tab.title }}</ion-label>
    </ion-tab-button>
  </ion-tab-bar>
</ion-tabs>
```

You should now be able to navigate between the 3 tabs!

![Serving the blank app](setup/tabs-setup.png)

### Default tab

If you want your user to be redirected in a specific tab when first loading your app, you can do so by updating the `routes` array in the `layout-routing.module.ts` file. For example, to define the `trip-list` tab as the default one:

```ts
const routes: Routes = [
  {
    path: "",
    component: LayoutPage,
    children: [
      // Previous routes
      {
        path: "",
        redirectTo: "trip-list", // Or whatever tabs is your default one
        pathMatch: "full",
      },
    ],
  },
];
```

## Set up security

To use the app, a user should identify themselves.

You will add a login screen that the user **must go through** before accessing the other screens.

Authentication will be performed by the [Travel Log API][travel-log-api] (or your own API if you don't use Travel Log)

The API requires a bearer token be sent to identify the user when making requests on some resources (e.g. when creating trips).
This bearer token must be sent in the `Authorization` header for all requests requiring identification.

Once login/logout is implemented, you will also set up an **HTTP interceptor** to automatically add this header to every request.

### Check the documentation of the API's authentication resource

The Travel Log API provides an [`/auth` resource](https://devmobil-travel-log-api.herokuapp.com/#api-Authentication-CreateAuthenticationToken)
on which you can make a POST request to authenticate.

You need to make a call that looks like this:

```json
POST /api/auth HTTP/1.1
Content-Type: application/json

{
  "username": "jdoe",
  "password": "test"
}
```

The response will contain the token we need for authentication,
as well as a representation of the authenticated user:

```json
HTTP/1.1 200 OK
Content-Type: application/json

{
  "token": "eyJhbGciOiJIU.eyJpc3MiOiI1OGM1YjUwZTA0Nm.gik21xyT4_NzsduWMLVp8",
  "user": {
    "createdAt": "2018-12-09T11:58:18.265Z",
    "href": "/api/users/d68cf4e9-1349-4d45-b356-c1294e49ef23",
    "id": "d68cf4e9-1349-4d45-b356-c1294e49ef23",
    "name": "jdoe",
    "tripsCount": 2,
    "updatedAt": "2018-12-09T11:58:18.265Z"
  }
}
```

You will need to perform this request and retrieve that information when the user logs in.

<a href="#top">↑ Back to top</a>

### Create model classes

Let's create a few classes to use as models when communicating with the API.
That way we will benefit from TypeScript's typing when accessing model properties.

> \*\*If you are NOT using the Travel Log API, you will need to adapt those models to what your API returns.

Create a `src/app/models/user.ts` file which exports a model representing a user of the API:

```ts
export type User = {
  id: string;
  href: string;
  name: string;
  tripsCount: number;
  createdAt: string;
  updatedAt: string;
};
```

Create a `src/app/models/auth-request.ts` file which exports a model representing a request to the authentication resource:

```ts
export type AuthRequest = {
  username: string;
  password: string;
};
```

Create a `src/app/models/auth-response.ts` file which exports a model representing a successful response from the authentication resource:

```ts
import { User } from "./user";

export type AuthResponse = {
  token: string;
  user: User;
};
```

### Create an authentication service

Since the new service we'll create will make Http requests, we **need** to import Angular's `HttpClientModule` in our app. Do so in the `src/app/app.module.ts` file:

```ts
// ...Other imports
// Import Angular's httpClientModule
import { HttpClientModule } from "@angular/common/http";

@NgModule({
  declarations: [AppComponent],
  entryComponents: [],
  imports: [
    /* ... */
    HttpClientModule, // Add Angular's httpClientModule to your module imports
  ],
  providers: [
    /* ... */
  ],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

Now, let's generate a reusable, injectable service to manage authentication:

```bash
$> ionic generate service auth/Auth
```

You can replace the content of the generated `src/app/auth/auth.service.ts` file with the following code:

```ts
import { HttpClient } from "@angular/common/http";
import { Injectable } from "@angular/core";
import { ReplaySubject, Observable } from "rxjs";
import { map } from "rxjs/operators";

import { AuthResponse } from "../models/auth-response";
import { User } from "../models/user";
import { AuthRequest } from "../models/auth-request";

const API_URL = "<REPLACE_ME>";

/**
 * Authentication service for login/logout.
 */
@Injectable({ providedIn: "root" })
export class AuthService {
  #auth$: ReplaySubject<AuthResponse | undefined>;

  constructor(private http: HttpClient) {
    this.#auth$ = new ReplaySubject(1);
    // Emit an empty value on startup for now
    this.#auth$.next();
  }

  isAuthenticated$(): Observable<boolean> {
    return this.#auth$.pipe(map((auth) => Boolean(auth)));
  }

  getUser$(): Observable<User> {
    return this.#auth$.pipe(map((auth) => auth?.user));
  }

  getToken$(): Observable<string> {
    return this.#auth$.pipe(map((auth) => auth?.token));
  }

  logIn$(authRequest: AuthRequest): Observable<User> {
    const authUrl = `${API_URL}/auth`;
    return this.http.post<AuthResponse>(authUrl, authRequest).pipe(
      map((auth) => {
        this.#auth$.next(auth);
        console.log(`User ${auth.user.name} logged in`);
        return auth.user;
      })
    );
  }

  logOut(): void {
    this.#auth$.next(null);
    console.log("User logged out");
  }
}
```

> You need to set the value of the constant `API_URL` to your dedicated API URL. In this example, `API_URL` will have a value of "https://devmobil-travel-log-api.herokuapp.com/api"

### Create the login screen

Generate a login page component to be added to the main `app-routing.module.ts` file:

```bash
$> ionic generate page auth/Login
```

Add the following HTML form **inside the `<ion-content>` tag** of `src/app/auth/login/login.page.html`:

> This assume that you have read [the slide doc explaining how to use forms with Angular and Ionic components][forms-with-angular-and-ionic]. If you **DID NOT**, now's the time...

```html
<form #loginForm="ngForm" (submit)="onSubmit(loginForm)">
  <ion-list>
    <!-- Username input -->
    <ion-item>
      <ion-label position="floating">Username</ion-label>
      <ion-input
        inputmode="text"
        #username="ngModel"
        required="true"
        name="username"
        [(ngModel)]="authRequest.username"
      ></ion-input>
    </ion-item>

    <!-- Error message displayed if the username is invalid -->
    <ion-item lines="none" *ngIf="username.invalid && username.touched">
      <ion-text color="danger">Username is required.</ion-text>
    </ion-item>

    <!-- Password input -->
    <ion-item>
      <ion-label position="floating">Password</ion-label>
      <ion-input
        inputmode="text"
        #password="ngModel"
        required="true"
        type="password"
        name="password"
        [(ngModel)]="authRequest.password"
      ></ion-input>
    </ion-item>

    <!-- Error message displayed if the password is invalid -->
    <ion-item lines="none" *ngIf="password.invalid && password.touched">
      <ion-text color="danger">Password is required.</ion-text>
    </ion-item>
  </ion-list>

  <div class="ion-padding">
    <!-- Submit button -->
    <ion-button type="submit" expand="block" [disabled]="loginForm.invalid"
      >Log in</ion-button
    >

    <!-- Error message displayed if the login failed -->
    <ion-text color="danger" *ngIf="loginError"
      >Username or password is invalid.</ion-text
    >
  </div>
</form>
```

Update `src/app/login/login.page.ts` as follows:

```ts
import { Component } from "@angular/core";
import { NgForm } from "@angular/forms";
import { Router } from "@angular/router";

import { AuthService } from "../auth.service";
import { AuthRequest } from "../../models/auth-request";

/**
 * Login page.
 */
@Component({
  templateUrl: "login.page.html",
})
export class LoginPage {
  /**
   * This authentication request object will be updated when the user
   * edits the login form. It will then be sent to the API.
   */
  authRequest: AuthRequest;

  /**
   * If true, it means that the authentication API has return a failed response
   * (probably because the name or password is incorrect).
   */
  loginError: boolean;

  constructor(private auth: AuthService, private router: Router) {
    this.authRequest = {
      username: undefined,
      password: undefined,
    };
  }

  /**
   * Called when the login form is submitted.
   */
  onSubmit(form: NgForm) {
    // Do not do anything if the form is invalid.
    if (form.invalid) {
      return;
    }

    // Hide any previous login error.
    this.loginError = false;

    // Perform the authentication request to the API.
    this.auth.logIn$(this.authRequest).subscribe({
      next: () => this.router.navigateByUrl("/"),
      error: (err) => {
        this.loginError = true;
        console.warn(`Authentication failed: ${err.message}`);
      },
    });
  }
}
```

<a href="#top">↑ Back to top</a>

### Use the authentication service to protect access to the layout page

Now that we have a service to manage authentication and a working page for users to log in, we need to make sure that unauthenticated user can not access restricted pages and are instead redirected to the login page.

We will use an [Angular Guard][angular-guard] to do this.

```bash
$> ionic generate guard auth/Auth
```

> When asked what interface you'd like to implement, select only the `CanActivate` interface.

Open the `src/app/auth/auth.guard.ts` file and **replace all its content** with:

```ts
import { Injectable } from "@angular/core";
import { CanActivate, Router, UrlTree } from "@angular/router";
import { Observable } from "rxjs";
import { AuthService } from "./auth.service";
import { map } from "rxjs/operators";

@Injectable({
  providedIn: "root",
})
export class AuthGuard implements CanActivate {
  constructor(private auth: AuthService, private router: Router) {}

  canActivate(): Observable<boolean | UrlTree> {
    return this.auth
      .isAuthenticated$()
      .pipe(
        map((isAuthenticated) =>
          isAuthenticated ? true : this.router.parseUrl("/login")
        )
      );
  }
}
```

To use this guard, open the `src/app/app-routing.module.ts` file and add a new `canActivate` property to the `''` route:

```ts
// ...Other imports
// TODO: import the AuthGuard
import { AuthGuard } from "./auth/auth.guard";

const routes: Routes = [
  {
    path: "",
    // Add the guard to the canActivate array of this route
    canActivate: [AuthGuard],
    loadChildren: () =>
      import("./layout/layout.module").then((m) => m.LayoutPageModule),
  },
  {
    path: "login",
    loadChildren: () =>
      import("./login/login.module").then((m) => m.LoginPageModule),
  },
];

@NgModule({
  imports: [
    /* ... */
  ],
  exports: [RouterModule],
})
export class AppRoutingModule {}
```

The login screen is ready!
If you reload your app, you should see that you are automatically redirected to the login page.

You can now try logging in, provided that there is an existing user in your API dataset.

> If it's not the case, you can create one with Postman by sending a request to the endpoint of your API that allows registerint new users

<a href="#top">↑ Back to top</a>

### Storing the authentication credentials

Now you can log in, but there's a little problem.

Every time the app is reloaded, you lose all data so you have to log back in.
This is particularly annoying for local development since the browser is automatically refreshed every time you change the code.

You need to use more persistent storage for the security credentials, that is the authentication token.
Ionic provides a [storage module][storage] which will automatically select an appropriate storage method for your platform.
It will use [SQLite][sqlite] on phones when available; for web platforms it will use [IndexedDB][indexed-db], [WebSQL][websql] or [Local Storage][local-storage].

To use the Ionic storage module, you must first install it:

```bash
$> npm i @ionic/storage-angular
```

Then import it into your application's module in `src/app/app.module.ts`:

```ts
// Other imports...
// TODO: import the ionic storage module.
import { IonicStorageModule } from "@ionic/storage-angular";

@NgModule({
  // ...
  imports: [
    // ...
    // TODO: add the ionic storage module into the app's module.
    IonicStorageModule.forRoot(),
  ],
  // ...
})
export class AppModule {}
```

Initialize the storage in `AppComponent` in `src/app/app.component.ts`:

```ts
// Other imports...
import { Storage } from "@ionic/storage";

export class AppComponent {
  constructor(storage: Storage) {
    storage.create();
  }
}
```

Now you can import the `Storage` service in `AuthService` in `src/app/auth/auth.service.ts`:

```ts
// Other imports...
// Import RxJS's from function, delayWhen operator and Ionic's storage provider.
import { Observable, ReplaySubject, from } from "rxjs";
import { delayWhen, map } from "rxjs/operators";
import { Storage } from "@ionic/storage";
```

You also need to inject it into the constructor:

```ts
constructor(private http: HttpClient, private storage: Storage)
```

Add a method to persist the authentication information using the storage module:

```ts
private saveAuth$(auth: AuthResponse): Observable<void> {
  return from(this.storage.set('auth', auth));
}
```

The storage module returns Promises, but we'll be plugging this new function into `logIn()` which uses Observables,
so we convert the Promise to an Observable before returning it, with the `from` function.

> The `from` method can be imported from `rxjs`

You can now update the `logIn()` method to persist the API's authentication response with the new `saveAuth()` method.
To do that, use RxJS's [`delayWhen`][rxjs-delay-when] operator, which allows us to delay an Observable stream (in this case, the one that indicates our user is authenticated) until another Observable emits
(in this case, the one that saves the authentication response).

> This way, we only emit the `auth.user` when we are **sure** that the `auth` object has been saved in the storage.

```ts
logIn(authRequest: AuthRequest): Observable<User> {

  const authUrl = `${API_URL}/auth`;
  return this.http.post<AuthResponse>(authUrl, authRequest).pipe(
    // Delay the observable stream while persisting the authentication response.
    delayWhen((auth) => this.saveAuth$(auth)),
    map(auth => {
      this.#auth$.next(auth);
      console.log(`User ${auth.user.name} logged in`);
      return auth.user;
    })
  );
}
```

> the `delayWhen` function can be imported from `rxjs/operators`.

When testing in the browser, you should already see the object being stored in IndexedDB (the default storage if using Chrome).

You must now load it when the app starts.
You can do that in the constructor of `AuthService` by replacing the line `this.#auth$.next();`:

```ts
constructor(private http: HttpClient, private storage: Storage) {
  this.#auth$ = new ReplaySubject(1);
  this.storage.get('auth').then((auth) => {
    // Emit the loaded value into the observable stream.
    this.#auth$.next(auth);
  });
}
```

> Since the storage provider's `get` method returns a promise, you can only use the result in a `.then` asynchronous callback:

Your app should now remember user credentials even when you reload it!

Finally, also update the `AuthService`'s `logOut()` method to remove the stored authentication from storage when a user logs out:

```ts
logOut() {
  this.#auth$.next(null);
  // Remove the stored authentication from storage when logging out.
  this.storage.remove('auth');
  console.log('User logged out');
}
```

<a href="#top">↑ Back to top</a>

### Log out

You should also add a UI component to allow the user to log out.
**As an example**, we will display a logout button in the title bar of the trip creation screen.

> This is **only for example**. In your real project, you might want to put this logout button in a more appropriate location... Just sayin'

Add an `<ion-buttons>` tag with a logout button in `src/app/layout/create-trip/create-trip.page.html`:

```html
<ion-header>
  <ion-toolbar>
    <ion-title>CreateTrip</ion-title>

    <!-- Logout button -->
    <ion-buttons slot="end">
      <ion-button (click)="logOut()">
        <ion-icon name="log-out"></ion-icon>
      </ion-button>
    </ion-buttons>
  </ion-toolbar>
</ion-header>
```

Let's assume that when logging out, we want the user redirected to the login page. To do that, you will need to:

- Inject the Angular Router, which will allow you to navigate to a defined route ;
- Inject the `AuthService`, so that we can use use its `logOut` method,
- Add a `logOut` method to the `CreateTripPage` component, since it's what we call in its HTML template above.

After doing all that, your `CreateTripPage` component should look something like this:

```ts
// Other imports...
import { Router } from "@angular/router";
import { AuthService } from "src/app/auth/auth.service";

@Component({
  /* ... */
})
export class CreateTripPage implements OnInit {
  constructor(
    // Inject the authentication provider.
    private auth: AuthService,
    // Inject the router
    private router: Router
  ) {}

  ngOnInit() {}

  // Add a method to log out.
  logOut() {
    console.log("logging out...");
    this.auth.logOut();
    this.router.navigateByUrl("/login");
  }
}
```

You should now see the logout button in the navigation bar after logging in.

> If you need to include this button in other places of your UI, it could be a good idea to create a dedicated component for this, which will contains the logout logic. This will prevent you from copy pasting the same logic every time you need it.

<a href="#top">↑ Back to top</a>

### Configuring an HTTP interceptor

Now that you have login and logout functionality, and an authentication service that stores an authentication token, you can authenticate for other API calls.

Looking at the API documentation, at some point you will need to [create a trip](https://devmobil-travel-log-api.herokuapp.com/#api-Trips-CreateTrip).
The documentation states that you must send a bearer token in the `Authorization` header, like this:

```
POST /api/trips HTTP/1.1
Authorization: Bearer 0a98wumv
Content-Type: application/json

{"some":"json"}
```

With Angular, you would have to make this call like this every time:

> **NOTE** that this is an example. You **should** not do this in your application's code.

```js
httpClient.post("http://example.com/path", body, {
  headers: {
    Authorization: `Bearer ${token}`,
  },
});
```

But it's a bit annoying to have to manually specify this header **for every request**.
After all, we know that we need it for most calls.

[HttpInterceptor][http-interceptor]s are Angular services that can be registered with the HTTP client to automatically intercept requests and change them or their responses according to your needs.

> You can learn more about Http interceptors by reading [this article](https://medium.com/@ryanchenkie_40935/angular-authentication-using-the-http-client-and-http-interceptors-2f9d1540eb8).

This solves our problem: we want to register an interceptor that will automatically add the `Authorization` header to all requests **if the user is logged in**.

To demonstrate that it works, start by adding a call to list trips in the `TripListPage` component in `src/app/layout/trip-list/trip-list.ts`:

```ts
// Other imports...
// TODO: import Angular's HTTP client.
import { HttpClient } from "@angular/common/http";
import { ViewDidEnter } from "@ionic/angular";
import { AuthService } from "src/app/auth/auth.service";

@Component({
  selector: "page-trip-list",
  templateUrl: "trip-list.html",
})
export class TripListPage implements ViewDidEnter {
  constructor(
    // Inject the AuthService
    private auth: AuthService,
    // Inject the HTTP client
    public http: HttpClient
  ) {}

  ionViewDidEnter(): void {
    // Make an HTTP request to retrieve the trips.
    const url = "https://devmobil-travel-log-api.herokuapp.com/api/trips";
    this.http.get(url).subscribe((trips) => {
      console.log(`Trips loaded`, trips);
    });
  }

  // ...
}
```

> **⚠** Doing an HTTP request **inside a component's code** is **NOT** a best practice. Components should not be responsible of retrieving the data, they should only be responsible of asking another service for it and providing it to their template.

> In your application, you should define dedicated services that will handle calling your API.

If you display the trip list tab and check XHR Network requests in your browser's developer tools,
you will see that there is no `Authorization` header sent even when the user is logged in.

Now let's generate the interceptor service:

```bash
$> ionic generate service auth/AuthInterceptor
```

Put the following content in the generated `src/app/auth/auth-interceptor.service.ts` file:

> Read the code to try and understand what's going on.

```ts
import {
  HttpRequest,
  HttpHandler,
  HttpEvent,
  HttpInterceptor,
} from "@angular/common/http";
import { Injectable, Injector } from "@angular/core";
import { Observable } from "rxjs";
import { first, switchMap } from "rxjs/operators";

import { AuthService } from "./auth.service";

@Injectable({ providedIn: "root" })
export class AuthInterceptorService implements HttpInterceptor {
  constructor(private injector: Injector) {}

  intercept(
    req: HttpRequest<any>,
    next: HttpHandler
  ): Observable<HttpEvent<any>> {
    // Retrieve AuthService at runtime from the injector.
    // (Otherwise there would be a circular dependency:
    //  AuthInterceptorService -> AuthService -> HttpClient -> AuthInterceptorService).
    const auth = this.injector.get(AuthService);

    // Get the bearer token (if any).
    return auth.getToken$().pipe(
      first(),
      switchMap((token) => {
        // Add it to the request if it doesn't already have an Authorization header.
        if (token && !req.headers.has("Authorization")) {
          req = req.clone({
            headers: req.headers.set("Authorization", `Bearer ${token}`),
          });
        }
        return next.handle(req);
      })
    );
  }
}
```

Now you simply need to register this interceptor in your application main module.
Since it's an HTTP interceptor, it's not like other providers and must be registered in a special way.

In `src/app/app.module.ts`, add:

```ts
// Other imports...
import { HttpClientModule, HTTP_INTERCEPTORS } from "@angular/common/http";
import { AuthInterceptorService } from "./auth/auth-interceptor.service";

@NgModule({
  // ...
  providers: [
    // Other providers...
    {
      provide: HTTP_INTERCEPTORS,
      useClass: AuthInterceptorService,
      multi: true,
    },
  ],
})
export class AppModule {}
```

The `multi: true` option is necessary because you can register multiple interceptors if you want
(read more about [multi providers](https://blog.thoughtram.io/angular2/2015/11/23/multi-providers-in-angular-2.html)).

Now all your API calls will have the `Authorization` header when the user is logged in.

> You can verify this by looking at the XHR Network requests in the trip list tab and search for the `Authorization` header in the request's headers

<a href="#top">↑ Back to top</a>

## Multi-environment & sensitive configuration

Sometimes you might have to store values that should not be committed to version control:

- **Environment-specific** values that may change depending on where you deploy your app
- **Sensitive information** like access tokens or passwords.

For example, in our earlier HTTP calls, the URL was **hardcoded**:

```ts
const url = "https://devmobil-travel-log-api.herokuapp.com/api/trips";
this.http.get(url).subscribe((trips) => {
  // ...
});
```

This is not optimal considering the multi-environment problem (and the fact that we copy-pasted it several time)
If you wanted to change environments, you would have to manually change the URL every time.

Let's find a way to centralize this configuration.

<a href="#top">↑ Back to top</a>

### Environment files

There is already a mechanism in place to handle those environment-specific values with Angular.

In `src/environments`, you should find two files: `environment.ts` and `environment.prod.ts`.

The purpose of those file is to hold the configuration values of a specific environment, so that you could easily swap one config with another to deploy your app in different environment ("development", "test", "staging", "production", etc).

The first file, `environment.ts` is the default file and the one that should hold the configuration for your development environment. It should **not be committed** as your development config might be different than the one of your fellows developers.

The other one, `environment.prod.ts`, is the file that will contain production specific values. **It should not be commited** at all.

Alas, both those files have already been commited when the project was set up... It's not a huge problem as both those files don't contain anything sensitive.

You need to tell git to untrack them, though, so **delete both of them** from your filesystem (we'll recreate them later), then **commit those deletions** in git:

```bash
$> rm src/environments/*
$> git add src/environments/*
$> git commit -m "Remove environment files from git"
```

Now, create a **placeholder file** whose purpose is to describe what are the environment data used in the app, so that each developer can create its own `environment.ts` on their local copy of the project.
Create the `environment.sample.ts` file in `src/environments`, with this exact content (comment included):

```ts
// Copy this file to environment.ts and replace the values with your configuration
export const environment = {
  production: false,
  apiUrl: "https://example.com/api",
};
```

> The `environment.sample.ts` file is a placeholder and should **NOT** contain the actual configuration.

<a href="#top">↑ Back to top</a>

### Create the actual configuration file

With this placeholder file, you can now (re)create the actual `src/environments/environment.ts` configuration file (by copying and renaming the `environment.sample.ts` file), this time with your actual configuration values, at least for development:

```ts
export const environment = {
  production: false,
  apiUrl: "https://devmobil-travel-log-api.herokuapp.com/api",
};
```

While we're at it, let's also (re)create the `environment.prod.ts` file, used for production builds, with this content:

```ts
export const environment = {
  production: true,
  // That's the same api Url in our case, but in real project, it would certainly be different (you don't want to develop using the same instance as the production application...)
  apiUrl: "https://devmobil-travel-log-api.herokuapp.com/api",
};
```

<a href="#top">↑ Back to top</a>

### Add the environment files to your `.gitignore` file

Of course, you **don't want to commit neither `environment.ts` nor `environment.prod.ts`**, but you do want to commit `environment.sample.ts`
so that anyone who clones your project can see what configuration options are required.
To do so, add these lines at the bottom of your `.gitignore` file:

```
# Environment files
src/environments/*
!src/environments/environment.sample.ts
```

The first line tells git not to track any file in the `src/environments` folder... except for the specific `environment.sample.ts` file (this is the second line).

You now have your uncommitted environment files!

### When are the environment files used

The `environment.ts` file is loaded when you execute the `ionic serve` command.

> If you have any `ionic serve` command running, kill it and start it again to apply the changes.

When executing the `ionic serve` command with the `--prod` flag, like so...

```bash
$> ionic serve --prod
```

...Ionic tells Angular to replaces the content of the `environment.ts` file by the content from `environment.prod.ts`, in the build (**it does not replace the content of the actual file, thankfully**).

This way, whatever the environment your app is running on, `environment.ts` is **always the file holding the adequate configuration!**

<a href="#top">↑ Back to top</a>

### Feed the configuration to Angular

Now that you have your configuration file, you want to use its values in your code.

Since it's a TypeScript file like any other, you simply have to import and use it.

> **Remember that you should only import `environment.ts` in your code**, not the `environment.prod.ts` or any other variant, as it's content will change depending on the environment.

```ts
// Other imports...
// TODO: import the environment config.
import { environment } from "src/environments/environment";

// ...
export class TripListPage {
  // ...
  ngOnInit() {
    const url = `${environment.apiUrl}/trips`;
    this.http.get(url).subscribe((trips) => {
      console.log(`Trips loaded`, trips);
    });
  }
  // ...
}
```

Do not forget to also update the authentication service in `src/app/auth/auth.service.ts`, which also has a hardcoded URL:

```ts
// Other imports...
// TODO: import the environment config.
import { environment } from "src/environments/environment";

// ...
export class AuthService {
  // ...
  logIn(authRequest: AuthRequest): Observable<User> {
    // TODO: replace the hardcoded API URL by the one from the environment config.
    const authUrl = `${environment.apiUrl}/auth`;
    // ...
  }
  // ...
}
```

> You can then safely delete the line that defines the `API_URL` constant.

<a href="#top">↑ Back to top</a>

[angular-component]: https://angular.io/guide/architecture#components
[angular-guard]: https://angular.io/guide/router#guards
[cordova]: https://cordova.apache.org/
[forms-with-angular-and-ionic]: https://mediacomem.github.io/comem-devmobil/latest/subjects/angular-forms-ionic/#1
[ionic]: https://ionicframework.com/
[ionic-tabs]: https://ionicframework.com/docs/api/tabs
[lazy-loading]: https://angular.io/guide/lazy-loading-ngmodules
[rxjs-delay-when]: http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html#instance-method-delayWhen
[sass]: http://sass-lang.com
[travel-log-api]: https://devmobil-travel-log-api.herokuapp.com
[storage]: https://github.com/ionic-team/ionic-storage
[sqlite]: https://sqlite.org
[indexed-db]: https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API
[websql]: https://www.w3.org/TR/webdatabase/
[local-storage]: https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage
[http-interceptor]: https://angular.io/api/common/http/HttpInterceptor
