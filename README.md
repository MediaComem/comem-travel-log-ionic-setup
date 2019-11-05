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
  - [Create a blank Ionic app and make it a repository](#create-a-blank-ionic-app-and-make-it-a-repository)
  - [Serve the application locally](#serve-the-application-locally)
- [Set up the navigation structure](#set-up-the-navigation-structure)
  - [Create the pages](#create-the-pages)
  - [Update the app to use the pages](#update-the-app-to-use-the-pages)
- [Set up security](#set-up-security)
  - [Check the documentation of the API's authentication resource](#check-the-documentation-of-the-apis-authentication-resource)
  - [Create model classes](#create-model-classes)
  - [Create an authentication service](#create-an-authentication-service)
  - [Create the login screen](#create-the-login-screen)
  - [Use the authentication service to protect access to the home page](#use-the-authentication-service-to-protect-access-to-the-home-page)
  - [Storing the authentication credentials](#storing-the-authentication-credentials)
  - [Log out](#log-out)
  - [Configuring an HTTP interceptor](#configuring-an-http-interceptor)
- [Multi-environment & sensitive configuration](#multi-environment--sensitive-configuration)
  - [Environment files](#environment-files)
  - [Create the actual configuration file](#create-the-actual-configuration-file)
  - [Add the environment files to your `.gitignore` file](#add-the-environment-files-to-your-gitignore-file)
  - [Configure angular to use environment files](#configure-angular-to-use-environment-files)
  - [Feed the configuration to Angular](#feed-the-configuration-to-angular)
- [Troubleshooting](#troubleshooting)
  - [Initializers are not allowed in ambient contexts](#initializers-are-not-allowed-in-ambient-contexts)
  - [`ionic serve` crashes with an `ECONNRESET` error when saving a file](#ionic-serve-crashes-with-an-econnreset-error-when-saving-a-file)
  - [Cordova doesn't want JDK 1.9](#cordova-doesnt-want-jdk-19)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Prerequisites

These instructions assume that you are using the Travel Log API based on one of the suggestions of the previous [Web-Oriented Architecture](https://github.com/MediaComem/comem-archioweb),
and that you are familiar with the [documentation of the reference API](https://comem-travel-log-api.herokuapp.com).

You will need to have [Node.js](https://nodejs.org) installed.
The latest LTS (Long Term Support) version is recommended (v12.13.0 at the time of writing these instructions).

<a href="#top">↑ Back to top</a>

## Features

This guide describes a proposed list of features and a user interface based on those features.
This is only a suggestion. You can support other features and make a different user interface.

The proposed app should users to do the following:

* Create new trips & places.
* See visited places on an interactive map.
* Browse the list of trips.

The following sections describe a proposed UI mockup of the app and steps to set up a skeleton implementation.

<a href="#top">↑ Back to top</a>

## Design the user interface

Before diving into the code, you should always take a moment to design the user interface of your app.
This doesn't have to be a final design, but it should at least be a sketch of what you want.
This helps you think in terms of the whole app and of the relationships between screens.

![UI Design](setup/ui-structure.png)

As you can see, we propose to use a tab view with 3 screens, and an additional 4th screen accessible from the trip list:

* The create trip tab.
* The places map tab.
* The trip list tab.
  * A trip details page.

Now that we know what we want, we can start setting up the app!

<a href="#top">↑ Back to top</a>

## Set up the application

### Create a blank Ionic app and make it a repository

Make sure you have Ionic and Cordova installed:

```bash
$> ionic -v
5.4.5
```
> If you have an error when running the above command, execute:
>  ```bash
>  $> npm install -g ionic@latest
>  ```

```
$> cordova -v
9.0.0
```
> If you have an error when running the above command, execute:
> ```bash
> $> npm install -g cordova@latest
> ```

Go in the directory where you want the app project to be located, then generate a blank Ionic app with the following command:

```bash
$> cd /path/to/projects
$> ionic start travel-log blank
```
When asked, to chose a **framework**, select **Angular**
```bash
Pick a framework!

Please select the JavaScript framework to use for your new app. To bypass this prompt next time, supply a value for the
--type option.

? Framework: (Use arrow keys)
> Angular | https://angular.io
  React   | https://reactjs.org
```
Then wait for the install to proceed... ⏳

Go into the app directory. The `ionic start` command should have already initialized a Git repository:

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

To make sure everything was set up correctly, use the following command from the repository to serve the application locally in your browser:

```bash
$> ionic serve
```

You should see something like this:

![Serving the blank app](setup/blank-app-serve.png)

<a href="#top">↑ Back to top</a>

## Set up the navigation structure

As defined in our UI design, we want the following 4 screens:

* The trip creation tab.
* The places map tab.
* The trip list tab.
  * The trip details screen.

Let's start by setting up the 3 tabs.
We will use Ionic's [Tabs][ionic-tabs] component.

<a href="#top">↑ Back to top</a>

### Create the pages

Each page will be an [Angular component][angular-component].
Ionic has a `generate` command that can automatically set up the files we need to create each page's component.

The existing `Home` page will contains the Tab layout, and each tab page is therefor a subpage of `Home`.

```bash
$> ionic generate page home/CreateTrip --module=home/home.module.ts
$> ionic generate page home/PlacesMap --module=home/home.module.ts
$> ionic generate page home/TripList --module=home/home.module.ts
```
> Not setting the `--module` param will add your new page to the main navigation array, which is not what we want.

> Use the up arrow key to get the last executed command. This will prevent you typing the same line three times

This will generate the following files:

```
src/app/home/create-trip/create-trip.module.ts
src/app/home/create-trip/create-trip.page.html
src/app/home/create-trip/create-trip.page.spec.ts
src/app/home/create-trip/create-trip.page.ts
src/app/home/create-trip/create-trip.page.scss
src/app/home/places-map/places-map.module.ts
src/app/home/places-map/places-map.page.html
src/app/home/places-map/places-map.page.spec.ts
src/app/home/places-map/places-map.page.ts
src/app/home/places-map/places-map.page.scss
src/app/home/trip-list/trip-list.module.ts
src/app/home/trip-list/trip-list.page.html
src/app/home/trip-list/trip-list.page.spec.ts
src/app/home/trip-list/trip-list.page.ts
src/app/home/trip-list/trip-list.page.scss
```

For each page, we have:

* A module declaration.
* An HTML template.
* A [Sass/SCSS][sass] stylesheet.
* An Angular component.
* A test suite with a default test.

Now update the HTML template for each page and add some content within the `<ion-content>` tag.
For example, in `src/app/home/create-trip/create-trip.page.html`:

```html
<ion-content class="ion-padding">
  Let's create a trip.
</ion-content>
```

### Update the app to use the pages

Now that the pages are ready, we need to display them.

First, **add the tab pages in the `Home` router** (`src/app/home/home.module.ts`) so that we will be able to navigate to them:

```ts
// ...imports omitted for brevity

// TODO: move the routes declaration outside the NgModule declaration
const routes: Routes = [
  {
    path: '', component: HomePage, children: [
      { path: '', redirectTo: 'map'}, // This defines the default tab
      {
        path: 'new',
        loadChildren: () => import('./create-trip/create-trip.module').then(m => m.CreateTripPageModule)
      },
      {
        path: 'map',
        loadChildren: () => import('./places-map/places-map.module').then(m => m.PlacesMapPageModule)
      },
      {
        path: 'list',
        loadChildren: () => import('./trip-list/trip-list.module').then(m => m.TripListPageModule)
      },
    ]
  }
];

@NgModule({
  imports: [
    CommonModule,
    FormsModule,
    IonicModule,
    RouterModule.forChild(routes) // TODO: configure the router with the above constant
  ],
  declarations: [ HomePage ]
})
export class HomePageModule { }
```
> Note how each page is not directly importted to the `Home` module ; instead, we're using **Lazy loading** to load each page's module (which in turn will load the proper component)

Second, **update the home page's component** (`src/app/home/home.page.ts`) to include the list of tabs we want:

```ts
import { Component } from '@angular/core';

// TODO: add an interface to represent a tab.
export interface HomePageTab {
  title: string; // The title of the tab in the tab bar
  icon: string; // The icon of the tab in the tab bar
  path: string; // The route's path that displays the tab
}

@Component({
  selector: 'app-home',
  templateUrl: 'home.page.html',
  styleUrls: ['home.page.scss'],
})
export class HomePage {

  tabs: HomePageTab[];

  constructor() {
    this.tabs = [
      { title: 'New Trip', icon: 'add', path: 'new' },
      { title: 'Places Map', icon: 'map', path: 'map'},
      { title: 'Trip List', icon: 'list', path: 'list'}
    ];
  }

}
```
> Be sure that the value of the tab's `path` property matches the corresponding route in the `home.module.ts` file.

Third, we will **replace the entire contents of the home page's template** (`src/app/home/home.page.html`) to use Ionic's [Tabs component][ionic-tabs].

Angular's `ngFor` directive allows us to iterate over the `tabs` array we declared in the home page's component,
and to put one `<ion-tab-button>` tag in the tab bar for each component:

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

## Set up security

To use the app, a user should identify themselves.
You will add a login screen that the user must go through before accessing the other screens.
Authentication will be performed by the [Travel Log API][travel-log-api].

The API requires a bearer token be sent to identify the user when making requests on some resources (e.g. when creating trips).
This token must be sent in the `Authorization` header for all requests requiring identification.
Once login/logout is implemented, you will also set up an HTTP interceptor to automatically add this header to every request.

### Check the documentation of the API's authentication resource

The Travel Log API provides an [`/auth` resource](https://comem-travel-log-api.herokuapp.com/#api-Authentication-CreateAuthenticationToken)
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

Create a `src/app/models/user.ts` file which exports a model representing a user of the API:

```ts
export class User {
  id: string;
  href: string;
  name: string;
  tripsCount: number;
  createdAt: string;
  updatedAt: string;
}
```

Create a `src/app/models/auth-request.ts` file which exports a model representing a request to the authentication resource:

```ts
export class AuthRequest {
  username: string;
  password: string;
}
```

Create a `src/app/models/auth-response.ts` file which exports a model representing a successful response from the authentication resource:

```ts
import { User } from './user';

export class AuthResponse {
  token: string;
  user: User;
}
```

### Create an authentication service

Let's generate a reusable, injectable service to manage authentication:

```bash
$> ionic generate service auth/Auth
```
Since our new service will make Http requests, we need to imports the `HttpClientModule` in our app. Do so in the `src/app/app.module.ts` file:

```ts
// ...Other imports
// TODO: import Angular's httpClientModule
import { HttpClientModule } from '@angular/common/http';

@NgModule({
  declarations: [ AppComponent ],
  entryComponents: [],
  imports: [
    /* ... */
    HttpClientModule // TODO: import Angular's httpClientModule
  ],
  providers: [/* ... */],
  bootstrap: [ AppComponent ]
})
export class AppModule { }
```

You can replace the contents of the generated `src/app/auth/auth.service.ts` file with the following code:

```ts
import { HttpClient } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { Observable, ReplaySubject } from 'rxjs';
import { map } from 'rxjs/operators';

import { AuthResponse } from '../models/auth-response';
import { User } from '../models/user';
import { AuthRequest } from '../models/auth-request';

/**
 * Authentication service for login/logout.
 */
@Injectable({ providedIn: 'root' })
export class AuthService {

  private auth$: Observable<AuthResponse>;
  private authSource: ReplaySubject<AuthResponse>;

  constructor(private http: HttpClient) {
    this.authSource = new ReplaySubject(1);
    this.authSource.next(undefined);
    this.auth$ = this.authSource.asObservable();
  }

  isAuthenticated(): Observable<boolean> {
    return this.auth$.pipe(map(auth => Boolean(auth)));
  }

  getUser(): Observable<User> {
    return this.auth$.pipe(map(auth => auth ? auth.user : undefined));
  }

  getToken(): Observable<string> {
    return this.auth$.pipe(map(auth => auth ? auth.token : undefined));
  }

  logIn(authRequest: AuthRequest): Observable<User> {

    const authUrl = 'https://comem-travel-log-api.herokuapp.com/api/auth';
    return this.http.post<AuthResponse>(authUrl, authRequest).pipe(
      map(auth => {
        this.authSource.next(auth);
        console.log(`User ${auth.user.name} logged in`);
        return auth.user;
      })
    );
  }

  logOut() {
    this.authSource.next(null);
    console.log('User logged out');
  }

}
```
### Create the login screen

Generate a login page component to be added to the main `app-routing.module.ts` file:

```bash
$> ionic generate page Login
```

Add the following HTML form inside the `<ion-content>` tag of `src/app/login/login.page.html`:

```html
<form #loginForm="ngForm" (submit)="onSubmit(loginForm)">
  <ion-list>

    <!-- Username input -->
    <ion-item>
      <ion-label position="floating">Username</ion-label>
      <ion-input inputmode="text" #username="ngModel" required="true" name="username"
        [(ngModel)]="authRequest.username"></ion-input>
    </ion-item>

    <!-- Error message displayed if the username is invalid -->
    <ion-item lines="none" *ngIf="username.invalid && username.touched">
      <ion-text color='danger'>Username is required.</ion-text>
    </ion-item>

    <!-- Password input -->
    <ion-item>
      <ion-label position="floating">Password</ion-label>
      <ion-input inputmode="text" #password="ngModel" required="true" type="password" name="password"
        [(ngModel)]="authRequest.password"></ion-input>
    </ion-item>

    <!-- Error message displayed if the password is invalid -->
    <ion-item lines="none" *ngIf="password.invalid && password.touched">
      <ion-text color='danger'>Password is required.</ion-text>
    </ion-item>

  </ion-list>

  <div class="ion-padding">

    <!-- Submit button -->
    <ion-button type="submit" expand="block" [disabled]="loginForm.invalid">Log in</ion-button>

    <!-- Error message displayed if the login failed -->
    <ion-text color='danger' *ngIf="loginError">Username or password is invalid.</ion-text>

  </div>
</form>
```

Update `src/app/login/login.page.ts` as follows:

```ts
import { Component } from '@angular/core';
import { NgForm } from '@angular/forms';
import { Router } from '@angular/router';

import { AuthService } from '../auth/auth.service';
import { AuthRequest } from '../models/auth-request';

/**
 * Login page.
 */
@Component({
  templateUrl: 'login.page.html'
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

  constructor(
    private auth: AuthService,
    private router: Router
  ) {
    this.authRequest = new AuthRequest();
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
    this.auth.logIn(this.authRequest).subscribe({
      next: () => {
        this.router.navigateByUrl('/home');
      },
      error: err => {
        this.loginError = true;
        console.warn(`Authentication failed: ${err.message}`);
      }
    });
  }
}

```

<a href="#top">↑ Back to top</a>

### Use the authentication service to protect access to the home page

Now that we have a service to manage authentication and a working page for users to log in, we need to make sure that unauthenticated user can not access restricted pages and are instead redirected to the login page.

We will use an [Angular Guard][angular-guard] to do this.

```bash
$> ionic generate guard auth/Auth
```
> When asked what interface you'd like to implement, select only the `CanActivate` interface.

Open the `src/app/auth/auth.guard.ts` file and **replace all its content** with:

```ts
import { Injectable } from '@angular/core';
import { CanActivate, Router, UrlTree } from '@angular/router';
import { Observable } from 'rxjs';
import { AuthService } from './auth.service';
import { map } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export class AuthGuard implements CanActivate {

  constructor(
    private auth: AuthService,
    private router: Router
  ) { }

  canActivate(): Observable<boolean | UrlTree> {
    return this.auth.isAuthenticated().pipe(map(isAuthenticated => {
      return isAuthenticated ? true : this.router.parseUrl('/login');
    }));
  }

}
```

To use this guard, open the `src/app/app-routing.module.ts` file and add a new `canActivate` property to the `'home'` route:

```ts
// ...Other imports
// TODO: import the AuthGuard
import { AuthGuard } from './auth/auth.guard';

const routes: Routes = [
  { path: '', redirectTo: 'home', pathMatch: 'full' },
  {
    path: 'home',
    loadChildren: () => import('./home/home.module').then(m => m.HomePageModule),
    // TODO: Add the guard to the canActivate array of this route
    canActivate: [ AuthGuard ]
  },
  {
    path: 'login',
    loadChildren: () => import('./login/login.module').then(m => m.LoginPageModule)
  },
];

@NgModule({
  imports: [/* ... */],
  exports: [ RouterModule ]
})
export class AppRoutingModule { }
```

The login screen is ready!
If you reload your app, you should see that you are automatically redirected to the login page.

You can now log in.
You should be able to use the username `jdoe` and the password `test` with the Travel Log API's standard dataset.

<a href="#top">↑ Back to top</a>

### Storing the authentication credentials

Now you can log in, but there's a little problem.

Every time the app is reloaded, you lose all data so you have to log back in.
This is particularly annoying for local development since the browser is automatically refreshed every time you change the code.

You need to use more persistent storage for the security credentials, i.e. the authentication token.
Ionic provides a [storage module](https://ionicframework.com/docs/storage/) which will automatically select an appropriate storage method for your platform.
It will use [SQLite](https://sqlite.org) on phones when available; for web platforms it will use [IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API), [WebSQL](https://www.w3.org/TR/webdatabase/) or [Local Storage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage).

To use the Ionic storage module, you must first install it:

```bash
$> npm i @ionic/storage
```
Then import it into your application's module in `src/app/app.module.ts`:

```ts
// Other imports...
// TODO: import the ionic storage module.
import { IonicStorageModule } from '@ionic/storage';

@NgModule({
  // ...
  imports: [
    // ...
    // TODO: import the ionic storage module into the app's module.
    IonicStorageModule.forRoot()
  ],
  // ...
})
export class AppModule {}
```

Now you can import the `Storage` provider in `AuthService` in `src/app/auth/auth.service.ts`:

```ts
// Other imports...
// TODO: import RxJS's from function, delayWhen operator and Ionic's storage provider.
import { Observable, ReplaySubject, from } from 'rxjs';
import { delayWhen, map } from 'rxjs/operators';
import { Storage } from '@ionic/storage';
```

You also need to inject it into the constructor:

```ts
constructor(private http: HttpClient, private storage: Storage)
```

Add a method to persist the authentication information using the storage module:

```ts
private saveAuth(auth: AuthResponse): Observable<void> {
  return from(this.storage.set('auth', auth));
}
```

The storage module returns Promises, but we'll be plugging this new function into `logIn()` which uses Observables,
so we convert the Promise to an Observable before returning it.

You can now update the `logIn()` method to persist the API's authentication response with the new `saveAuth()` method.
To do that, use RxJS's [`delayWhen`][rxjs-delay-when] operator, which allows us to delay an Observable stream (in this case, the one that indicates our user is authenticated) until another Observable emits
(in this case, the one that saves the authentication response):

```ts
logIn(authRequest: AuthRequest): Observable<User> {

  const authUrl = 'https://comem-travel-log-api.herokuapp.com/api/auth';
  return this.http.post<AuthResponse>(authUrl, authRequest).pipe(
    // TODO: delay the observable stream while persisting the authentication response.
    delayWhen(auth => {
      return this.saveAuth(auth);
    }),
    map(auth => {
      this.authSource.next(auth);
      console.log(`User ${auth.user.name} logged in`);
      return auth.user;
    })
  );
}
```

When testing in the browser, you should already see the object being stored in IndexedDB (the default storage if using Chrome).

You must now load it when the app starts.
You can do that in the constructor of `AuthService`.

Since the storage provider's `get` method returns a promise,
you can only use the result in a `.then` asynchronous callback:

```ts
constructor(private http: HttpClient, private storage: Storage) {

  this.authSource = new ReplaySubject(1);
  this.auth$ = this.authSource.asObservable();

  // TODO: load the stored authentication response from storage when the app starts.
  this.storage.get('auth').then(auth => {
    // Push the loaded value into the observable stream.
    this.authSource.next(auth);
  });
}
```

Your app should now remember user credentials even when you reload it!

Finally, also update the authentication provider's `logOut()` method to remove the stored authentication from storage:

```ts
logOut() {
  this.authSource.next(null);
  // TODO: remove the stored authentication response from storage when logging out.
  this.storage.remove('auth');
  console.log('User logged out');
}
```

<a href="#top">↑ Back to top</a>

### Log out

You should also add a UI component to allow the user to log out.
As an example, you will display a logout button in the trip creation screen.

Add an `<ion-buttons>` tag with a logout button in `src/app/home/create-trip/create-trip.page.html`:

```html
<ion-header>
  <ion-toolbar>
    <ion-title>CreateTrip</ion-title>

    <!-- Logout button -->
    <ion-buttons slot="end">
      <ion-button (click)="logOut()">
        <ion-icon name='log-out'></ion-icon>
      </ion-button>
    </ion-buttons>
  </ion-toolbar>
</ion-header>
```

Let's assume that when logging out, we want the user redirected to the login page,
and we want the navigation stack to be cleared (so that pressing the back button doesn't bring the user back to a protected screen).

To do that, you will need:

* To inject the Angular Router, which will allow you to navigate to a defined     route.
* To add a `logOut` method to the `CreateTripPage` component,
  since it's what we call in its HTML template above.
* To inject `AuthService` and use its `logOut` method.

After doing all that, your `CreateTripPage` component should look something like this:

```ts
// Other imports...
// TODO: import the authentication provider and login page.
import { AuthService } from '../../providers/auth/auth';
import { LoginPage } from '../login/login';

@Component({
  selector: 'page-create-trip',
  templateUrl: 'create-trip.html'
})
export class CreateTripPage {

  constructor(
    // TODO: inject the authentication provider.
    private auth: AuthService,
    public navCtrl: NavController,
    public navParams: NavParams
  ) {
  }

  // TODO: add a method to log out.
  logOut() {
    this.auth.logOut();
    this.router.navigateByUrl('/login');
  }

}
```

You should now see the logout button in the navigation bar after logging in.

You might want to encapsulate it into a reusable component later, to include in other screens.

<a href="#top">↑ Back to top</a>

### Configuring an HTTP interceptor

Now that you have login and logout functionality, and an authentication service that stores an authentication token, you can authenticate for other API calls.

Looking at the API documentation, at some point you will need to [create a trip](https://comem-travel-log-api.herokuapp.com/#api-Trips-CreateTrip).
The documentation states that you must send a bearer token in the `Authorization` header, like this:

```
POST /api/trips HTTP/1.1
Authorization: Bearer 0a98wumv
Content-Type: application/json

{"some":"json"}
```

With Angular, you would make this call like this:

```js
httpClient.post('http://example.com/path', body, {
  headers: {
    Authorization: `Bearer ${token}`
  }
});
```

But it's a bit annoying to have to specify this header for every request.
After all, we know that we need it for most calls.

[Interceptors](https://medium.com/@ryanchenkie_40935/angular-authentication-using-the-http-client-and-http-interceptors-2f9d1540eb8)
are Angular services that can be registered with the HTTP client to automatically react to requests (or responses).
This solves our problem: we want to register an interceptor that will automatically add the `Authorization` header to all requests if the user is logged in.

To demonstrate that it works, start by adding a call to list trips in the `TripListPage` component in `src/pages/trip-list/trip-list.ts`:

```ts
// Other imports...
// TODO: import Angular's HTTP client.
import { HttpClient } from '@angular/common/http';

@Component({
  selector: 'page-trip-list',
  templateUrl: 'trip-list.html'
})
export class TripListPage {

  constructor(
    private auth: AuthService,
    // TODO: inject the HTTP client.
    public http: HttpClient,
    public navCtrl: NavController,
    public navParams: NavParams
  ) {
  }

  ionViewDidLoad() {
    // TODO: make an HTTP request to retrieve the trips.
    const url = 'https://comem-travel-log-api.herokuapp.com/api/trips';
    this.http.get(url).subscribe(trips => {
      console.log(`Trips loaded`, trips);
    });
  }

  // ...

}
```

If you display the trip list page and check network requests in Chrome's developer tools,
you will see that there is no `Authorization` header sent even when the user is logged in.

Now you can generate the interceptor service:

```bash
$> ionic generate service auth/AuthInterceptor
```

Put the following contents in the generated `src/app/auth/auth-interceptor.service.ts` file:

```ts
import { HttpRequest, HttpHandler, HttpEvent, HttpInterceptor } from '@angular/common/http';
import { Injectable, Injector } from '@angular/core';
import { Observable } from 'rxjs';
import { first, switchMap } from 'rxjs/operators';

import { AuthService } from './auth.service';

@Injectable({ providedIn: 'root' })
export class AuthInterceptorProvider implements HttpInterceptor {

  constructor(private injector: Injector) { }

  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {

    // Retrieve AuthService at runtime from the injector.
    // (Otherwise there would be a circular dependency:
    //  AuthInterceptorProvider -> AuthService -> HttpClient -> AuthInterceptorProvider).
    const auth = this.injector.get(AuthService);

    // Get the bearer token (if any).
    return auth.getToken().pipe(
      first(),
      switchMap(token => {

        // Add it to the request if it doesn't already have an Authorization header.
        if (token && !req.headers.has('Authorization')) {
          req = req.clone({
            headers: req.headers.set('Authorization', `Bearer ${token}`)
          });
        }

        return next.handle(req);
      })
    );
  }

}
```

Now you simply need to register the interceptor in your application module.
Since it's an HTTP interceptor, it's not like other providers and must be registered in a special way.
In `src/app/app.module.ts`, add:

```ts
// Other imports...
import { AuthInterceptorProvider } from './auth/auth-interceptor.service';

@NgModule({
  // ...
  providers: [
    // Other providers...
    { provide: HTTP_INTERCEPTORS, useClass: AuthInterceptorProvider, multi: true }
  ]
})
export class AppModule {}
```

The `multi: true` option is necessary because you can register multiple interceptors if you want
(read more about [multi providers](https://blog.thoughtram.io/angular2/2015/11/23/multi-providers-in-angular-2.html)).

Now all your API calls will have the `Authorization` header when the user is logged in.

<a href="#top">↑ Back to top</a>

## Multi-environment & sensitive configuration

Sometimes you might have to store values that should not be committed to version control:

* **Environment-specific** values that may change depending on how you deploy your app (e.g. the URL of your API).
* **Sensitive information** like access tokens or passwords.

For example, in our earlier HTTP calls, the URL was **hardcoded**:

```ts
const url = 'https://comem-travel-log-api.herokuapp.com/api/trips';
this.http.get(url).subscribe(trips => {
  // ...
});
```

This is not optimal considering the multi-environment problem.
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

You need to tell git to untrack them, though, so delete both of them from your filesystem (we'll recreate them later), then commit those deletions in git:

```bash
$> rm src/environments/*
$> git add src/environments/*
$> git commit -m "Remove environment files from git"
```

Now, create a placeholder file whose purpose will be to explain to other developers of the project what their own environment file should contains so they could fill in the actual values.
Create the `environment.sample.ts` file in `src/environments`, with this content:

```ts
// Copy this file to environment.ts and fill in appropriate values.
export const environment = {
  production: false,
  apiUrl: 'https://example.com/api'
};
```

> The `environment.sample.ts` file is a placeholder and should **NOT** contain the actual configuration.

<a href="#top">↑ Back to top</a>

### Create the actual configuration file

With this placeholder file, you can now create the actual `src/environments/environment.ts` configuration file (by copying the `environment.sample.ts` file), this time containing the actual configuration values, at least for development:

```ts
export const environment = {
  production: false,
  apiUrl: 'https://comem-travel-log-api.herokuapp.com/api'
}
```

While we're at it, let's also create the `environment.prod.ts` file, used for production builds, with this content:

```ts
export const environment = {
  production: true,
  // That's the same api Url in our case, but in real project, it would certainly be different
  apiUrl: 'https://comem-travel-log-api.herokuapp.com/api'
}
```

<a href="#top">↑ Back to top</a>

### Add the environment files to your `.gitignore` file

Of course, you **don't want to commit neither `environment.ts` nor `environment.prod.ts`**, but you do want to commit `environment.sample.ts`
so that anyone who clones your project can see what configuration options are required.
Add these lines at the bottom of your `.gitignore` file:

```
# Environment files
src/environments/*
!src/environments/environment.sample.ts
```

The first line tells git to not track any file in the `src/environments` folder... except for the specific `environment.sample.ts` file (this is the second line).

You now have your uncommitted environment files!

### When are the environment files used

The `environment.ts` file is loaded when you execute the `ionic serve` command.

> If you have any `ionic serve` command running, kill it and start it again to take the changes into account.

When executing the same command with the `--prod` flag:

```bash
$> ionic serve --prod
```
Ionic replaces the content of `environment.ts` file by the content from `environment.prod.ts`, in the build (**it does not replace the content of the actual file, thankfully**).

This way, whatever the environment your app is running on, `environment.ts` is **always the file holding the adequate configuration!**

<a href="#top">↑ Back to top</a>

### Feed the configuration to Angular

Now that you have your configuration files, you want to use its values in code.

Since it's a TypeScript file like any other, you simply have to import and use it.

> **Remember that you only ever have to import `environment.ts` in your code**, as it's content will change depending on the environment.

```ts
// Other imports...
// TODO: import the environment config.
import { environment } from 'src/environments/environment';

// ...
export class TripListPage {
  // ...
  ngOnInit() {
    const url = `${environment.apiUrl}/trips`;
    this.http.get(url).subscribe(trips => {
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
import { environment } from 'src/environments/environment';

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

<a href="#top">↑ Back to top</a>

## Troubleshooting

### Initializers are not allowed in ambient contexts

You might see the following error in your console or browser:

```
[app-scripts] [16:01:24]  typescript: node_modules/@asymmetrik/ngx-leaflet/dist/leaflet/core/leaflet.directive.d.ts, line: 6
[app-scripts]             Initializers are not allowed in ambient contexts.
```

This is due to the fact that at the time of writing, the latest version of the `ngx-leaflet` library is based on version 7 of Angular,
while Ionic uses version 5 of Angular. The `ngx-leaflet` library depends on a new TypeScript feature available in Angular 7 but not in Angular 5, causing this error.

You can solve the problem by downgrading the `ngx-leaflet` library to the previous version in `package.json`:

```json
  ...
  "dependencies": {
    ...
    "@asymmetrik/ngx-leaflet": "^4.0.0",
    ...
  },
  ...
```

Then run `npm install` in the project's directory to install that older version.

### `ionic serve` crashes with an `ECONNRESET` error when saving a file

The output of `ionic serve` after saving a file:

```
[10:43:01]  build started ...
[10:43:01]  deeplinks update started ...
[10:43:01]  deeplinks update finished in 2 ms
[10:43:01]  transpile started ...
[10:43:02]  transpile finished in 609 ms
[10:43:02]  webpack update started ...
[10:43:05]  webpack update finished in 3.17 s
[10:43:05]  sass update started ...
[10:43:06]  sass update finished in 682 ms
[10:43:06]  build finished in 4.46 s

events.js:183
      throw er; // Unhandled 'error' event
      ^

Error: read ECONNRESET
    at _errnoException (util.js:1024:11)
    at TCP.onread (net.js:615:25)
```

This is due to a deprecation in the Web Sockets library.
As a workaround until this is fixed in Ionic, downgrade the library:

```
npm i -D -E ws@3.3.2
```

Reference:

* https://forum.ionicframework.com/t/ionic-serve-crash-on-save/115615/39
* https://github.com/ionic-team/ionic-app-scripts/issues/1345

### Cordova doesn't want JDK 1.9

```
$> ionic cordova run android
Running app-scripts build: --platform android --target cordova
[10:14:06]  build dev started ...
[10:14:06]  clean started ...
[10:14:06]  clean finished in 2 ms
[10:14:06]  copy started ...
[10:14:06]  deeplinks started ...
[10:14:06]  deeplinks finished in 36 ms
[10:14:06]  transpile started ...
[10:14:08]  transpile finished in 2.39 s
[10:14:08]  preprocess started ...
[10:14:08]  preprocess finished in less than 1 ms
[10:14:08]  webpack started ...
[10:14:08]  copy finished in 2.53 s
[10:14:13]  webpack finished in 4.51 s
[10:14:13]  sass started ...
[10:14:13]  sass finished in 709 ms
[10:14:13]  postprocess started ...
[10:14:13]  postprocess finished in 4 ms
[10:14:13]  lint started ...
[10:14:13]  build dev finished in 7.73 s
> cordova run android

You have been opted out of telemetry. To change this, run: cordova telemetry on.
Android Studio project detected

ANDROID_HOME=/Users/jdoe/Library/Android/sdk
JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-9.jdk/Contents/Home
(node:6282) UnhandledPromiseRejectionWarning: Unhandled promise rejection (rejection id: 1): CordovaError: Requirements check failed for JDK 1.8 or greater
(node:6282) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code.

[OK] Your app has been deployed.
     Did you know you can live-reload changes from your app with --livereload?

[10:14:15]  lint finished in 1.98 s
```

At the time of writing these instructions, Android and Cordova do not support version 9 of the JDK.

Install the JDK 8, find its full path, and put it first in your path when running `ionic` or `cordova`:

```
# For example, on macOS
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_161.jdk/Contents/Home
```

Alternatively, add that line to your `.bash_profile` to do it automatically.

Reference:

* https://forum.ionicframework.com/t/error-requirements-check-failed-for-jdk-1-8-or-greater/68734
* https://issues.apache.org/jira/browse/CB-13606



[angular-component]: https://angular.io/guide/architecture#components
[ionic-tabs]: https://ionicframework.com/docs/components/#tabs
[rxjs-delay-when]: http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html#instance-method-delayWhen
[sass]: http://sass-lang.com
[travel-log-api]: https://comem-travel-log-api.herokuapp.com
