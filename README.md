# COMEM+ Citizen Engagement Ionic Setup

<a name="top"></a>

This repository contains instructions to build a skeleton application that will serve as a starting point to develop the Citizen Engagement mobile application.
The completed skeleton app is available [here](https://github.com/MediaComem/comem-citizen-engagement-ionic-starter).

This tutorial is used in the [COMEM+](http://www.heig-vd.ch/comem) [Mobile Applications course](https://github.com/MediaComem/comem-appmob) taught at [HEIG-VD](http://www.heig-vd.ch).

<!-- START doctoc -->
<!-- END doctoc -->




## Prerequisites

These instructions assume that you are using the Citizen Engagement API described in the previous [Web Services course](https://github.com/MediaComem/comem-webserv),
and that you are familiar with the [documentation of the reference API](https://mediacomem.github.io/comem-citizen-engagement-api).

You will need to have [Node.js](https://nodejs.org) installed.
The latest LTS (Long Term Support) version is recommended (v8.9.3 at the time of writing these instructions).

<a href="#top">Back to top</a>



## Features

This guide describes a proposed list of features and a user interface based on those features.
This is only a suggestion. You can support other features and make a different user interface.

The proposed app should allow citizens to do the following:

* add new issues:
  * the issue should have a type and description;
  * the user should be able to take a photo of the issue;
  * the issue should be geolocated;
* see existing issues on an interactive map;
* browse the list of existing issues:
  * issues should be sorted by date;
* see the details of an issue:
  * date;
  * description;
  * picture;
  * comments;
* add comments to an issue.

The following sections describe a proposed UI mockup of the app and steps to set up a skeleton implementation.

<a href="#top">Back to top</a>



## 1. Design the user interface

Before diving into the code, you should always take a moment to design the user interface of your app.
This doesn't have to be a final design, but it should at least be a sketch of what you want.
This helps you think in terms of the whole app and of the relationships between screens.

![UI Design](setup/ui-structure.png)

As you can see, we propose to use a tab view with 3 screens, and an additional 4th screen accessible from the issue list:

* the new issue tab;
* the issue map tab;
* the issue list tab:
  * the issue details screen.

Now that we know what we want, we can start setting up the app!

<a href="#top">Back to top</a>



## 2. Set up the application



### Create a blank Ionic app and make it a repository

Make sure you have Ionic and Cordova installed:

```bash
$> npm install -g ionic cordova
```

Go in the directory where you want the app, then generate a blank Ionic app with the following command:

```bash
$> cd /path/to/projects
$> ionic start citizen-engagement blank

? Would you like to integrate your new app with Cordova to target native iOS and Android? Yes
? Install the free Ionic Pro SDK and connect your app? No
```

Go into the app directory. The `ionic start` command should have already initialized a Git repository:

```bash
$> cd citizen-engagement
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



## 3. Set up the navigation structure

As defined in our UI design, we want the following 4 screens:

* the issue creation tab;
* the issue map tab;
* the issue list tab:
  * the issue details screen.

Let's start by setting up the 3 tabs.
We will use Ionic's [Tabs][ionic-tabs] component.

<a href="#top">Back to top</a>



### Create the pages

Each page will be an [Angular component][angular-component].
Ionic has a `generate` command that can automatically set up the files we need to create each page's component:

```bash
$> ionic generate page --no-module CreateIssue
$> ionic generate page --no-module IssueMap
$> ionic generate page --no-module IssueList
```

This will generate the following files:

```
src/pages/create-issue/create-issue.html
src/pages/create-issue/create-issue.scss
src/pages/create-issue/create-issue.ts
src/pages/issue-map/issue-map.html
src/pages/issue-map/issue-map.scss
src/pages/issue-map/issue-map.ts
src/pages/issue-list/issue-list.html
src/pages/issue-list/issue-list.scss
src/pages/issue-list/issue-list.ts
```

For each page, we have:

* An HTML template.
* A [Sass/SCSS][sass] stylesheet.
* An Angular component.

Now update the HTML template for each page and add some content within the `<ion-content>` tag.
For example, in `src/pages/create-issue/create-issue.html`:

```html
<ion-content padding>
  Let's create an issue.
</ion-content>
```



### Update the app to use the pages

Now that the pages are ready, we need to display them.

First, Angular and Ionic need to be aware of your new components.
You must **declare** them in 2 places in your main Angular module in `src/app/app.module.ts`:

* Add them to the `declarations` array to register the components with Angular.
* Add them to the `entryComponents` array so that Ionic is able to inject them dynamically (e.g. when switching tabs).

```js
import { CreateIssuePage } from '../pages/create-issue/create-issue';
import { IssueListPage } from '../pages/issue-list/issue-list';
import { IssueMapPage } from '../pages/issue-map/issue-map';

@NgModule({
  declarations: [
    MyApp,
    HomePage,
    CreateIssuePage,
    IssueListPage,
    IssueMapPage
  ],
  imports: [ /* ... */ ],
  bootstrap: [ /* ... */ ],
  entryComponents: [
    MyApp,
    HomePage,
    CreateIssuePage,
    IssueListPage,
    IssueMapPage
  ],
  providers: [ /* ... */ ]
})
export class AppModule {}
```

Second, **update the home page's component** (`src/pages/home/home.ts`) to include the list of tabs we want:

```js
import { CreateIssuePage } from '../create-issue/create-issue';
import { IssueMapPage } from '../issue-map/issue-map';
import { IssueListPage } from '../issue-list/issue-list';

@Component({
  selector: 'page-home',
  templateUrl: 'home.html'
})
export class HomePage {

  /**
   * Application tabs.
   *
   * This is a list of objects representing tabs.
   * Each tab has a `title` and `icon` to customize the tab,
   * and the `component` property to determine which page to display.
   */
  tabs: any[];

  constructor(public navCtrl: NavController) {
    this.tabs = [
      { title: 'New Issue', icon: 'add', component: CreateIssuePage },
      { title: 'Issue Map', icon: 'map', component: IssueMapPage },
      { title: 'Issue List', icon: 'list', component: IssueListPage }
    ];
  }

}
```

Third, we will **replace the contents of the home page's template** (`src/pages/home/home.html`) to use Ionic's Tabs component.

Angular's `ngFor` directive allows us to iterate over the `tabs` array we declared in the home page's component,
and to put one `<ion-tab>` tag in the page for each component:

```html
<ion-tabs>
  <ion-tab *ngFor='let tab of tabs' [tabTitle]='tab.title' [tabIcon]='tab.icon' [root]='tab.component'></ion-tab>
</ion-tabs>
```

You should now be able to navigate between the 3 tabs!



## 4. Set up security

To use the app, a citizen should identify him- or herself.
You will add a login screen that the user must go through before accessing the other screens.
Authentication will be performed by the [Citizen Engagement API][citizen-engagement-api].

The API requires a bearer token be sent to identify the user when making requests on some resources (e.g. when creating issues).
This token must be sent in the `Authorization` header for all requests requiring identification.
Once login/logout is implemented, you will also set up an HTTP interceptor to automatically add this header to every request.



### Check the documentation of the API's authentication resource

The Citizen Engagement API provides an [`/auth` resource](https://mediacomem.github.io/comem-citizen-engagement-api/#auth_post)
on which you can make a POST request to authenticate.

You need to make a call that looks like this:

```json
POST /api/auth HTTP/1.1
Content-Type: application/json

{
  "name": "jdoe",
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
    "firstname": "John",
    "id": "58c5b50e046ea004e4af9d32",
    "lastname": "Doe",
    "name": "jdoe",
    "roles": [
      "citizen"
    ]
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
  firstname: string;
  lastname: string;
  roles: string[];
}
```

Create a `src/models/auth-request.ts` file which exports a model representing a request to the authentication resource:

```ts
export class AuthRequest {
  name: string;
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

You can replace the contents of the generated `srv/providers/auth/auth.ts` file with the following code:

```ts
import { HttpClient } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { Response } from '@angular/http';

import { AuthRequest } from '../../models/auth-request';
import { AuthResponse } from '../../models/auth-response';
import { User } from '../../models/user';

/**
 * Authentication service for login/logout.
 */
@Injectable()
export class AuthProvider {

  private auth: AuthResponse;

  constructor(private http: HttpClientj {
  }

  isAuthenticated(): boolean {
    return !!this.auth;
  }

  getUser(): User {
    return this.auth ? this.auth.user : null;
  }

  getToken(): string {
    return this.auth ? this.auth.token : null;
  }

  async logIn(authRequest: AuthRequest): Promise<User> {
    this.auth = await this.http.post<AuthResponse>('https://comem-citizen-engagement.herokuapp.com/api/auth', authRequest).toPromise();
    console.log(`User ${this.auth.user.name} logged in`);
    return this.auth.user;
  }

  logOut() {
    this.auth = null;
    console.log('User logged out');
  }

}
```



### Create the login screen

Generate a login page component:

```bash
$> ionic generate page --no-module Login
```

Add the following HTML form inside the `<ion-content>` tag of `src/pages/login/login.html`:

```html
<form (submit)="onSubmit($event)">
  <ion-list>

    <!-- Name input -->
    <ion-item>
      <ion-label floating>Name</ion-label>
      <ion-input type="text" name="name" #nameInput="ngModel" [(ngModel)]="authRequest.name" required></ion-input>
    </ion-item>

    <!-- Error message displayed if the name is invalid -->
    <ion-item [hidden]="nameInput.valid || nameInput.pristine" no-lines>
      <p ion-text color="danger">Name is required.</p>
    </ion-item>

    <!-- Password input -->
    <ion-item>
      <ion-label floating>Password</ion-label>
      <ion-input type="password" name="password" #passwordInput="ngModel" [(ngModel)]="authRequest.password" required></ion-input>
    </ion-item>

    <!-- Error message displayed if the password is invalid -->
    <ion-item [hidden]="passwordInput.valid || passwordInput.pristine" no-lines>
      <p ion-text color="danger">Password is required.</p>
    </ion-item>

  </ion-list>

  <div padding>

    <!-- Submit button -->
    <button type="submit" [disabled]="form.invalid" ion-button block>Log in</button>

    <!-- Error message displayed if the login failed -->
    <p [hidden]="!loginError" ion-text color="danger">Name or password is invalid.</p>

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
    this.auth.logIn(this.authRequest).subscribe(user => {
      this.navCtrl.setRoot(HomePage);
    }, err => {
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
import { AuthProvider } from '../providers/auth/auth';
import { LoginPage } from '../pages/login/login';
```

Update the component's constructor as follows:

```ts
constructor(private auth: AuthProvider, platform: Platform, statusBar: StatusBar, splashScreen: SplashScreen) {

  // Direct the user to the correct page depending on whether he or she is logged in.
  if (this.auth.isAuthenticated()) {
    this.rootPage = HomePage;
  } else {
    this.rootPage = LoginPage;
  }

  // ...
}
```

The login screen is ready!
If you reload your app, you should see that you are automatically redirected to the login page.
You also can't go to any other screen any more, since we haven't implemented the actual login yet.

<a href="#top">Back to top</a>



### Storing the authentication credentials

Now you can log in and log out, but there's a little problem.
Every time the app is reloaded, you lose all data so you have to log back in.
This is particularly annoying for local development since the browser is automatically refreshed every time you change the code.

You need to use more persistent storage for the security credentials, i.e. the authentication token.
Ionic provides a [storage module](https://ionicframework.com/docs/storage/) which will automatically select an appropriate storage method for your platform.
It will use [SQLite](https://sqlite.org) on phones when available; for web platforms it will use [IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API), [WebSQL](https://www.w3.org/TR/webdatabase/) or [Local Storage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage).

To use the Ionic storage module, you must import it into your application's module in `src/app/app.module.ts`:

```ts
// Other imports...
import { IonicStorageModule } from '@ionic/storage';

@NgModule({
  // ...
  imports: [
    // Other modules...
    IonicStorageModule.forRoot()
  ],
  // ...
})
export class AppModule {}
```

You also need to inject it into the constructor of `AuthProvider` in `src/providers/auth/auth.ts`:

```ts
constructor(private http: HttpClient, private storage: Storage)
```

You can now update the `logIn` method to persist the API's authentication response with `this.storage.set`.
Note that `set` is asynchronous and returns a promise, so you should wait for it to finish before returning the logged user.

```ts
async logIn(authRequest: AuthRequest): Promise<User> {
  this.auth = await this.http.post<AuthResponse>('https://comem-citizen-engagement.herokuapp.com/api/auth', authRequest).toPromise();
  console.log(`User ${this.auth.user.name} logged in`);
  await this.storage.set('auth', this.auth);
  return this.auth.user;
}
```

When testing in the browser, you should already see the object being stored in IndexedDB (the default storage if using Chrome).

You must now load it when the app starts.
You can do that in the constructor of `AuthProvider`.
Since `get` returns a promise, you can only use the result in a `.then` asynchronous callback:

```ts
this.storage.get('auth').then(auth => {
  this.auth = auth;
});
```

However, simply doing this in the constructor of `AuthProvider` will not work.
Why? Because of the code in you main component in `src/app/app.component.ts`:

```ts
if (this.auth.isAuthenticated()) {
  this.rootPage = HomePage;
} else {
  this.rootPage = LoginPage;
}
```

This code checks whether the user is authenticated immediately when the app starts.
However, at that time, the promise returned by `this.storage.get('auth')` has not yet been resolved
(since promises are always asynchronous).

One way to solve the problem is to store the "initialization" promise in `AuthProvider` and provide a method to know when it is resolved:

```ts
private initializationPromise: Promise<void>;

constructor(private http: HttpClient, private storage: Storage) {
  this.initializationPromise = this.storage.get('auth').then(auth => {
    this.auth = auth;
  });
}

waitForInitialization(): Promise<void> {
  return this.initializationPromise;
}
```

That way, you can wait for initialization to be complete in the main component in `src/app/app.component.ts`:

```ts
this.auth.waitForInitialization().then(() => {
  if (this.auth.isAuthenticated()) {
    this.rootPage = HomePage;
  } else {
    this.rootPage = LoginPage;
  }
});
```

Your app should now remember user credentials even when you reload it!

<a href="#top">Back to top</a>



### Log out

You should also allow the user to log out.
As an example, you will display a logout button in the issue creation screen.

Add an `<ion-buttons>` tag with a logout button in `src/pages/create-issue/create-issue.html`:

```html
<ion-navbar>
  <ion-title>CreateIssue</ion-title>

  <!-- Logout button -->
  <ion-buttons end>
    <button ion-button icon-only (click)="logOut()">
      <ion-icon name="log-out"></ion-icon>
    </button>
  </ion-buttons>
</ion-navbar>
```

Let's assume that when logging out, we want the user redirected to the login page,
and we want the navigation stack to be cleared (so that pressing the back button doesn't bring the user back to a protected screen).

To do that, you will need:

* To inject the Ionic application (`App` from the `ionic-angular` package),
  which will allow you to set the root page and thereby clear the navigation stack.
* To add a `logOut` method to the `CreateIssuePage` component,
  since it's what we call in its HTML template above.
* To inject `AuthProvider` and use its `logOut` method.

After doing all that, your `CreateIssuePage` component should look something like this:

```ts
import { Component } from '@angular/core';
import { App, NavController, NavParams } from 'ionic-angular';

import { AuthProvider } from '../../providers/auth/auth';
import { LoginPage } from '../login/login';

@Component({
  templateUrl: 'create-issue.html'
})
export class CreateIssuePage {

  constructor(private app: App, private auth: AuthProvider, public navCtrl: NavController, public navParams: NavParams) {
  }

  logOut() {
    this.auth.logOut().then(() => {
      this.app.getRootNavs()[0].setRoot(LoginPage);
    });
  }

}
```

You should now see the logout button in the navigation bar after logging in.

Later, you might want to encapsulate it into a reusable component to include in other screens.

<a href="#top">Back to top</a>



### Configuring an HTTP interceptor

Now that you have login and logout functionality, and an authentication service that stores an authentication token, you can authenticate for other API calls.

Looking at the API documentation, at some point you will need to [create an issue](https://mediacomem.github.io/comem-citizen-engagement-api/#issues_post).
The documentation states that you must send a bearer token in the `Authorization` header, like this:

```
POST /api/issues HTTP/1.1
Authorization: Bearer 0a98wumv
Content-Type: application/json

{"some":"json"}
```

With Angular, you would make this call like this:

```js
this.http.post('https://comem-citizen-engagement.herokuapp.com/api/auth', issue, {
  headers: {
    Authorization: `Bearer ${token}`
  }
});
```

But it's a bit annoying to have to specify this header for every request.
After all, we know that we need it for many calls.

[Interceptors](https://medium.com/@ryanchenkie_40935/angular-authentication-using-the-http-client-and-http-interceptors-2f9d1540eb8)
are Angular services that can be registered with the HTTP client to automatically react to requests (or responses).
This solves our problem: we want to register an interceptor that will automatically add the `Authorization` header to all requests if the user is logged in.

To demonstrate that it works, start by adding a call to list issues in the `IssueListPage` component in `src/pages/issue-list/issue-list.ts`:

```ts
// Other imports...
import { HttpClient } from '@angular/common/http';

@Component({
  templateUrl: 'issue-list.html'
})
export class IssueListPage {

  constructor(public http: HttpClient, public navCtrl: NavController, public navParams: NavParams) {
  }

  ionViewDidLoad() {
    this.http.get('https://comem-citizen-engagement.herokuapp.com/api/issues').subscribe(issues => {
      console.log(`Issues loaded`);
    });
  }

}
```

If you display the issue list page and check network requests in Chrome's developer tools,
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

import { AuthProvider } from '../auth/auth';

@Injectable()
export class AuthInterceptorProvider implements HttpInterceptor {

  constructor(private injector: Injector) {}

  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {

    // Retrieve AuthProvider at runtime from the injector.
    // (Otherwise there would be a circular dependency: AuthInterceptorProvider -> AuthProvider -> HttpClient -> AuthInterceptorProvider).
    const auth = this.injector.get(AuthProvider);

    // Get the bearer token (if any).
    const token = auth.getToken();

    // Add it to the request if it doesn't already have an Authorization header.
    if (token && !req.headers.has('Authorization')) {
      req = req.clone({
        headers: req.headers.set('Authorization', `Bearer ${token}`)
      });
    }

    // Perform the request.
    return next.handle(req);
  }

}
```

Now you simply need to register the interceptor in your application module.
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



## 5. Multi-environment configuration

A problem you will quickly encounter with Ionic is multiple environments.
You want to be able to:

* serve the app locally;
* run the app on a connected device.

In the first case, as you've seen, you need to set up a proxy to work around the same-origin policy.
But when you run the app on a device, Cordova sets things up so you don't have to do that, and you can't use the proxy.

This means that you need to change your URLs depending on which **environment** you're running the app in:

* when serving the app locally, use `/api-proxy`;
* when running the app on a device, use `https://comem-citizen-engagement.herokuapp.com/api`.

In the [login implementation you did earlier](#security-api-login), the URL was hardcoded:

```js
$http({
  method: 'POST',
  url: '/api-proxy/auth',
  data: loginCtrl.user
})
```

This is very bad considering the multi-environment problem.
You would have to manually change the URL every time you change environments.
Even if you set up a global variable so you only have to do it in once place, think what will happen when you start integrating with other APIs or services.
You will have that many settings to change every time.

Let's find a way to automate this process.

<a href="#top">Back to top</a>



### Write environment-specific configuration files

A pattern used in many frameworks is to have a configuration file for each environment.
Since we only have the URL of the API for now, the configuration would look like this:

```json
{
  "apiUrl": "https://<API_HOST>/api"
}
```

Let's set this up now.
Create a `config` directory at the root of your application.

Add `development.json` in that directory.
This file contains the configuration for the **development environment**, when you develop the app locally:

```json
{
  "apiUrl": "/api-proxy"
}
```

Now add `production.json` in the same directory.
This file contains the configuration for the **production environment**, meaning when you run the app on a device:

```json
{
  "apiUrl": "https://comem-citizen-engagement.herokuapp.com/api"
}
```

<a href="#top">Back to top</a>



### Do not put configuration files under version control

Another good practice is to **NOT put your configuration files under version control**, especially if your Git repository is public.
API endpoints should not be exposed for everyone to see.
You might also store credentials for other APIs and services in the configuration files.
You definitely don't want those exposed.

Add the following line to your `.gitignore` file at the root of the repository:

```
config/*
```

Now there's just a little problem: if you clone this repository on another machine, you won't have any configuration file.
This is good because you don't want anyone who clones the repository to obtain configuration that should remain secret.
But maybe you forgot what goes into the configuration file.

That is why you should always provide a sample configuration file.
Add `sample.json` to the `config` directory:

```json
{
  "apiUrl": "https://example.com/api"
}
```

Also add this line to the `.gitignore` file, **below the other line you added**:

```
!config/sample.json
```

This is an exception to the previous rule.
It tells Git to keep the `sample.json` file even though you're ignoring everything else in the directory.

<a href="#top">Back to top</a>



### Feed the configuration to Angular

Now that you have your environment-specific configuration files, you want to be able to use this configuration in Angular controllers and services.
You don't have direct access to those files from the Angular application.
You need to convert the JSON file for the correct environment into something Angular can use.

This kind of configuration is usually found in Angular constants:

```js
.constant('apiUrl', 'https://example.com/api')
```

Of course, you can't just use a hardcoded URL like this.
Let's create a new `constants.js` file to hold these constants.
Unlike other JavaScript files that are in `www/js`, you should create this one at the root of the repository (at the same level as `www`):

```js
angular.module('citizen-engagement')
  .constant('apiUrl', '@apiUrl@')
;
```

As you can see, there's no URL here; it's only a placeholder.
We'll use [Gulp](http://gulpjs.com), the automation tool bundled with Ionic, to insert the actual configuration values.
Gulp allows you to define custom tasks for your project; for example, you could define a task to compress your Javascript files.
In this case, you're going to define a task to inject the configuration into the constants template, and save the result where your app can use it.

Install the `gulp-replace` dependency that can do the replacing for us:

```
npm install --save gulp-replace
```

Include it at the top of `gulpfile.js`:

```js
var replace = require('gulp-replace');
```

Now add the task (still in `gulpfile.js`):

```js
function saveConfig(environment) {

  var config = require('./config/' + environment + '.json');

  // Use `constants.js` as the source.
  gulp.src(['constants.js'])

    // Replace all occurrences of @apiUrl@.
    .pipe(replace(/@apiUrl@/g, config.apiUrl))

    // Save the result in www/js.
    .pipe(gulp.dest('www/js'));
}

gulp.task('config-development', function(){
  saveConfig('development');
});

gulp.task('config-production', function(){
  saveConfig('production');
});
```

Now, when you want to switch environments, just run `gulp config-development` or `gulp config-production` from your terminal, and the Angular constants in `www/js/constants.js` will be updated.

**You must also add this file to `.gitignore`**, or all this work to avoid exposing configuration values will have been for nothing:

```
www/js/constants.js
```

All that remains is to use the constants in your app.

Since this is a new file, you must add it to `www/index.html`:

```html
  <head>
    <!-- ... meta, link, etc ... -->

    <!-- your app's js -->
    <script src="js/app.js"></script>
    <script src="js/auth.js"></script>
    <script src="js/constants.js"></script>
  </head>
```

You can now inject `apiUrl` anywhere you want.
Let's add it to the injections of `LoginCtrl` (in `www/js/auth.js`):

```js
.controller('LoginCtrl', function(apiUrl, AuthService, $http, $ionicHistory, $ionicLoading, $scope, $state) {
  // ...
});
```

Now update the API call to use `apiUrl` (same file):

```js
$http({
  method: 'POST',
  url: apiUrl + '/auth',
  data: loginCtrl.user
})
```

Finally!
Your configuration is now hidden from public view and properly injected into the app.

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



[angular-component]: https://angular.io/guide/architecture#components
[citizen-engagement-api]: https://comem-citizen-engagement.herokuapp.com/api
[ionic-tabs]: https://ionicframework.com/docs/components/#tabs
[sass]: http://sass-lang.com
