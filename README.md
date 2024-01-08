# COMEM+ Travel Log Ionic Setup

<a name="top"></a>

This repository contains instructions to build a skeleton application that can serve as a starting point to develop the Travel Log mobile application.
The completed skeleton app is available [here](https://github.com/Tazaf/comem-travel-log-ionic-starter).

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

These instructions assume that you are using the Travel Log API based on one of the suggestions of the previous course [Web-Oriented Architecture](https://github.com/MediaComem/comem-archioweb),
and that you are familiar with the [documentation of the reference API](https://comem-travel-log-api.onrender.com/).

You will need to have [Node.js](https://nodejs.org) installed.
The latest LTS (Long Term Support) version is recommended (^20.0.0 at the time of writing).

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

Before diving into the code, you should **always take a moment to design the user interface** of your app.
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

Make sure you have the [Ionic CLI][ionic-cli] installed, and that your computer is correctly configured to deploy on a mobile device :

```bash
$> ionic -v
7.1.5
```

> If you have an error when running the above command, this probably means that you need to install the [Ionic CLI][ionic-cli]. To do so, execute:
>
> ```bash
> $> npm install -g ionic@latest
> ```

Go in the directory where you want your app source code to be located, then start generating your Ionic app with the following command:

> No need to create a dedicated directory for your app ; it will be created for you by the CLI tool

```bash
$> ionic start
```

The command will ask you if you want to use the app creation wizard. Answering "yes" will open your browser on a user-friendly configuration page (note that you'll need a free Ionic account to finish the wizard). This is **entirely optional** and you can define this configuration using the CLI creation wizard also.

> Since there is no way of generating an app with the **Blank** starter from this wizard, its recommended to answer "No" and keep using the CLI instead.

You'll be asked to give a name to your new application. You are free to name it as you wish. For the rest of this starter though, we'll call it `travel-log`.

Whatever you choose to do, be sure to:

- Select **Angular** as your app's underlying framework
- Select the **Standalone** approach for Angular
- Select the **Blank** starter template

  > If you are using the browser creation wizard, select whichever template seems to fit best your app's mockups.

  > Even though it is not recommended to start your project using a template other than the **Blank** one, you are more than advised to generate test apps using those other starters to look at how they are developed and use what you need in your own app.

Then wait for the install to proceed... ⏳

Go into the app directory. The `ionic start` command should already have initialized a Git repository and made the first commit, so let's check this out:

```bash
$> cd travel-log
$> git log
commit XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX (HEAD -> master)
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

We will create a `Layout` page that will contain the layout for the tabs (that is where the tabs bar and the slot for each tab will be displayed) ; each tab page should therefor be a subpage of `Layout`.

First let's **delete the `src/app/home` folder entirely**. Then update the `app.routes.ts` file to empty the routes array:

```ts
const routes: Routes = [
  // Nothing here !
];
```

Then, use the `ionic generate` command to create the `Layout` page.

```bash
$> ionic generate page Layout
```

Update the `app.route.ts` file to add the routes leading to this new `Layout` page:

```ts
import { Routes } from "@angular/router";

export const routes: Routes = [
  {
    path: "",
    loadComponent: () =>
      import("./layout/layout.page").then((m) => m.LayoutPage),
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
src/app/layout/create-trip/create-trip.page.scss
src/app/layout/create-trip/create-trip.page.html
src/app/layout/create-trip/create-trip.page.spec.ts
src/app/layout/create-trip/create-trip.page.ts
```

> For the PlacesMap page

```
src/app/layout/places-map/places-map.page.scss
src/app/layout/places-map/places-map.page.html
src/app/layout/places-map/places-map.page.spec.ts
src/app/layout/places-map/places-map.page.ts
```

> For the TripList page

```
src/app/layout/trip-list/trip-list.page.scss
src/app/layout/trip-list/trip-list.page.html
src/app/layout/trip-list/trip-list.page.spec.ts
src/app/layout/trip-list/trip-list.page.ts
```

For each page, we have:

- An HTML template (`*.page.html`).
- A [Sass/SCSS][sass] stylesheet (`*.page.scss`).
- An Angular component (`*.page.ts`).
- A test suite with a default test (`*.page.spec.ts`).

Now update the HTML template for each page and add some content at the end of the `<ion-content>` tag.
For example, in `src/app/layout/create-trip/create-trip.page.html`:

```html
<ion-content [fullscreen]="true">
  <!-- Leave the previous content here -->
  <!-- Add the div below. The .ion-padding class adds some space around the content -->
  <div class="ion-padding">Let's create some trips</div>
</ion-content>
```

### Update the app to use the pages

Now that the pages are ready, we need to display them.

For each of your new pages, Ionic added a route leading to them in `app.routes.ts`. This should look like this (minus the comments):

```ts
import { Routes } from "@angular/router";

export const routes: Routes = [
  {
    // Default route
    path: "",
    loadComponent: () =>
      import("./layout/layout.page").then((m) => m.LayoutPage),
  },
  {
    // Route that loads the CreateTripPage component
    path: "create-trip",
    loadComponent: () =>
      import("./layout/create-trip/create-trip.page").then(
        (m) => m.CreateTripPage
      ),
  },
  {
    // Route that loads the PlacesMapPage component's
    path: "places-map",
    loadComponent: () =>
      import("./layout/places-map/places-map.page").then(
        (m) => m.PlacesMapPage
      ),
  },
  {
    // Route that loads the TripListPage component
    path: "trip-list",
    loadComponent: () =>
      import("./layout/trip-list/trip-list.page").then((m) => m.TripListPage),
  },
];
```

For the tabs to propertly works, you need to **change this structure** so that each tab page route is **a child** of the `LayoutPage` route.

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
import { CommonModule } from "@angular/common";
import { Component } from "@angular/core";
import { FormsModule } from "@angular/forms";
import { IonicModule } from "@ionic/angular";
import { add, map, list } from "ionicons/icons";

// Custom type that represent a tab data.
declare type PageTab = {
  title: string; // The title of the tab in the tab bar
  icon: string; // The icon of the tab in the tab bar
  path: string; // The route's path of the tab to display
};

@Component({
  selector: "app-layout",
  templateUrl: "./layout.page.html",
  styleUrls: ["./layout.page.scss"],
  standalone: true,
  imports: [IonicModule, CommonModule, FormsModule],
})
export class LayoutPage {
  tabs: PageTab[];

  constructor() {
    this.tabs = [
      { title: "New Trip", icon: add, path: "create-trip" },
      { title: "Places Map", icon: map, path: "places-map" },
      { title: "Trip List", icon: list, path: "trip-list" },
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
      <ion-icon [icon]="tab.icon"></ion-icon>
      <ion-label>{{ tab.title }}</ion-label>
    </ion-tab-button>
  </ion-tab-bar>
</ion-tabs>
```

You should now be able to navigate between the 3 tabs!

![Serving the blank app](setup/tabs-setup.png)

### Default tab

If you want your user to be redirected to a specific tab when first loading your app, you can do so by updating the `routes` array in the `app.routes.ts` file. For example, to define the `trip-list` tab as the default one:

```ts
import { Routes } from "@angular/router";

export const routes: Routes = [
  {
    path: "",
    loadComponent: () =>
      import("./layout/layout.page").then((m) => m.LayoutPage),
    children: [
      // Previous routes
      {
        path: "",
        redirectTo: "trip-list", // Or whatever tab should be the default one.
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

The API requires a "Bearer token" be sent to identify the user when making requests on some resources (e.g. when creating trips).
This token must be sent in the `Authorization` header for all requests requiring identification.

Once login/logout is implemented, you will also set up an **HTTP interceptor** to automatically add this header to every request.

### Check the documentation of the API's authentication resource

The Travel Log API provides an [`/auth` resource](https://demo-travel-log-api.onrender.com/#api-Authentication-CreateAuthenticationToken)
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

> **If you are NOT using the Travel Log API, you will need to adapt those models to what your API returns**.

Create a `src/app/security/user.model.ts` file which exports a model representing a user of the API:

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

Create a `src/app/security/auth-request.model.ts` file which exports a model representing a request to the authentication resource:

```ts
export type AuthRequest = {
  username: string;
  password: string;
};
```

Create a `src/app/security/auth-response.model.ts` file which exports a model representing a successful response from the authentication resource:

```ts
import { User } from "./user.model";

export type AuthResponse = {
  token: string;
  user: User;
};
```

### Create an authentication service

Since the new service we'll create will make Http requests, we **need** to provide Angular's `HttpClient` own service to our entire app. Do so in the `src/main.ts` file:

```ts
// Other imports should not be touched
import { enableProdMode, importProvidersFrom } from "@angular/core";
import { provideHttpClient } from "@angular/common/http";

if (environment.production) {
  enableProdMode();
}

bootstrapApplication(AppComponent, {
  providers: [
    // Previous providers should not be touched...
    provideHttpClient(), // <-- Add this line
  ],
});
```

Now, let's generate a reusable, injectable service to manage authentication:

```bash
$> ionic generate service security/Auth
```

You can replace the content of the generated `src/app/security/auth.service.ts` file with the following code:

```ts
import { Injectable } from "@angular/core";
import { Observable, ReplaySubject, filter, map } from "rxjs";
import { AuthResponse } from "./auth-response.model";
import { HttpClient } from "@angular/common/http";
import { User } from "./user.model";
import { AuthRequest } from "./auth-request.model";

/***********************************************************/
/*********!!! REPLACE BELOW WITH YOUR API URL !!! **********/
/***********************************************************/
const API_URL = "<REPLACE_ME>";

/**
 * Authentication service for login/logout.
 */
@Injectable({ providedIn: "root" })
export class AuthService {
  #auth$: ReplaySubject<AuthResponse | undefined>;

  constructor(private http: HttpClient) {
    this.#auth$ = new ReplaySubject(1);
    // Emit an undefined value on startup for now
    this.#auth$.next(undefined);
  }

  /**
   * @returns An `Observable` that will emit a `boolean` value
   * indicating whether the current user is authenticated.
   * This `Observable` will never complete and must be unsubscribed for when not needed.
   */
  isAuthenticated$(): Observable<boolean> {
    return this.#auth$.pipe(map((auth) => Boolean(auth)));
  }

  /**
   * @returns An `Observable` that will emit the currently authenticated `User` object only if there
   * currently is an authenticated user.
   */
  getUser$(): Observable<User | undefined> {
    return this.#auth$.pipe(map((auth) => auth?.user));
  }

  /**
   * @returns An `Observable` that will emit the currently authenticated user's `token`, only if there
   * currently is an authenticated user.
   */
  getToken$(): Observable<string | undefined> {
    return this.#auth$.pipe(map((auth) => auth?.token));
  }

  /**
   * Sends an authentication request to the backend API in order to log in a user with the
   * provided `authRequest` object.
   *
   * @param authRequest An object containing the authentication request params
   * @returns An `Observable` that will emit the logged in `User` object on success.
   */
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

  /**
   * Logs out the current user.
   */
  logOut(): void {
    this.#auth$.next(undefined);
    console.log("User logged out");
  }
}
```

> You need to set the value of the constant `API_URL` to your dedicated API URL. In this example, `API_URL` will have a value of "https://demo-travel-log-api.onrender.com/api"

### Create the login screen

Generate a login page component with:

```bash
$> ionic generate page auth/Login
```

> This will add a `login` route at the end of the `app.routes.ts` array.

Add the following HTML form **at the end of the `<ion-content>` tag** of `src/app/security/login/login.page.html`:

> This assume that you have read [the slide doc explaining how to use forms with Angular and Ionic components][forms-with-angular-and-ionic]. If you **DID NOT**, now would be an appropriate time...

```html
<div class="ion-padding">
  <form #loginForm="ngForm" (submit)="onSubmit(loginForm)">
    <!-- Username input -->
    <ion-input
      label="Username"
      labelPlacement="floating"
      inputmode="text"
      #username="ngModel"
      required="true"
      name="username"
      [(ngModel)]="authRequest.username"
    ></ion-input>
    <!-- Error message displayed if the username is invalid -->
    <ion-text color="danger" *ngIf="username.invalid && username.touched"
      >Username is required.</ion-text
    >

    <!-- Password input -->
    <ion-input
      label="Password"
      labelPlacement="floating"
      inputmode="text"
      #password="ngModel"
      required="true"
      type="password"
      name="password"
      [(ngModel)]="authRequest.password"
    ></ion-input>

    <!-- Error message displayed if the password is invalid -->
    <ion-text color="danger" *ngIf="password.invalid && password.touched"
      >Password is required.</ion-text
    >

    <!-- Submit button -->
    <ion-button type="submit" expand="block" [disabled]="loginForm.invalid"
      >Log in</ion-button
    >

    <!-- Error message displayed if the login failed -->
    <ion-text color="danger" *ngIf="loginError"
      >Username or password is invalid.</ion-text
    >
  </form>
</div>
```

Update `src/app/security/login/login.page.ts` as follows:

```ts
import { CommonModule } from "@angular/common";
import { Component } from "@angular/core";
import { FormsModule, NgForm } from "@angular/forms";
import { Router } from "@angular/router";
import { IonicModule } from "@ionic/angular";
import { AuthRequest } from "../auth-request.model";
import { AuthService } from "../auth.service";

@Component({
  selector: "app-login",
  templateUrl: "./login.page.html",
  styleUrls: ["./login.page.scss"],
  standalone: true,
  imports: [IonicModule, CommonModule, FormsModule],
})
export class LoginPage {
  /**
   * This authentication request object will be updated when the user
   * edits the login form. It will then be sent to the API.
   *
   * NOTE: The "Partial<AuthRequest>" type here has the same properties as "AuthRequest",
   * but they are all optional.
   */
  authRequest: Partial<AuthRequest> = {};

  /**
   * If true, it means that the authentication API has return a failed response
   * (probably because the name or password is incorrect).
   */
  loginError = false;

  constructor(private auth: AuthService, private router: Router) {
    this.authRequest = {};
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
    // NOTE: Since our form is valid, it means that "this.authRequest" is actually
    // a perfectly valid "AuthRequest" object, and that's what we are telling TypeScript
    // here with "as AuthRequest".
    this.auth.logIn$(this.authRequest as AuthRequest).subscribe({
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

Create a new file at `src/app/security/only-authenticated.guard.ts` with the following content:

```ts
import { inject } from "@angular/core";
import { CanActivateFn, Router } from "@angular/router";
import { AuthService } from "./auth.service";
import { map } from "rxjs";

export const onlyAuthenticated: CanActivateFn = () => {
  const authService = inject(AuthService);
  const router = inject(Router);

  return authService
    .isAuthenticated$()
    .pipe(
      map((isAuthenticated) =>
        isAuthenticated ? true : router.parseUrl("/login")
      )
    );
};
```

To use this guard, open the `src/app/app.routes.ts` file and add a new `canActivate` property to the first `''` route:

```ts
// Previous imports remain untouched
import { onlyAuthenticated } from "./security/only-authenticated.guard";

export const routes: Routes = [
  {
    path: "",
    loadComponent: () =>
      import("./layout/layout.page").then((m) => m.LayoutPage),
    canActivate: [onlyAuthenticated], // <-- Add this line
    children: [
      // Children routes remain untouched
    ],
  },
  {
    path: "login",
    loadComponent: () =>
      import("./security/login/login.page").then((m) => m.LoginPage),
  },
];
```

The login screen is ready!
If you reload your app, you should see that you are automatically redirected to the login page.

You can now try logging in, provided that there is an existing user in your API dataset.

> If it's not the case, you can create one with Postman by sending a request to the endpoint of your API that allows registering new users

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

Then add it to the `importProvidersFrom()` function call in `src/main.ts`:

```ts
// Other imports remain untouched
import { IonicStorageModule } from "@ionic/storage-angular";

if (environment.production) {
  enableProdMode();
}

bootstrapApplication(AppComponent, {
  providers: [
    // Other providers remain untouched
    importProvidersFrom(IonicStorageModule.forRoot()),
  ],
});
```

Initialize the storage in `AppComponent` in `src/app/app.component.ts`:

```ts
// Other imports...
import { Storage } from "@ionic/storage-angular";

export class AppComponent {
  constructor(storage: Storage) {
    storage.create();
  }
}
```

Now you can import the `Storage` service in `AuthService` in `src/app/security/auth.service.ts`:

```ts
// Other imports remain untouched
import { /* Other imports */, from } from "rxjs";
import { Storage } from "@ionic/storage-angular";

/**
 * Authentication service for login/logout.
 */
@Injectable({ providedIn: "root" })
export class AuthService {
  // Untouched code

  constructor(private http: HttpClient, private readonly storage: Storage) {
    // Untouched code
  }

  // Untouched code

  /**
   * Persists the provided `AuthResponse` to the storage.
   *
   * @param auth The AuthResponse to persist
   * @returns An `Observable` that will emit when the authentication is persisted
   */
  #saveAuth$(auth: AuthResponse): Observable<void> {
    return from(this.storage.set("auth", auth));
  }
}

// Untouched code
```

The storage module returns Promises, but we'll be plugging this new function into `logIn$()` which uses Observables,
so we convert the Promise to an Observable before returning it, with the `from` function.

> The `from` method can be imported from `rxjs`

You can now update the `logIn$()` method to persist the API's authentication response with the new `#saveAuth$()` method.
To do that, use RxJS's [`delayWhen`][rxjs-delay-when] operator, which allows us to delay an Observable stream (in this case, the one that indicates our user is authenticated) until another Observable emits
(in this case, the one that saves the authentication response).

> This way, we only emit the `auth.user` when we are **sure** that the `auth` object has been saved in the storage.

```ts
logIn$(authRequest: AuthRequest): Observable<User> {

  const authUrl = `${API_URL}/auth`;
  return this.http.post<AuthResponse>(authUrl, authRequest).pipe(
    // Delay the observable stream while persisting the authentication response.
    delayWhen((auth) => this.#saveAuth$(auth)),
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
        <ion-icon [icon]="logOutIcon"></ion-icon>
      </ion-button>
    </ion-buttons>
  </ion-toolbar>
</ion-header>
```

Let's assume that when logging out, we want the user redirected to the login page. To do that, you will need to:

- Inject the Angular Router, which will allow you to navigate to a defined route ;
- Inject the `AuthService`, so that we can use use its `logOut()` method,
- Add a `logOut()` method in the `CreateTripPage` component, since it's what we call in its HTML template above.

We'll also need to import the logo what we use in the button.

After doing all that, your `CreateTripPage` component should look something like this:

```ts
// Other imports...
import { Router } from "@angular/router";
import { AuthService } from "src/app/security/auth.service";
import { logOut as logOutIcon } from "ionicons/icons";

@Component({
  /* ... */
})
export class CreateTripPage implements OnInit {
  readonly logOutIcon = logOutIcon;

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

Looking at the API documentation, at some point you will need to [create a trip](https://comem-travel-log-api.onrender.com/#api-Trips-CreateTrip).
The documentation states that you must send a bearer token in the `Authorization` header, like this:

```
POST /api/trips HTTP/1.1
Authorization: Bearer 0a98wumv
Content-Type: application/json

{"some":"json"}
```

With Angular, you would have to make this call like this every time:

> **NOTE** that this is an example. You **should not** do this in your application's code.

```js
this.http.post("http://example.com/path", body, {
  headers: {
    Authorization: `Bearer ${token}`,
  },
});
```

But it's a bit annoying to have to manually specify this header **for every request**.
After all, we know that we need it for most calls.

[HttpInterceptor][http-interceptor]s are Angular services that can be registered with the HTTP client to automatically intercept requests and change them or their responses according to your needs.

This solves our problem: we want to register an interceptor that will automatically add the `Authorization` header to all requests **if the user is logged in**.

To demonstrate that it works, start by adding a call to list trips in the `TripListPage` component in `src/app/layout/trip-list/trip-list.page.ts`:

```ts
// Other imports remain untouched
import { IonicModule, ViewWillEnter } from "@ionic/angular";
import { HttpClient } from "@angular/common/http";

@Component({
  /* ... */
})
export class TripListPage implements ViewWillEnter {
  constructor(private readonly http: HttpClient) {}

  ionViewWillEnter(): void {
    // Make an HTTP request to retrieve the trips.
    const url = "https://demo-travel-log-api.onrender.com/api/trips";
    this.http.get(url).subscribe((trips) => {
      console.log(`Trips loaded`, trips);
    });
  }
}
```

> **⚠** Doing an HTTP request **inside a component's code** is **NOT** a best practice. Components should not be responsible of retrieving the data, they should only be responsible of asking another service for it and providing it to their template.

> In your application, you should define dedicated services that will handle calling your API.

If you display the trip list tab and check XHR Network requests in your browser's developer tools,
you will see that there is no `Authorization` header sent even when the user is logged in.

Let's create a new file at `src/app/security/auth.interceptor.ts`, with the following content:

> Read the code to try and understand what's going on.

```ts
import { HttpInterceptorFn } from "@angular/common/http";
import { inject } from "@angular/core";
import { first, switchMap } from "rxjs";
import { AuthService } from "./auth.service";

export const authInterceptor: HttpInterceptorFn = (req, next) => {
  // Get the instance of the AuthService
  const auth = inject(AuthService);

  // Get the bearer token (if any).
  return auth.getToken$().pipe(
    // first() will re-emit the first emitted value of the source Observable
    // (here, getToken$()), then complete. An Obseravble returned by an Interceptor
    // MUST complete at some point, otherwise the intercepted request will be forever hanging
    first(),
    switchMap((token) => {
      // Add it to the request if it doesn't already have an Authorization header.
      if (token && !req.headers.has("Authorization")) {
        req = req.clone({
          headers: req.headers.set("Authorization", `Bearer ${token}`),
        });
      }
      return next(req);
    })
  );
};
```

Now you need to register this interceptor in your application configuration. In `src/main.ts`, add:

```ts
// Other imports remain untouched
import { /* ... */, withInterceptors } from '@angular/common/http';
import { authInterceptor } from './app/security/auth.interceptor';

if (environment.production) {
  enableProdMode();
}

bootstrapApplication(AppComponent, {
  providers: [
    // Other providers remain untouched
    provideHttpClient(withInterceptors([ authInterceptor ])),
    // Other providers remain untouched
  ],
});
```

Now all your API calls will have the `Authorization` header when the user is logged in.

> You can verify this by looking at the XHR Network requests in the trip list tab and search for the `Authorization` header in the request's headers

<a href="#top">↑ Back to top</a>

## Multi-environment & sensitive configuration

Sometimes you might have to store values that should not be committed to version control:

- **Environment-specific** values that may change depending on where you deploy your app
- **Sensitive information** like access tokens or passwords.

For example, in our earlier HTTP calls, the URL was **hardcoded**:

```ts
const url = "https://demo-travel-log-api.onrender.com/api/trips";
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

The other one, `environment.prod.ts`, is the file that will contain production specific values. **It should not be commited** at all (especially when using a public repository), since it could contain **VERY** sensitive information.

Alas, both those files have already been commited when the project was set up... It's not a huge problem as both those files don't currently contain anything sensitive.

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

> The `environment.sample.ts` file is a **placeholder** and should **NOT** contain the actual configuration.

<a href="#top">↑ Back to top</a>

### Create the actual configuration file

With this placeholder file, you can now (re)create the actual `src/environments/environment.ts` configuration file (by copying and renaming the `environment.sample.ts` file), this time with your actual configuration values, at least for development:

```ts
export const environment = {
  production: false,
  apiUrl: "https://demo-travel-log-api.onrender.com/api",
};
```

While we're at it, let's also (re)create the `environment.prod.ts` file, used for production builds, with this content:

```ts
export const environment = {
  production: true,
  // That's the same api Url in our case, but in real project, it would certainly be different (you don't want to develop using the same instance as the production application...)
  apiUrl: "https://demo-travel-log-api.onrender.com/api",
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
export class TripListPage implements ViewWillEnter {
  // ...
  ionViewWillEnter(): void {
    const url = `${environment.apiUrl}/trips`;
    this.http.get(url).subscribe((trips) => {
      console.log(`Trips loaded`, trips);
    });
  }
  // ...
}
```

Do not forget to also update the authentication service in `src/app/security/auth.service.ts`, which also has a hardcoded URL:

```ts
// Other imports...
// TODO: import the environment config.
import { environment } from "src/environments/environment";

// TODO: Remove the API_URL constant

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

[angular-component]: https://angular.dev/guide/components
[angular-guard]: https://angular.dev/guide/routing/common-router-tasks#preventing-unauthorized-access
[cordova]: https://cordova.apache.org/
[forms-with-angular-and-ionic]: https://mediacomem.github.io/comem-devmobil/latest/subjects/angular-forms-ionic/#1
[ionic-cli]: https://ionicframework.com/docs/cli
[ionic-tabs]: https://ionicframework.com/docs/api/tabs
[lazy-loading]: https://angular.io/guide/lazy-loading-ngmodules
[rxjs-delay-when]: http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html#instance-method-delayWhen
[sass]: http://sass-lang.com
[travel-log-api]: https://demo-travel-log-api.onrender.com/
[storage]: https://github.com/ionic-team/ionic-storage
[sqlite]: https://sqlite.org
[indexed-db]: https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API
[websql]: https://www.w3.org/TR/webdatabase/
[local-storage]: https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage
[http-interceptor]: https://angular.dev/guide/http/interceptors
