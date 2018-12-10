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
  - [Authentication observable magic](#authentication-observable-magic)
  - [Configuring an HTTP interceptor](#configuring-an-http-interceptor)
- [Multi-environment & sensitive configuration](#multi-environment--sensitive-configuration)
  - [Create a sample configuration file](#create-a-sample-configuration-file)
  - [Create the actual configuration file](#create-the-actual-configuration-file)
  - [Add the configuration file to your `.gitignore` file](#add-the-configuration-file-to-your-gitignore-file)
  - [Feed the configuration to Angular](#feed-the-configuration-to-angular)
- [Troubleshooting](#troubleshooting)
  - [`ionic serve` crashes with an `ECONNRESET` error when saving a file](#ionic-serve-crashes-with-an-econnreset-error-when-saving-a-file)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->




## Prerequisites

These instructions assume that you are using the Travel Log API based on one of the suggestions of the previous [Web Services course](https://github.com/MediaComem/comem-webserv),
and that you are familiar with the [documentation of the reference API](https://comem-travel-log-api.herokuapp.com).

You will need to have [Node.js](https://nodejs.org) installed.
The latest LTS (Long Term Support) version is recommended (v8.9.4 at the time of writing these instructions).

<a href="#top">Back to top</a>



## Features

This guide describes a proposed list of features and a user interface based on those features.
This is only a suggestion. You can support other features and make a different user interface.

The proposed app should users to do the following:

* Create new trips & places.
* See visited places on an interactive map.
* Browse the list of trips.

The following sections describe a proposed UI mockup of the app and steps to set up a skeleton implementation.

<a href="#top">Back to top</a>



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

<a href="#top">Back to top</a>



## Set up the application



### Create a blank Ionic app and make it a repository

Make sure you have Ionic and Cordova installed:

```bash
$> npm install -g ionic cordova
```

Go in the directory where you want the app, then generate a blank Ionic app with the following command:

```bash
$> cd /path/to/projects
$> ionic start travel-log blank

? Would you like to integrate your new app with Cordova to target native iOS and Android? Yes
? Install the free Ionic Pro SDK and connect your app? No
```

Go into the app directory. The `ionic start` command should have already initialized a Git repository:

```bash
$> cd travel-log
$> git log
commit 2a3f83f14ae2a82d00cb2b2960dda1c1e0b0a432 (HEAD -> master)
Author: John Doe <john.doe@example.com>
Date:   Mon Dec 18 10:00:01 2017 +0100

    Initial commit
```

<a href="#top">Back to top</a>



### Serve the application locally

To make sure everything was set up correctly, use the following command from the repository to serve the application locally in your browser:

```bash
$> ionic serve
```

You should see something like this:

![Serving the blank app](setup/blank-app-serve.png)

<a href="#top">Back to top</a>



## Set up the navigation structure

As defined in our UI design, we want the following 4 screens:

* The trip creation tab.
* The places map tab.
* The trip list tab.
  * The trip details screen.

Let's start by setting up the 3 tabs.
We will use Ionic's [Tabs][ionic-tabs] component.

<a href="#top">Back to top</a>



### Create the pages

Each page will be an [Angular component][angular-component].
Ionic has a `generate` command that can automatically set up the files we need to create each page's component:

```bash
$> ionic generate page --no-module CreateTrip
$> ionic generate page --no-module PlacesMap
$> ionic generate page --no-module TripList
```

This will generate the following files:

```
src/pages/create-trip/create-trip.html
src/pages/create-trip/create-trip.scss
src/pages/create-trip/create-trip.ts
src/pages/places-map/places-map.html
src/pages/places-map/places-map.scss
src/pages/places-map/places-map.ts
src/pages/trip-list/trip-list.html
src/pages/trip-list/trip-list.scss
src/pages/trip-list/trip-list.ts
```

For each page, we have:

* An HTML template.
* A [Sass/SCSS][sass] stylesheet.
* An Angular component.

Now update the HTML template for each page and add some content within the `<ion-content>` tag.
For example, in `src/pages/create-trip/create-trip.html`:

```html
<ion-content padding>
  Let's create a trip.
</ion-content>
```



### Update the app to use the pages

Now that the pages are ready, we need to display them.

First, Angular and Ionic need to be aware of your new components.
You must **declare** them in 2 places in your main Angular module in `src/app/app.module.ts`:

* Add them to the `declarations` array to register the components with Angular.
* Add them to the `entryComponents` array so that Ionic is able to inject them dynamically (e.g. when switching tabs).

```js
// Other imports...
// TODO: import the new components.
import { CreateTripPage } from '../pages/create-trip/create-trip';
import { TripListPage } from '../pages/trip-list/trip-list';
import { PlacesMapPage } from '../pages/places-map/places-map';

@NgModule({
  declarations: [
    MyApp,
    HomePage,
    CreateTripPage, // TODO: add the components to "declarations".
    TripListPage,
    PlacesMapPage
  ],
  imports: [ /* ... */ ],
  bootstrap: [ /* ... */ ],
  entryComponents: [
    MyApp,
    HomePage,
    CreateTripPage, // TODO: add the components to "entryComponents".
    TripListPage,
    PlacesMapPage
  ],
  providers: [ /* ... */ ]
})
export class AppModule {}
```

Second, **update the home page's component** (`src/pages/home/home.ts`) to include the list of tabs we want:

```js
// Other imports...
// TODO: import the new components.
import { CreateTripPage } from '../create-trip/create-trip';
import { PlacesMapPage } from '../places-map/places-map';
import { TripListPage } from '../trip-list/trip-list';

// TODO: add an interface to represent a tab.
export interface HomePageTab {
  title: string;
  icon: string;
  component: Function;
}

@Component({
  selector: 'page-home',
  templateUrl: 'home.html'
})
export class HomePage {

  // TODO: declare a list of tabs to the component.
  tabs: HomePageTab[];

  constructor(public navCtrl: NavController) {
    // TODO: define some tabs.
    this.tabs = [
      { title: 'New Trip', icon: 'add', component: CreateTripPage },
      { title: 'Places Map', icon: 'map', component: PlacesMapPage },
      { title: 'Trip List', icon: 'list', component: TripListPage }
    ];
  }

}
```

Third, we will **replace the entire contents of the home page's template** (`src/pages/home/home.html`) to use Ionic's [Tabs component][ionic-tabs].

Angular's `ngFor` directive allows us to iterate over the `tabs` array we declared in the home page's component,
and to put one `<ion-tab>` tag in the page for each component:

```html
<ion-tabs>
  <ion-tab *ngFor='let tab of tabs'
           [tabTitle]='tab.title' [tabIcon]='tab.icon' [root]='tab.component'>
  </ion-tab>
</ion-tabs>
```

You should now be able to navigate between the 3 tabs!



## Set up security

To use the app, a user should identify him- or herself.
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

<a href="#top">Back to top</a>



### Create model classes

Let's create a few classes to use as models when communicating with the API.
That way we will benefit from TypeScript's typing when accessing model properties.

Create a `src/models/user.ts` file which exports a model representing a user of the API:

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

Create a `src/models/auth-request.ts` file which exports a model representing a request to the authentication resource:

```ts
export class AuthRequest {
  username: string;
  password: string;
}
```

Create a `src/models/auth-response.ts` file which exports a model representing a successful response from the authentication resource:

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
$> ionic generate provider Auth
```

Register the new provider in the module's `providers` array in `src/app/app.module.ts`.
You also need to add Angular's `HttpClientModule` to the module's `imports` array because the provider uses `HttpClient`:

```ts
// Other imports...
// TODO: import Angular's HttpClientModule and the new provider.
import { HttpClientModule } from '@angular/common/http';
import { AuthProvider } from '../providers/auth/auth';

@NgModule({
  declarations: [ /* ... */ ],
  imports: [
    // ...
    // TODO: import Angular's HttpClientModule.
    HttpClientModule
  ],
  bootstrap: [ /* ... */ ],
  entryComponents: [ /* ... */],
  providers: [
    // ...
    // TODO: register the new provider.
    AuthProvider
  ]
})
export class AppModule {}
```

You can replace the contents of the generated `srv/providers/auth/auth.ts` file with the following code:

```ts
import { HttpClient } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { Response } from '@angular/http';
import { Observable, ReplaySubject } from 'rxjs/Rx';
import { map } from 'rxjs/operators';

import { AuthRequest } from '../../models/auth-request';
import { AuthResponse } from '../../models/auth-response';
import { User } from '../../models/user';

/**
 * Authentication service for login/logout.
 */
@Injectable()
export class AuthProvider {

  private auth$: Observable<AuthResponse>;
  private authSource: ReplaySubject<AuthResponse>;

  constructor(private http: HttpClient) {
    this.authSource = new ReplaySubject(1);
    this.authSource.next(undefined);
    this.auth$ = this.authSource.asObservable();
  }

  isAuthenticated(): Observable<boolean> {
    return this.auth$.pipe(map(auth => !!auth));
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

Generate a login page component:

```bash
$> ionic generate page --no-module Login
```

Add this new component to the module's `declarations` and `entryComponents` arrays in `src/app/app.module.ts`:

```ts
// Other imports...
// TODO: import the new component.
import { LoginPage } from '../pages/login/login';

@NgModule({
  declarations: [
    // ...
    // TODO: add the component to the declarations.
    LoginPage
  ],
  imports: [ /* ... */ ],
  bootstrap: [ /* ... */ ],
  entryComponents: [
    // ...
    // TODO: add the component to the entry components.
    LoginPage
  ],
  providers: [ /* ... */ ]
})
export class AppModule {}
```

Add the following HTML form inside the `<ion-content>` tag of `src/pages/login/login.html`:

```html
<form (submit)='onSubmit($event)'>
  <ion-list>

    <!-- Username input -->
    <ion-item>
      <ion-label floating>Username</ion-label>
      <ion-input type='text' name='username'
                 #usernameInput='ngModel' [(ngModel)]='authRequest.username' required></ion-input>
    </ion-item>

    <!-- Error message displayed if the username is invalid -->
    <ion-item *ngIf='usernameInput.invalid && usernameInput.dirty' no-lines>
      <p ion-text color='danger'>Username is required.</p>
    </ion-item>

    <!-- Password input -->
    <ion-item>
      <ion-label floating>Password</ion-label>
      <ion-input type='password' name='password'
                 #passwordInput='ngModel' [(ngModel)]='authRequest.password' required></ion-input>
    </ion-item>

    <!-- Error message displayed if the password is invalid -->
    <ion-item *ngIf='passwordInput.invalid && passwordInput.dirty' no-lines>
      <p ion-text color='danger'>Password is required.</p>
    </ion-item>

  </ion-list>

  <div padding>

    <!-- Submit button -->
    <button type='submit' [disabled]='form.invalid' ion-button block>Log in</button>

    <!-- Error message displayed if the login failed -->
    <p *ngIf='loginError' ion-text color='danger'>Username or password is invalid.</p>

  </div>
</form>
```

Update `src/pages/login/login.ts` as follows:

```ts
import { Component, ViewChild } from '@angular/core';
import { NgForm } from '@angular/forms';
import { NavController, NavParams } from 'ionic-angular';

import { AuthRequest } from '../../models/auth-request';
import { AuthProvider } from '../../providers/auth/auth';
import { HomePage } from '../home/home';

/**
 * Login page.
 *
 * See https://ionicframework.com/docs/components/#navigation for more info on
 * Ionic pages and navigation.
 */
@Component({
  templateUrl: 'login.html'
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

  /**
   * The login form.
   */
  @ViewChild(NgForm)
  form: NgForm;

  constructor(private auth: AuthProvider, private navCtrl: NavController) {
    this.authRequest = new AuthRequest();
  }

  /**
   * Called when the login form is submitted.
   */
  onSubmit($event) {

    // Prevent default HTML form behavior.
    $event.preventDefault();

    // Do not do anything if the form is invalid.
    if (this.form.invalid) {
      return;
    }

    // Hide any previous login error.
    this.loginError = false;

    // Perform the authentication request to the API.
    this.auth.logIn(this.authRequest).subscribe(undefined, err => {
      this.loginError = true;
      console.warn(`Authentication failed: ${err.message}`);
    });
  }
}
```

<a href="#top">Back to top</a>



### Use the authentication service to protect access to the home page

Add the following imports to `src/app/app.component.ts`:

```ts
import { LoginPage } from '../pages/login/login';
import { AuthProvider } from '../providers/auth/auth';
```

Make sure that it doesn't have a default home page any more:

```ts
// ...
export class MyApp {
  rootPage: any;
  // ...
}
```

Update the component's constructor as follows:

```ts
constructor(
  // TODO: inject the authentication provider.
  private auth: AuthProvider,
  platform: Platform,
  statusBar: StatusBar,
  splashScreen: SplashScreen
) {

  // TODO: redirect the user to the login page if not authenticated.
  // Direct the user to the correct page depending on whether he or she is logged in.
  this.auth.isAuthenticated().subscribe(authenticated => {
    if (authenticated) {
      this.rootPage = HomePage;
    } else {
      this.rootPage = LoginPage;
    }
  });

  // ...
}
```

The login screen is ready!
If you reload your app, you should see that you are automatically redirected to the login page.

You can now log in.
You should be able to use the username `jdoe` and the password `test` with the Travel Log API's standard dataset.

<a href="#top">Back to top</a>



### Storing the authentication credentials

Now you can log in, but there's a little problem.
Every time the app is reloaded, you lose all data so you have to log back in.
This is particularly annoying for local development since the browser is automatically refreshed every time you change the code.

You need to use more persistent storage for the security credentials, i.e. the authentication token.
Ionic provides a [storage module](https://ionicframework.com/docs/storage/) which will automatically select an appropriate storage method for your platform.
It will use [SQLite](https://sqlite.org) on phones when available; for web platforms it will use [IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API), [WebSQL](https://www.w3.org/TR/webdatabase/) or [Local Storage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage).

To use the Ionic storage module, you must import it into your application's module in `src/app/app.module.ts`:

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

Now you can import the `Storage` provider in `AuthProvider` in `src/providers/auth/auth.ts`:

```ts
// Other imports...
// TODO: import RxJS's delayWhen operator and Ionic's storage provider.
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
  return Observable.fromPromise(this.storage.set('auth', auth));
}
```

The storage module returns Promises, but we'll be plugging this new function into `logIn()` which uses Observables,
so we convert the Promise to an Observable before returning it.

You can now update the `logIn()` method to persist the API's authentication response with the new `saveAuth()` method.
To do that, use RxJS's [`delayWhen`][rxjs-delay-when] operator, which allows us to delay an Observable stream until another Observable emits
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
You can do that in the constructor of `AuthProvider`.

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

<a href="#top">Back to top</a>



### Log out

You should also add a UI component to allow the user to log out.
As an example, you will display a logout button in the trip creation screen.

Add an `<ion-buttons>` tag with a logout button in `src/pages/create-trip/create-trip.html`:

```html
<ion-navbar>
  <ion-title>CreateTrip</ion-title>

  <!-- Logout button -->
  <ion-buttons end>
    <button ion-button icon-only (click)='logOut()'>
      <ion-icon name='log-out'></ion-icon>
    </button>
  </ion-buttons>
</ion-navbar>
```

Let's assume that when logging out, we want the user redirected to the login page,
and we want the navigation stack to be cleared (so that pressing the back button doesn't bring the user back to a protected screen).

To do that, you will need:

* To inject the Ionic application (`App` from the `ionic-angular` package),
  which will allow you to set the root page and thereby clear the navigation stack.
* To add a `logOut` method to the `CreateTripPage` component,
  since it's what we call in its HTML template above.
* To inject `AuthProvider` and use its `logOut` method.

After doing all that, your `CreateTripPage` component should look something like this:

```ts
// Other imports...
// TODO: import the authentication provider and login page.
import { AuthProvider } from '../../providers/auth/auth';
import { LoginPage } from '../login/login';

@Component({
  selector: 'page-create-trip',
  templateUrl: 'create-trip.html'
})
export class CreateTripPage {

  constructor(
    // TODO: inject the authentication provider.
    private auth: AuthProvider,
    public navCtrl: NavController,
    public navParams: NavParams
  ) {
  }

  // TODO: add a method to log out.
  logOut() {
    this.auth.logOut();
  }

}
```

You should now see the logout button in the navigation bar after logging in.

You might want to encapsulate it into a reusable component later, to include in other screens.

<a href="#top">Back to top</a>



### Authentication observable magic

Note that we saved ourselves a bit of trouble by implementing authentication with Observables in the main component:

```ts
// Direct the user to the correct page depending on whether he or she is logged in.
this.auth.isAuthenticated().subscribe(authenticated => {
  if (authenticated) {
    this.rootPage = HomePage;
  } else {
    this.rootPage = LoginPage;
  }
});
```

It subscribes to the Observable authentication stream when the app starts and **keeps listening to events**,
meaning that it will get notified of any change in the current authentication status:

* When a user **logs in**, the subscription callback is called with `true` to indicate that a user logged in,
  and the user is redirected to the `HomePage`.
* When a user **logs out**, the subscription callback is called again with `false`,
  and the user is redirected to the `LoginPage`.

That way, we didn't have to explicitly add some more code to redirect the user to the login page
when called the authentication provider's `logOut()` function from the `CreateTripPage` component's `logOut()` method.

<a href="#top">Back to top</a>



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
    private auth: AuthProvider,
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
$> ionic generate provider AuthInterceptor
```

Put the following contents in the generated `src/providers/auth-interceptor/auth-interceptor.ts` file:

```ts
import { HttpRequest, HttpHandler, HttpEvent, HttpInterceptor } from '@angular/common/http';
import { Injectable, Injector } from '@angular/core';
import { Observable } from 'rxjs/Rx';
import { first, switchMap } from 'rxjs/operators';

import { AuthProvider } from '../auth/auth';

@Injectable()
export class AuthInterceptorProvider implements HttpInterceptor {

  constructor(private injector: Injector) {}

  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {

    // Retrieve AuthProvider at runtime from the injector.
    // (Otherwise there would be a circular dependency:
    //  AuthInterceptorProvider -> AuthProvider -> HttpClient -> AuthInterceptorProvider).
    const auth = this.injector.get(AuthProvider);

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
import { AuthInterceptorProvider } from '../providers/auth-interceptor/auth-interceptor';

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

<a href="#top">Back to top</a>



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

<a href="#top">Back to top</a>



### Create a sample configuration file

Create a `src/app/config.sample.ts` file.
This file is a placeholder which should **NOT** contain the actual configuration.
Its purpose is to explain to other developers of the project that they should create a `config.ts` file and fill in the actual values:

```ts
// Copy this file to config.ts and fill in appropriate values.
export const config = {
  apiUrl: 'https://example.com/api'
}
```

<a href="#top">Back to top</a>



### Create the actual configuration file

You can now create the actual `src/app/config.ts` configuration file, this time containing the actual configuration values:

```ts
export const config = {
  apiUrl: 'https://comem-travel-log-api.herokuapp.com/api'
}
```

<a href="#top">Back to top</a>



### Add the configuration file to your `.gitignore` file

Of course, you **don't want to commit `config.ts`**, but you do want to commit `config.sample.ts`
so that anyone who clones your project can see what configuration options are required.
Add these 2 lines to your `.gitignore` file:

```
src/app/config.*
!src/app/config.sample.ts
```

The first line ignores any `config.*` file in the `src/app` directory.
The second line adds an exception: that the `config.sample.ts` should **not** be ignored.
You now have your uncommitted configuration file!

Not only that, but if you often need to quickly swap between different configurations,
you may create several versions of the configuration file, like `config.dev.ts` or `config.prod.ts`, which will all be ignored by Git.
Everytime you need to use one of them, simply overwrite `config.ts` with the correct one:

```bash
$> cp src/app/config.dev.ts src/app/config.ts
$> cp src/app/config.prod.ts src/app/config.ts
```

This is something you could put in your `package.json`'s `scripts` section:

```json
"scripts": {
  "dev": "cp src/app/config.dev.ts src/app/config.ts",
  "prod": "cp src/app/config.prod.ts src/app/config.ts",
  "...": "..."
}
```

<a href="#top">Back to top</a>



### Feed the configuration to Angular

Now that you have your configuration files, you want to use its values in code.

Since it's a TypeScript file like any other, you simply have to import and use it,
for example in `src/pages/trip-list/trip-list.ts`:

```ts
// Other imports...
// TODO: import the configuration.
import { config } from '../../app/config';

// ...
export class TripListPage {
  // ...
  ionViewDidLoad() {
    console.log('ionViewDidLoad TripListPage');

    // TODO: replace the hardcoded API URL by the one from the configuration.
    const url = `${config.apiUrl}/trips`;
    this.httpClient.get(url).subscribe(trips => {
      console.log('Trips loaded', trips);
    });
  }
  // ...
}
```

Do not forget to also update the authentication provider in `src/providers/auth/auth.ts`,
which also has a hardcoded URL:

```ts
// Other imports...
// TODO: import the configuration.
import { config } from '../../app/config';

// ...
export class AuthProvider {
  // ...
  logIn(authRequest: AuthRequest): Observable<User> {

    // TODO: replace the hardcoded API URL by the one from the configuration.
    const authUrl = `${config.apiUrl}/auth`;
    // ...
  }
  // ...
}
```

<a href="#top">Back to top</a>



## Troubleshooting



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
