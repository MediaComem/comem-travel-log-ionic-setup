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
  <ion-tab *ngFor='let tab of tabs' tabTitle='{{ tab.title }}' tabIcon='{{ tab.icon }}' [root]='tab.component'></ion-tab>
</ion-tabs>
```

You should now be able to navigate between the 3 tabs!



## 4. Set up security

To use the app, a citizen should identify him- or herself.
You will add a login screen that the user must go through before accessing the other screens.

You will use the API you previously implemented to actually log in.
If you do not have a running API, you may also use the [reference implementation](https://mediacomem.github.io/comem-citizen-engagement-api/).

The reference implementation authenticates users with a bearer token
This token must be sent in the `Authorization` header for all requests requiring identification.

<a href="#top">Back to top</a>



### Create the login screen

Let's start by creating a login screen.
Add a `login.html` template in `www/templates`:

```html
<ion-view view-title="Citizen Engagement">
  <ion-content class="padding">

    <!-- A banner. Choose an image and save it as www/img/banner.jpg -->
    <!--<img src="img/banner.jpg" width="100%" alt="Citizen Engagement" />-->

    <!-- A short welcome message. -->
    <div class="card">
      <div class="item item-text-wrap">
        Welcome to Citizen Engagement!
        Please log in to use the application.
      </div>
    </div>

    <!-- The login form that the citizen must fill. -->
    <form name="loginForm">

      <!-- Ionic uses lists to group related input elements. -->
      <div class="list">
        <label class="item item-input">
          <!-- Note the required="required" attribute used for validation. -->
          <input type="text" placeholder="Username" ng-model="loginCtrl.user.name" required />
        </label>
        <label class="item item-input">
          <input type="password" placeholder="Password" ng-model="loginCtrl.user.password" required />
        </label>
      </div>

      <!-- Display an error message here, above the submit button, if an error occurred. -->
      <p ng-if="loginCtrl.error" class="error">{{ loginCtrl.error }}</p>

      <!--
        The submit button.
        The "ng-click" directive indicates which scope function to call when the form is submitted.
        The "ng-disabled" directive is used to disable the button if the form is invalid (i.e. the first name or last name is missing).
      -->
      <button class="button button-full button-positive" ng-click="loginCtrl.logIn()" ng-disabled="loginForm.$invalid">Log in</button>
    </form>
  </ion-content>
</ion-view>
```

This proposed login template has an image banner.
Don't forget to choose an image and save it as `www/img/banner.jpg`.

Define the state for the login screen in `www/js/app.js`

```js
.state('login', {
  url: '/login',
  templateUrl: 'templates/login.html'
})
```

<a href="#top">Back to top</a>



### Create the authentication service

Now that we have our login screen, we must configure the app to redirect the user to it if he hasn't yet logged in.
To do that, we need a way to tell whether the user has logged in or not.
This will be our first AngularJS service: the authentication service.

Create a new `www/js/auth.js` file containing this:

```js
angular.module('citizen-engagement').factory('AuthService', function() {

  var service = {
    authToken: null,

    setAuthToken: function(token) {
      service.authToken = token;
    },

    unsetAuthToken: function() {
      service.authToken = null;
    }
  };

  return service;
});
```

This file defines a new service, `AuthService`, which manages the user authentication with the `authToken` property.
If this property has a value, the user is logged in; otherwise, he isn't.

To use this new file, you must add it to `www/index.html` like this:

```html
  <head>
    <!-- ... meta, link, etc ... -->

    <!-- your app's js -->
    <script src="js/app.js"></script> <!-- The existing module for the whole application. -->
    <script src="js/auth.js"></script> <!-- Your new module. -->
  </head>
```

We will implement the rest of this service later.
First, let's make sure the app shows the login screen.
Edit `www/js/app.js` and add this `run` block:

```js
angular.module('citizen-engagement').run(function(AuthService, $rootScope, $state) {

  // Listen for the $stateChangeStart event of AngularUI Router.
  // This event indicates that we are transitioning to a new state.
  // We have the possibility to cancel the transition in the callback function.
  $rootScope.$on('$stateChangeStart', function(event, toState) {

    // If the user is not logged in and is trying to access another state than "login"...
    if (!AuthService.authToken && toState.name != 'login') {

      // ... then cancel the transition and go to the "login" state instead.
      event.preventDefault();
      $state.go('login');
    }
  });
});
```

The login screen is ready!
If you reload your app, you should see that you are automatically redirected to the login page.
You also can't go to any other screen any more, since we haven't implemented the actual login yet.

<a href="#top">Back to top</a>



### Set up a proxy (for local development only)

If you are serving your Ionic app locally with `ionic serve`,
**beware of the [same-origin policy](http://en.wikipedia.org/wiki/Same-origin_policy)!**

In a web application, the browser will only allow remote calls to the same **origin**.
Your Ionic app is essentially a web page, served at `http://localhost:8100`.
If your API lives at `https://api.example.com`, you won't be able to call it from your Ionic app because the browser will block the call.

To work around this issue, add the following `proxies` configuration to your `ionic.config.json` file:

```
{
  "name": "citizen-engagement",
  "app_id": "",
  "proxies": [
    {
      "path": "/api-proxy",
      "proxyUrl": "https://comem-citizen-engagement.herokuapp.com/api"
    }
  ]
}
```

This will proxy calls to your API if your URL path starts with `/api-proxy`.
For example, if you call `/api-proxy/users` from your application, it will actually call `https://comem-citizen-engagement.herokuapp.com/api/users`.

**You must** terminate `ionic serve` and re-launch it to take this configuration into account.

<a href="#top">Back to top</a>



### Log in with the API

You will now integrate with the API!

In the proposed implementation, we will use the `/auth` resource [described in the reference implementation](https://mediacomem.github.io/comem-citizen-engagement-api/#auth_post).
This action either returns an existing user or creates it if it doesn't exist.

We need to make the following call:

```json
POST /api/auth HTTP/1.1
Content-Type: application/json

{
  "name": "jdoe",
  "password": "test"
}
```

The response will contain the token we need for authentication:

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

You must make this call when the user clicks on the "Log in" button of the login template.
As you can see in the proposed login template you added earlier, the submit button already has an `ng-click` directive.

```html
<button class="button button-full button-positive" ng-click="loginCtrl.logIn()" ng-disabled="loginForm.$invalid">Log in</button>
```

The directive defines what happens when the button is clicked.
In this case, it calls the `logIn()` function.

To react to view events, we need an AngularJS **controller**.
In the controller, you will be able to add the `register()` function to the scope.

Let's add it to `www/js/auth.js` after the service:

```js
angular.module('citizen-engagement').controller('LoginCtrl', function(AuthService, $http, $ionicHistory, $ionicLoading, $scope, $state) {
  var loginCtrl = this;

  // The $ionicView.beforeEnter event happens every time the screen is displayed.
  $scope.$on('$ionicView.beforeEnter', function() {
    // Re-initialize the user object every time the screen is displayed.
    // The first name and last name will be automatically filled from the form thanks to AngularJS's two-way binding.
    loginCtrl.user = {};
  });

  // Add the register function to the scope.
  loginCtrl.logIn = function() {

    // Forget the previous error (if any).
    delete loginCtrl.error;

    // Show a loading message if the request takes too long.
    $ionicLoading.show({
      template: 'Logging in...',
      delay: 750
    });

    // Make the request to authenticate the user.
    $http({
      method: 'POST',
      url: '/api-proxy/auth',
      data: loginCtrl.user
    }).then(function(res) {

      // If successful, give the token to the authentication service.
      AuthService.setAuthToken(res.data.token);

      // Hide the loading message.
      $ionicLoading.hide();

      // Set the next view as the root of the history.
      // Otherwise, the next screen will have a "back" arrow pointing back to the login screen.
      $ionicHistory.nextViewOptions({
        disableBack: true,
        historyRoot: true
      });

      // Go to the issue creation tab.
      $state.go('tab.newIssue');

    }).catch(function() {

      // If an error occurs, hide the loading message and show an error message.
      $ionicLoading.hide();
      loginCtrl.error = 'Could not log in.';
    });
  };
});
```

Take a moment to read the comments in the controller's code.

Simply defining the controller isn't enough.
You must also update the login state to use it (in `www/js/app.js`):

```js
.state('login', {
  url: '/login',
  controller: 'LoginCtrl',
  controllerAs: 'loginCtrl',
  templateUrl: 'templates/login.html'
})
```

If your API works, you should now be able to log in!

<a href="#top">Back to top</a>



### Log out

You should also allow the user to log out.

Let's add another controller to `www/js/auth.js`:

```js
angular.module('citizen-engagement').controller('LogoutCtrl', function(AuthService, $state) {
  var logoutCtrl = this;

  logoutCtrl.logOut = function() {
    AuthService.unsetAuthToken();
    $state.go('login');
  };
});
```

And add the logout button.
You can add override any screen's navigation buttons using Ionic's `<ion-nav-buttons>` directive.

For example, you can update your issue creation template (`www/templates/newIssue.html`) to look like this:

```html
<ion-view view-title="New Issue">
  <ion-nav-buttons side="secondary">
    <button type="button" ng-controller="LogoutCtrl as logoutCtrl" ng-click="logoutCtrl.logOut()" class="button">Log Out</button>
  </ion-nav-buttons>
  <ion-content>
    <p>Hello! This is the issue creation screen.</p>
  </ion-content>
</ion-view>
```

You should now see the logout button in the navigation bar after logging in.
You may also add the logout button to the other two tabs if you want.

<a href="#top">Back to top</a>



### Storing the authentication credentials

Now you can log in and log out, but there's a little problem.
Every time the app is reloaded, you lose all data so you have to log back in.
This is particularly annoying for local development since the browser is automatically refreshed every time you change the code.

You need to use more persistent storage for the security credentials, i.e. the authentication token.
Since an Ionic app is a web app, the simplest is to use [Web Storage](http://en.wikipedia.org/wiki/Web_storage),
or more specifically [Local Storage](http://diveintohtml5.info/storage.html).

Starting with HTML 5, you have access to the `localStorage` variable from your Javascript code.
You may store any key/value pair in local storage.
It will persist even after you navigate away to another page or close the page.

```js
localStorage.foo = "bar";
console.log(localStorage.foo); // => "bar"
```

However, you **should not** use it directly like this in an AngularJS application; that is not the Angular way.
In an Angular application, you should always wrap such functionality in a service.
This encourages each service to do only one thing and to do it well.
It also makes it easier to unit test the service.

Instead of writing a service yourself, you should always check if someone has already done it for you.
In this case, we propose to use the [auth0/angular-storage](https://github.com/auth0/angular-storage) library which is a nice AngularJS wrapper around the native local storage functionality.

As shown in their documentation, it can be installed with Bower.
Since your Ionic app already uses Bower by default, it's trivial to add the Bower dependency:

```bash
$> bower install --save a0-angular-storage
```

If you look in `www/lib`, you should see a new `a0-angular-storage` directory.

As with all Javascript files, you must add it to `www/index.html` to use it.
You should add it next to the existing Ionic script tag:

```html
  <head>
    <!-- ... meta, link, etc ... -->

    <!-- ionic/angularjs js -->
    <script src="lib/ionic/js/ionic.bundle.js"></script> <!-- The existing Ionic dependency. -->
    <script src="lib/a0-angular-storage/dist/angular-storage.js"></script> <!-- The new dependency. -->

    <!-- ... more scripts ... -->
  </head>
```

And as with all AngularJS modules, you must add it as a dependency in `www/app.js`:

```js
angular.module('citizen-engagement', ['ionic', 'angular-storage'])
```

You can now update the `AuthService` in the same file to use persistent storage:

```js
angular.module('citizen-engagement').factory('AuthService', function(store) {

  var service = {
    authToken: store.get('authToken'),

    setAuthToken: function(token) {
      service.authToken = token;
      store.set('authToken', token);
    },

    unsetAuthToken: function() {
      service.authToken = null;
      store.remove('authToken');
    }
  };

  return service;
});
```

Your app should now remember user credentials even when you reload it!

<a href="#top">Back to top</a>



### Configuring an HTTP interceptor

Now that you have login and logout functionality, and an authentication service that can give you an authentication token, you can authenticate for other API calls.
Taking an example from the reference implementation, you could [retrieve the list of issues](https://mediacomem.github.io/comem-citizen-engagement-api/#issues_get).
The documentation states that we must send a bearer token in the `Authorization` header, like this:

```
GET /api/issues HTTP/1.1
Authorization: Bearer 0a98wumv
```

With Angular, you would make this call like this:

```js
angular.module('citizen-engagement').controller('AnyCtrl', function(AuthService, $http) {
  $http({
    url: '/api-proxy/issues',
    headers: {
      Authorization: 'Bearer ' + AuthService.authToken
    }
  }).then(function(res) {
    // ...
  });
})
```

But it's a bit annoying to have to specify this header for every request.
After all, we know that we need it for most, if not all calls.

[**Interceptors**](http://www.webdeveasy.com/interceptors-in-angularjs-and-useful-examples/) are Angular services that can be registered with the `$http` service to automatically modify all requests (or responses).
This solves our problem: we want to register an interceptor that will automatically add the `X-User-Id` header to all requests if the user is logged in.

First, let's create the interceptor service.
Since it's authentication-related, let's add it to `www/js/auth.js`:

```js
angular.module('citizen-engagement').factory('AuthInterceptor', function(AuthService) {
  return {

    // The request function will be called before all requests.
    // In it, you can modify the request configuration object.
    request: function(config) {

      // If the user is logged in, add the X-User-Id header.
      if (AuthService.authToken) {
        config.headers.Authorization = 'Bearer ' + AuthService.authToken;
      }

      return config;
    }
  };
});
```

Now you simply need to register the interceptor with the provider of the `$http` service.
In the same file, add:

```js
angular.module('citizen-engagement').config(function($httpProvider) {
  $httpProvider.interceptors.push('AuthInterceptor');
});
```

Now all your API calls will have this header when the user is logged in.

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
[ionic-tabs]: https://ionicframework.com/docs/components/#tabs
[sass]: http://sass-lang.com
