![sears holdings](Readme_Imgs/logo-united.jpg)

# Crew Scheduling GUI Architecture

#### Alert Monitoring Platform
-----------------


## Index
-----------------
* [Index](#index)
* [Revisions](#revisions)
* [Introduction](#introduction)
  * [Glossary](#glossary)
* [Assumptions](#assumptions)
* [Reusable code](#reusable-code)
  * [Separation of concerns](#separation-of-concerns)
  * [Features modules definitions](#features-modules-definitions)
  * [Key principles](#key-principles)
  * [Behavior extensibility pattern](#behavior-extensibility-pattern)
  * [Widget extension model](#widget-extension-model)
* [Navigation orchestration](#navigation-orchestration)
* [POC](#poc)
* [Configuration management draft](#configuration-management-draft)
  * [Initial setup](#initial-setup)
  * [Platform-repository correlation](#platform-repository-correlation)
  * [Improvements on code distribution](#improvements-on-code-distribution)
  * [Improvements on code optimization](#improvements-on-code-optimization)
* [Pull requests](#pull-requests)

## Revisions
-----------------
| Version | Date | Author | Description |
| ------- | ---- | ------ | ----------- |
| 1.0_A | 3/21/2014 | Nicolás Ávila | First draft capturing outcome of internal discussions | 
| 1.0_B | 3/30/2014 | Nicolás Ávila | Introduced changes after team review |
| 1.0_C | 4/08/2014 | Nicolás Ávila | Fixed filters on diagram 2 |
| 1.0_D | 8/04/2014 | Nicolás Ávila | Added issues on pull requests | 



## Introduction
-----------------
The purpose of this document is to define the highlights of the architecture attempted for ShopSears 2.5 application.

### Glossary
-----------------
| Term | Definition |
| ---- | ---------- |
| Application | An application is a standalone deliverable of code that can be utilized by itself as part of a solution. It’s interfaces to interact with other applications must be clearly defined, and provides abstraction of the context of execution. This means they can be utilized in more than one platform. What really defines an application is not just the underlying tech stack, but the flows and features that it is specialized for. Examples of such are the Web application used for shopsears, WLCC or the web applications for SYW kiosk.
| Platform | An application joins a group of applications that will work together to provide end users with a solution. ShopSears for iPad will be considered one platform, and so will be as well the Microsoft POS. 
| Platform family | A group of platforms providing similar UX, features and flows, will be considered a platform family as long as a significant portion of the applications can be reused from one to the next.

## Assumptions
-----------------
For the case of ShopSears, there are several platforms holding native applications, connecting to a shared web application that provides common set of features and flows.

We are working on the assumption that the experience will be kept as similar from one to the other as possible.

Native components will change for sure from one platform to the next, but the interfaces will be kept unchanged on any possible case.

## Reusable code
-----------------
For this application, we intend to divide code in a way that allows it to be reused from one app to the next.

Even expecting the experience to be kept regular among the apps in ShopSears platform ‘family’, we decide to plan for the scenario where customized applications are needed to be built in order to support each platform.

This will bring flexibility to adapt to change and also encourage of a decreased effort and higher consistency due to reuse in future applications being developed.

The following diagrams shows the scenario where more than one application rely in common components.

![code specialization model](Readme_Imgs/code_specialization_model.png)
Diagram 1: reuse layers

### Separation of concerns
-----------------
Although Diagram 1 looks like the common MVC diagram. That is not the correct interpretation.

The **AngularJs seed** presented in it is a base platform for builds, environments management, good practices support and optimization. It is based on Yeoman and it can be reused from one platform to the next regardless the business it involves. It could be used by other organizational units and even outside sears because it is not business specific.

The **FED reusable code** layer will work pretty similar to a library for the sake of our applications. It is tied to Sears business and based on SAL services. It involves all components that can be shared among more than one application.

The **application layer** comes on top of all that to wire the required elements for each application, allowing the override flows(routes), html, css, configuration, modules required. It allows also the redefinition of components and creation of application specific components.

The following diagram intends to show the way where we expect MVC layers to be laid out across each of these specialization layers.

![components in each reuse layer](Readme_Imgs/components_in_each_reuse_layer.png)
Diagram 2: MVC layers against specialization layers

### Features modules definitions
-----------------
Feature specific codes will be included both in the application layer and the application reuse layer.

Let’s take a look at diagram 1, once we drill down to add modules.

![modules in layers](Readme_Imgs/modules_in_layers.png)
Diagram 3: Feature modules across reuse layers

As you can see, the Seed is not affected by feature modules since it is business agnostic.
The FED reuse layer will create reusable components, and will group them by modules.
These components will be included by different applications, and complete the presentation layer.

Diagram 3 shows some of the modules on FED reuse layer being repeated on App specific layer. This does not mean the same components exist in both layers but that the app will add to the elements provided by the FED reuse layer.

For the following example, we will assume that TryOn is a feature that is only supported by ShopSears.

![expected components in layers](Readme_Imgs/expected_components_in_layers.png)
Diagram 4: Example for expected components in each layer

### Key principles
-----------------
* Widgetization

This applies both to UI components and logic components. It intends not just to maximize reuse but also reinforces separation of concerns and testability of components and increments.
* Abstraction of presentation

Many angular applications still rely on jQuery to modify the view. This shouldn’t be the case for us by any means.
Separating the presentation form the controller by communicating by using the scope will allow the view to handle specifics for each application.
* Restrict use of root scope

Root scope should be kept as clean as possible to reduce coupling among components.
* Separation of concerns

Angular’s controllers are supposed to be kept as clean as possible. In order to get that, we should intend to externalize as much as possible into services that extend this structure.
By doing this, we do not only decrease complexity and help the application to be easier to unit-test, but we also maximize the chance for reuse of these logical extensions.
* Self explanatory structure

The overall goal would  be to have a regular structure that is easy to understand for a new developer brought to the team, once he is presented the configuration management description (involving repositories and 

* Behavior extensibility pattern

It aims to give the different **application layers** the possibility of using the same set of controllers from the **FED reusable code** layer but letting each application extend the controllers functionality as needed. The details of this pattern are explained in the [Behavior extensibility pattern](#behavior-extensibility-pattern) and [widget extension model](#widget-extension-model) sections below.

### Behavior extensibility pattern
----------------------------------
Different applications might have the same functionality but with a slight different behavior. The main example is the Browse functionality of Kmart kiosk and mWeb.

In Kmart kiosk when the user browse throught a hierarchy of categories he navigates to a new page each time while in mWeb the current page is refreshed with the new information.
However, the mechanism is almost the same: react to an user event, invoke a SAL service, get the SAL response and show it to the user. The difference is just the way it is shown.

To handle this we developed a two level set of controllers (the component which handles the application logic):
* Level one: the core functionality, following the previous example here we have the SAL service invocation and the parse of the response.
* Level two: the specific functionality, in our example it involves the event handler (because different applications can trigger the same action based on different user events) and the data visualization.

This is implemented using the out of the box $controller AngularJS service. This service allow us to call a controller from inside another and inyect in the caller controller (level two) the functionality defined in the called controller (level one).
In this way the caller controller inherits all the functionality of the called controller plus it has the ability to define its own event handlers to decide there how to use the information retrieved by the inherited functionality (navigate to a new page or refresh the current one for instance).

An example of inheritance will look like this: $constroller('BaseCtrl', {$scope: $scope});.

### Widget extension model
--------------------------
There might be cases were we would need to build a widget on top of an existing functionality. A clear example of this scenario is the product details page (PDP) of Nicky and Adams kiosk.
In that application the PDP can be thought as a widget build on top of the Kmart kiosk application's PDP since it shows a subset of the information present in that page along side a different look and feel.

To handle shuch scenarios we will follow a model or pattern similar to the one explaind in the previous section.
This means, the widget will consist on a custom template with its own styles plus a controller that will extend from the controller of the functionality or feature that we are taking as reference. This extension will be again made using the $scontroller AngularJS service.

Widgets will be include in a page using the ngInclude AngularJS directive and its template will reference the widget's controller with the ngController AngularJS directive.

In the case where the base controller consumes URL parameters using the AngularJS $routeParams service but the widget controller needs to take them from another source (e.g. an http service) we will pass a custom $routeParams object to the base controller, like this:

var routeParams = { ... };

$controller('BaseCtrl', {$scope: $scope, $routeParams: routeParams});

## Navigation orchestration
---------------------------
To navigate from one page to another we will use a custom AngularJS service that will take care of this.
That service will do mainly two things: a) normalize the target URL to be sure it is a valid URL b) navigate to the target URL by changing the hash part of the current URL with the help of the $location.path() AngularJS method.

The navigation procedure will not only have to take care of the forward navigation but also of the back one.
The back navigation is kind of special because there might be cases where we do not want to go back to the immediately previous URL but to an older one.
This happens for instance if a user is trying to go back to a multipage form, in that case the back navigation should take the user to the first page of the form.

To handle these cases the controller of each URL will inform the navigation service if the target URL should be kept in the history or not.
In this way when the user hits the back button we will have control over the page he will land on since the pages where we do not want him to be able to go back will not be present in the history and so will not be accesible through this action.

To achieve this behavior the navigation service will use the .replace() method when changing the current URL to the target one if the controller asks so.

## POC
-----------------
A POC was built to evaluate the feasibility of a common code being used among several applications.

The code is in a private github repo on the url:

  https://github.searshc.com/StealthGlobant/shop-sears-2.5
  
The POC involves the MSFT POC and a sketch of SS 2.5 for iPods.
See below captures of both being built from the same codebase.

## Configuration management draft
-----------------
Interaction between apps might involve different repositories interacting, please refer to pending definitions.
Only elements not related to the Angular seed will be shown.

For all other elements in the app pls refer to the Angular seed repo **[pending upload copy to sears repo]**

* root dir
   * shared
      * scripts
         * components
             * common
                 * …
             * PDP
                 * ...
             * ...
   * apps
      * SS2.5
         * styles
         * img
         * components
             * PDP
                 * …
             * TryOn
                 * …
             * ...
      * SYW Kiosk
         * ...
      * ….


### Configuration Management
-----------------
In order to be able to report progress in the actual applications, they CM apprach and deliverable optimization will be simplified at first and improved as the development process moves forward.

### Initial setup
-----------------
Separate repositories will be created for each application. An extra repository will be created for the common code.

![code in repos](Readme_Imgs/code_in_repos.png)
Diagram 5: Application specific code in separate repos rely in components brought from the shared code repo

Limited team members from different application teams will have access to push to the shared repository.
The code in the shared code repo will be pulled into a separate folder the application specific repository.
Grunt will join the content from both sources when building the application.

### Platform-repository correlation
-----------------
The suggested approach does not state that the same repository and codebase needs to be shared for different clients of the same application.
An application could intend to have differential templates or use responsive design as part of the same containing applications to support multiple device sizes.

However, if the flows and layouts for the same application on different sized happen to be 
Once built, the code could be targeted to be downloaded from a web server, or packed within the native deliverable.

![app distribution platforms](Readme_Imgs/app_distribution_platforms.png)
Diagram 6: One application being distributed to different platforms

### Improvements on code distribution
-----------------
Although today’s idea is that shared code will be used as a library, we will evaluate if it is possible standardize interfaces so they can be used as ng-modules.

This is, however, not necessary for this process to work.

### Improvements on code optimization
-----------------
By now, all code will be included in the build of each platform.
Over time and as the size for the shared code grows, we will improve the build process to pull only what is necessary for each platform.


## Pull requests issues
-----------------
Common causes for a pull requests for being rejected
### Changing global element styles
Global style elements shall be changed ONLY with SME guidance.
A change in this area will affect the whole app.
Elements under the .ss-widgets class are supposed to be reusable.
If needing to override a style, use the same class within your .module-class-name{}

### Obtrusive css
Sass encourages the use of non obtrusive css.
All styles in your .scss file should be enclosed by a unique .my-feature-class-name{} that applies only to your feature

### Globals and rootScope
Rootscope and Global variables should not be used.
Create a service, save the value you need and keep the scope as atomic as possible.

### Adding css files
Css files will be calculated by SASS.
When added directly in the codebase, SASS might not catch the newer version and will not compile styles.
