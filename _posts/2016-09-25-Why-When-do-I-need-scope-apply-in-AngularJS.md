---
layout: post
title: "Why/When do I need $scope.$apply() in AngularJS"
date: 2016-09-25T01:15:00.000Z
intro: "When we start building complex/production applications in AngularJS we all have come across this scenario where we require to manually trigger the Angular Digest Cycle using $scope.$apply(). In this blog, I will be sharing my learnings."
categories: front-end
---

# Intro

When we start building complex/production applications in AngularJS we all have come across this scenario where we require to manually trigger the [Angular Digest Cycle](https://www.sitepoint.com/understanding-angulars-apply-digest/) using $scope.$apply(). In this blog, I will be sharing my learnings.

# Why do I need $scope.$apply()

AngularJS uses a clever trick of re-rendering the entire view if any of the scope properties is changed. So it is really important for Angular to know when the scope is changed so that it can update the DOM automatically. This is one the reasons why angular has wrappers for native JS methods like `setTimeout`, `setInterval`, etc. By using these methods, Angular internally takes care of updating the DOM internally. But what if the API/Library that you are using is not available as an Angular wrapper (the new native Promise API in my case). Well, in that case, you need to trigger the DOM update yourself using $scope.$apply().

# Example

I've created a [fiddle](http://jsfiddle.net/rgabs/xLc4tnrf/) for this example comparing `$http` service by angular with the new ES6 native Promise API.

Code Explanation:

```html
<div ng-controller="MyCtrl">
  Hello, {% raw  %}{{name}}{% endraw %}!
</div>
```

Here, our view is bound to the `MyCtrl` controller is which scope contains a property called `name`.

```javascript
var myApp = angular.module('myApp',[]);

myApp.factory('myService', function ($http){
    return {
      getData: ()=>{
        return $http.get('/')
    },
    getDataFromPromise: ()=>{
        return new Promise((resolve,reject)=>{
           resolve($http.get('/'))
      })
    }
  }
})
function MyCtrl($scope, myService) {
    $scope.name = 'nananananana';
    myService.getData().then(()=>{
      $scope.name ='Batman'
    })
}
```

The above code will work as expected and will change output will be `Hello, Batman!`.

Now lets use the the new ES6 Promise API to do the same.

```javascript
function MyCtrl($scope, myService) {
    $scope.name = 'nananananana';
    myService.getDataFromPromise().then((res)=>{
      $scope.name ='Bruce Wayne' //The view will not update
    })
}
```
The expectation here is that the View will update the name to 'Bruce Wayne' but it will not because we are not **directly** using the $http service.

The quick fix here would be calling $scope.$apply(); method just after updating the scope property in the controller. (Please be extra cautious here as there might be a digest cycle already running.)

## Fiddle

<script async src="//jsfiddle.net/rgabs/xLc4tnrf/embed/js,html,css,result/dark/"></script>

# Important things to take care when using $scope.$apply()

- Make sure that the there is no digest cycle is running when you manually trigger it or else you might get an error saying `Error: $digest already in progress`. Refer to this blog for solving this issue (<http://www.bennadel.com/blog/2605-scope-evalasync-vs-timeout-in-angularjs.htm>)
- Prefer using Angular service wrappers instead of using native JS functions. (In this case, use $q service by Angular instead of native Promise API)
- Try to use directives whenever possible as $scope.$apply() will just update the view associated with that specific directive.
